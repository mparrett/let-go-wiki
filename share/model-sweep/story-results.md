# Story results: a GPU droplet that documents let-go

> **Cost figures revised 2026-09-10.** The runner now records token usage, and
> the first measured run showed the input at **10,660 tokens** against the 5,250
> these numbers were estimated from — a byte-count estimate was 2x low, because
> a commit diff tokenises far worse than prose (1.97 bytes/token, not 4). Input
> is now measured; output is still estimated, so the per-run costs below are
> good to about an order of magnitude, not to the cent. Re-running the sweep
> would settle both sides.


> *When a commit lands in `let-go`, a GitHub Action wakes up, gets a machine with
> a language model running on it, checks out `let-go-wiki`, runs a documentation
> job, opens a pull request with the result, and then the machine goes away.
> Ephemeral; the only thing that remains is logs. No idle cost.*

Status as of 2026-09-10, after two days.

**The pipeline is built and running in production. The model still runs on
somebody else's hardware.** Everything except "a machine with a language model
running on it" is done and has produced a real pull request. The local-model half
is unstarted, and the two days of costing say it should probably stay that way
unless the reason is independence rather than price.

---

## a. Did Monk work for DigitalOcean orchestration?

**Yes.** Provisioned, deployed, ran to completion, destroyed, verified nothing
left behind.

| | |
|---|---|
| Cluster | `letgo-docs-14e1746d1b5d`, one `s-1vcpu-2gb`, `nyc2` |
| Provisioning | 10m 33s to `active`, 15m 17s fully settled |
| Job | ran to completion against the live Anthropic API |
| Teardown | cluster destroyed; cluster list empty, context back to local |
| Total trial spend | **~2 cents** |

Two findings came out of it that changed the design:

**Provisioning dominates.** The job takes about a minute; the machine takes ten
or more to exist. A cluster per commit spends ninety percent of its life booting.
That does not undermine ephemerality, but it does kill the
one-machine-per-commit shape — batching, or warm-with-a-deadline, is the right
arrangement. At $5.51/hr that boot time is **$0.92 spent before the model sees a
token**.

**The rehearsal paid for itself.** The first cloud deploy failed with
`exec container process /usr/local/bin/python: Exec format error` — a base-image
digest pinned on Apple Silicon had silently locked every build to arm64 while
every deploy target was x86_64. Nothing local could have caught it, because local
is also arm64. That is now an asserted property in CI.

**In phase one the droplet earned nothing.** A GitHub Actions runner is already
ephemeral. The cluster was built as practice for
a GPU phase where mistakes are expensive, and it justified itself by catching the
architecture bug, not by doing work CI could not.

Full review of the platform itself: [`monk-io-review.md`](monk-io-review.md).

---

## b. Did the overall workflow work, on both backends?

**Yes on Anthropic, end to end, in production.** **Yes on OpenRouter, but only
through the container — not through Monk and not through CI.** Precisely:

| Path | Anthropic | OpenRouter |
|---|---|---|
| Local, no API key (`claude-cli` backend) | ✅ | n/a |
| Container on local `monkd`, live API | ✅ | ✅ (2026-09-10, 9 models) |
| Container on a DO droplet via Monk | ✅ | ❌ not run |
| GitHub Action, end to end | ✅ → opened a PR | ❌ not run |

**The production run.** `wiki-docs.yml` dispatched on `mparrett/let-go`, dry then
real, ~57s wall clock each (model call 32–37s, image build 18–21s), producing
[let-go-wiki#1](https://github.com/mparrett/let-go-wiki/pull/1). The two-step
credential split was proven in the process: in the dry run the real step is
`skipped`, and vice versa, so the wiki write token is *absent* from the dry run's
environment rather than merely unused.

**The OpenRouter sweep.** Nine models on one commit, ~3 cents total, through the
existing `openai-compat` backend. Six produced a page that cleared the gate;
three were rejected by the runner's own validation — invented tags, broken links,
unterminated frontmatter.

The headline result matters for part (c): **a 9B model did the job.**
`qwen/qwen3.5-9b` scored 7/7 on content markers and cleared the validator, with a
per-var table, source line citations, and the caveat that the performance figures
are a single-machine proxy. A 70B scored 1/7 and was rejected outright.
**Parameter count predicted nothing.**

A second sweep complicated the ranking without changing that conclusion:
`gpt-oss-120b` and `gpt-oss-20b` also reached 7/7, and `gpt-oss-120b` did it at
less than half the cost. So `qwen3.5-9b` is not uniquely best. It stays the model
that matters for self-hosting for a different reason — **size**. At 9B it fits
the 20GB card; `gpt-oss-120b` needs 80GB, which is the expensive tier.

**What the dogfooding cost us, and bought us.** Running the thing on itself found
three defects that nothing else would have: an index entry written outside any
section, an entirely empty page that passed the wiki's validator, and an
evaluation entrypoint holding a credential it never used. All three are fixed and
merged. The empty-page one is the instructive one — `check_wiki.py` requires the
schema keys and has no opinion about whether a page says anything, so a
frontmatter-only file cleared every automated gate and would have been proposed
for human review.

**Known gap:** the Anthropic path is the only one exercised through Monk and
through CI. The eval entrypoint's Monk configuration is correct as written and
was reviewed, but has never been deployed.

---

## c. The local model: not started, and the economics argue against it

Untouched. Here is what the costing says before anyone spends a day on it.

### Self-hosting costs two orders of magnitude more than renting

For the *same open-weights model*:

| Serving | Cost per run |
|---|---|
| `gpt-oss-120b` via OpenRouter | **$0.0007** (measured) |
| Same class of model, ephemeral RTX 4000 Ada | **~$0.158** |

The reason is structural, not a bad rate: a shared provider amortises a GPU
across thousands of tenants, and we would amortise a whole GPU across one
30-second request — most of it boot time. Always-on does not rescue it either;
break-even against ephemeral is ~4,380 runs/month (~144/day) **for every card**,
since both sides scale with the same hourly rate. `let-go` produces single-digit
runs a day.

So the GPU phase cannot be justified on price. It can be justified on
independence from a third party, data locality, or running a custom or
fine-tuned model nobody hosts — all legitimate, none of them cost.

### The cheap cards exist, but Monk does not offer them

| Card | VRAM | DO $/hr | +25% fee | reachable via Monk |
|---|---|---|---|---|
| RTX 4000 Ada | 20GB | 0.76 | **0.95** | **no** |
| RTX 6000 Ada / L40S | 48GB | 1.57 | **1.96** | **no** |
| MI325X | 256GB | 3.80 | 4.75 | yes |
| H100 | 80GB | 4.41 | 5.51 | yes |

The 9B model that cleared the gate fits comfortably on the 20GB card. The
cheapest card Monk exposes is six times its price, and the cheaper-to-rent
`gpt-oss-120b` would need that expensive tier anyway. *(Checked `nyc1` and `nyc2`, not all
sixteen regions.)*

### Three routes, ranked

**1. Serverless inference — already effectively done.** OpenRouter *is* the
serverless option, and the sweep proved the integration works. If the goal is
"not managed by us", this is finished; it costs fractions of a cent per run and
needs no further work. Dedicated serverless GPU platforms would sit in the same
box and would need a look outside DigitalOcean.

**2. Provision a cheap GPU outside Monk.** DO sells the RTX 4000 Ada directly.
This gets a real self-hosting test for ~$0.95/hr, at the cost of stepping outside
the orchestration layer we just validated, which defeats the point of the Monk
trial.

**3. Wait for the catalog to widen.** Worth filing as a missing-capability
request. The cheap single-GPU tier is the whole basis of a self-hosting plan.

### Already in place

The integration is a **base-URL swap, with zero code changes**. The
`openai-compat` backend takes `MODEL_BASE_URL` and `MODEL_API_KEY`; pointing it
at a vLLM or Ollama endpoint instead of OpenRouter is a configuration change. The
sweep exercised that path nine times today.

Missing before a GPU run:

- **A cost janitor.** Nothing currently creates clusters automatically, so it is
  not urgent — but it becomes urgent the instant something does. A leaked H100 is
  ~$132/day, and DigitalOcean bills a droplet for *existing*, not for running, so
  powering one off saves nothing. Only destroying it stops the meter.
- **Batching.** Given the 10-minute provisioning floor, per-commit clusters are
  the wrong shape on GPU far more than on a $0.025/hr CPU node.
- **A measured provisioning time for a GPU node.** Every figure above uses the
  CPU node's ~10 minutes as a stand-in.

---

## Where it stands

| | |
|---|---|
| ✅ | Pipeline built, tested, running as a GitHub Action; opened a real PR |
| ✅ | Monk orchestrates DO: provision, deploy, run, destroy, verified clean |
| ✅ | Two model backends behind one interface; both exercised |
| ✅ | Nine candidate models evaluated against the real task and the wiki's validator |
| ⬜ | Automatic trigger — `on: push` is written and deliberately commented out |
| ⬜ | Local model on a GPU — unstarted, and blocked on catalog rather than capability |
| ⬜ | Cost janitor — not needed until something provisions automatically |

The capability objection to self-hosting is gone: a model small enough to serve
on a $0.95/hr card does this job well. What remains is that nobody can rent that
card through Monk yet, and that renting the same model from a shared provider is
three orders of magnitude cheaper. Those are the two facts the next decision
should turn on.

---

*Evidence: `devlog/` in this repository, dated entries with raw logs, timings,
screenshots, and the full model-sweep results.*
