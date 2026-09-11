# Model sweep — which model can document a commit?

Nine models, one question: given commit `14e1746d1b5d` of `let-go`, which adds a
design document, does the wiki need a new page — and if so, write it. Same
commit and same wiki state every time.

**Interactive version** (click a row, read pages side by side):
<https://mparrett.github.io/let-go-wiki/assets/model-sweep.html>

## Results

Sorted cheapest first. Tokens are measured, not estimated — the runner records
`response.usage`. **Markers** counts how many of seven specifics from the source
document reached the page. **Fits** is the smallest DigitalOcean card that could
serve the model.

| Model | Size | Fits | Verdict | Markers | In | Out | $/run | Filed as |
|---|---|---|---|---|--:|--:|--:|---|
| `openai/gpt-oss-120b` | 120B MoE | H100 80GB | ✅ passed | 7/7 | 10,293 | 1,749 | $0.0007 | `concepts/ir-pipeline-dynamic-vars.md` |
| `openai/gpt-oss-20b` | 20B MoE | RTX 4000 Ada 20GB | ✅ passed | 7/7 | 10,289 | 4,374 | $0.0009 | `concepts/ir-pipeline-dynamic-vars.md` |
| `meta-llama/llama-3.3-70b-instruct` | 70B | H100 80GB | ❌ failed | 1/7 | 10,188 | 221 | $0.0011 | `sources/ir-dynamic-vars.md` |
| `mistralai/mistral-small-3.2-24b-instruct` | 24B | L40S 48GB | ✅ passed | 7/7 | 10,657 | 1,722 | $0.0011 | `concepts/ir-dynamic-vars.md` |
| `google/gemma-3-12b-it` | 12B | RTX 4000 Ada 20GB | ❌ failed | 7/7 | 11,141 | 3,895 | $0.0011 | `concepts/ir-dynamic-vars.md` |
| `qwen/qwen3-32b` | 32B | L40S 48GB | ⚠️ model-proposed-nothing | 0/7 | 10,394 | 1,400 | $0.0012 | *no page* |
| `qwen/qwen3.5-9b` | 9B | RTX 4000 Ada 20GB | ✅ passed | 7/7 | 10,660 | 4,425 | $0.0017 | `sources/design-ir-dynamic-vars.md` |
| `qwen/qwen3.6-35b-a3b` | 35B MoE | L40S 48GB | ✅ passed | 7/7 | 10,660 | 9,148 | $0.0093 | `concepts/ir-dynamic-vars.md` |
| `anthropic/claude-sonnet-5` | — | cloud only | ✅ passed | 7/7 | 15,490 | 3,381 | $0.0648 | `sources/design-ir-dynamic-vars.md` |

## What it says

**Parameter count predicted nothing.** A 70B scored 1/7 and was rejected; a 9B
scored 7/7 and passed.

**The cheapest model is also a top scorer.** `gpt-oss-120b` at $0.0007 with 7/7.
Nothing in the table orders price and quality together.

**Size, not rank, is what matters for self-hosting.** `gpt-oss-120b` is cheapest
to rent but needs 80GB — the expensive tier. `qwen3.5-9b` scores as well and fits
the 20GB card at $0.95/hr, which is the card that makes a cheap self-hosting plan
possible. That card is not in Monk's catalog.

**Sonnet costs 38x the 9B and 93x `gpt-oss-120b`** for the same 7/7 result here.

**Passing the validator is not the same as being good.** In an earlier sweep
`gpt-oss-20b` cleared every automated gate with a page that was frontmatter and
nothing else. The schema was right; the page said nothing.

## The reference

The wiki already keeps a page for this source document, written by **Claude Fable
5.1** and kept by a human reviewer (`32443ce`, `status: active`, unedited since).
That is the bar a candidate clears: not an arbitrary human, but frontier-model
output that survived review.

<details>
<summary><b>Reference page</b> — <code>sources/design-ir-dynamic-vars.md</code></summary>

````markdown
---
type: Source
category: source
title: "IR pipeline dynamic vars (design reference)"
description: "The single index of every ^:dynamic var the IR compile and lowering pipeline reads: compilation-mode knobs, pass toggles, cross-package lowering control, per-compile state, and how to verify a var change with the ir-stress harness."
tags: [compiler, reference, tooling]
resource: "https://github.com/nooga/let-go/blob/main/docs/design/ir-dynamic-vars.md"
sources: ["doc: https://github.com/nooga/let-go/blob/main/docs/design/ir-dynamic-vars.md, 2026-09-05"]
created: "2026-09-05"
updated: "2026-09-05"
status: active
---

# IR pipeline dynamic vars (design reference)

Design reference added in #555 (last-verified 2026-07-19). The pipeline is configured almost entirely through dynamic vars scattered across `core.lg`, `passes/pipeline.lg`, `passes/inline.lg`, `passes/fusion.lg`, `passes/typeinfer.lg`, and `lower_go.lg`; only `*strict-structured?*` has an environment seed. The page is the index.

## Key takeaways

- **Mode control:** `*ir-compile*`, `*ir-compile-verbose*`, `*ir-compile-fallback-log*`, `*target*` (`:bytecode` or `:go`).
- **Cost of `*ir-compile*`:** pays back only on allocation-bound work (5 to 12 times fewer allocations, about 13% wall-clock, break-even near 64 runs on persistent-map); a net loss on compute-bound code. Figures from a single-machine gctrace proxy, directional not a contract.
- **Pass toggles:** `*enable-fusion*` (on, measured about 20% fewer allocations across the test suite), `*enable-inline*` (off), `*max-unroll*`, `*typeinfer-max-drains*`, `*strict-structured?*`, `*direct-calls-disabled?*`, `*pass-trace*`.
- **Cross-package lowering:** `*emit-exported-wrappers*`, `*cross-pkg-registry*`, `*wrapper-target-names*`, `*export-name-overrides*`, `*deftype-ctor-types*`.
- **Verification:** any default flip is a coverage change until `make ir-stress` / `ir-stress-gate` / `jank-stress` / `parity-full` say otherwise; diff bucket tallies, rebaseline with the tool, never hand-edit the EDN.

## Derived pages

[compile-paths](../concepts/compile-paths.md) · [ir-optimizations](../concepts/ir-optimizations.md) · [go-backend](../concepts/go-backend.md)

# Citations

[1] https://github.com/nooga/let-go/blob/main/docs/design/ir-dynamic-vars.md

````

</details>

## What each model wrote

<details>
<summary>✅ passed &nbsp; <b>openai/gpt-oss-120b</b> &nbsp; <code>$0.0007</code> &nbsp; 7/7 &nbsp; <code>concepts/ir-pipeline-dynamic-vars.md</code></summary>

````markdown
---
type: Concept
category: concept
title: "IR Pipeline Dynamic Vars"
description: "Design documentation of the IR pipeline dynamic variables that control compilation and lowering"
tags: [compiler, runtime, concept]
resource: "https://github.com/mparrett/let-go/tree/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: mparrett/let-go docs/design/ir-dynamic-vars.md, 2026-09-11"]
created: "2026-09-11"
updated: "2026-09-11"
status: speculative
---

# IR Pipeline Dynamic Vars

The IR compile/lowering pipeline in **let‑go** is configured almost entirely through a set of `^:dynamic` vars (often called *knobs*) rather than command‑line flags or build tags. These vars are defined across several source files (`core.lg`, `passes/pipeline.lg`, `passes/inline.lg`, `passes/fusion.lg`, `passes/typeinfer.lg`, `lower_go.lg`) and govern both **what** is compiled to IR and **how** it is lowered.

## Why they matter

- **Compilation‑mode control** – knobs such as `*ir-compile*` and `*target*` decide whether the IR path is used at all and which backend (`:bytecode` vs `:go`) is targeted.
- **Performance tuning** – flags like `*enable-fusion*`, `*enable-inline*`, and `*max-unroll*` affect allocation patterns, code size, and compile‑time.
- **Cross‑package linking** – vars prefixed with `*emit-exported-wrappers*` or `*cross-pkg-registry*` control the generation of Go wrappers needed for inter‑package calls.
- **Coverage impact** – changing any knob modifies the set of forms that successfully convert to IR. Therefore each change is a *coverage change* that must be verified.

## Verification harness

Every knob is tied to the **ir‑stress** harness, which runs a curated corpus of `.lg` sources through the selected IR path and reports per‑defn buckets (`:ok`, `:missing-form/set!`, `:stress/timeout`, …). The harness is documented in `scripts/ir-stress.md` and invoked via make targets:

```sh
make ir-stress            # basic coverage check
make ir-stress-gate      # ratcheted check against baseline
make jank-stress         # coverage over the jank Clojure‑compat suite
make parity-full         # compare AOT vs eval‑mode pipelines
```

A change to a knob is considered safe when the **ir‑stress‑gate** exits cleanly, indicating no regression in native‑lowering coverage. If intentional coverage moves occur, the baseline is re‑baselined with `make ir-stress-rebaseline`.

## Key knobs (selected)

| Var | Default | Semantics |
|---|---|---|
| `*ir-compile*` | `false` | Enables the IR‑compile path for single‑arity `defn`s. |
| `*target*` | `:bytecode` | Switches backend: `:go` routes to `ir.lower-go`. |
| `*enable-fusion*` | `true` | Activates transducer deforestation; lowers allocation count. |
| `*enable-inline*` | `false` | Master switch for the inline pass; gated by `ir‑stress`. |
| `*max-unroll*` | `32` | Caps unrolling of rest arguments; higher values increase code size. |
| `*strict-structured?*` | `false` | When true, structural‑control‑flow drift throws; seeded from `LG_STRICT_STRUCTURED`. |
| `*emit-exported-wrappers*` | `false` | Emits Go wrappers for cross‑package calls when targeting Go. |
| `*pass-trace*` | `nil` | Bind to an atom to capture per‑pass instruction traces. |

## Per‑compile state (non‑knobs)

These vars are re‑initialized each compilation run (e.g., `*current-fn*`, `*inline-registry*`, `*lowered-registry*`). They are listed for discoverability but are **not** intended to be manually set.

## Architectural context

The dynamic‑var system is a core part of the **IR pipeline** concept ([IR pipeline](../concepts/ir-pipeline.md)). It enables fine‑grained, runtime‑configurable control of compilation without recompiling the compiler itself, supporting both rapid experimentation and reproducible builds.

## Diagram

```mermaid
flowchart TD
    A[Start compile-form*] --> B{*target*}
    B -->|:bytecode| C[ir.lower (bytecode backend)]
    B -->|:go| D[ir.lower-go (Go backend)]

    C --> E{Knobs affect passes}
    D --> E

    E --> F[Passes: fusion, inline, type‑infer, …]
    F --> G[Emit lowered code / Go wrappers]

    G --> H[Runtime execution]
```

---

````

</details>

<details>
<summary>✅ passed &nbsp; <b>openai/gpt-oss-20b</b> &nbsp; <code>$0.0009</code> &nbsp; 7/7 &nbsp; <code>concepts/ir-pipeline-dynamic-vars.md</code></summary>

````markdown
---
type: Concept
category: concept
title: "IR pipeline dynamic vars"
description: "Index of dynamic vars controlling the IR compile/lowering pipeline"
tags: [compiler, vm, bytecode]
resource: "https://github.com/mparrett/let-go/blob/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: mparrett/let-go docs/design/ir-dynamic-vars.md, 2026-09-11"]
created: "2026-09-11"
updated: "2026-09-11"
status: speculative
---
# IR pipeline dynamic vars

The IR compile/lowering pipeline is configured almost entirely through
`^:dynamic` vars rather than flags or options. They are scattered across
`core.lg`, `passes/pipeline.lg`, `passes/inline.lg`, `passes/fusion.lg`,
`passes/typeinfer.lg`, and `lower_go.lg`, and only one (`*strict-structured?*`)
has an environment seed — the rest are set with `binding` / `set!`. This page
is the single index.

Two kinds of var live here:

- **Knobs** — you set them (via `binding` or `set!`) to change what the
  pipeline does. Documented with defaults and semantics below.
- **Per-compile state** — `nil`/empty-initialized vars the pipeline rebinds
  as it runs. Listed at the end so they're discoverable, but they are not
  settings; binding them by hand will usually just break a compile.

For build tags, environment variables, and `lgbgen` CLI flags, see the
companion sections in the guide; this page covers the dynamic vars only.

Every knob here changes what the pipeline converts to IR (or how it lowers
it), so a change to one is only as good as its coverage evidence. Measure it
with the ir-stress harness — see [Verifying a var change](#verifying-a-var-change)
at the end of this page and `scripts/ir-stress.md` for the harness itself.

## Compilation-mode control

The entry knobs that decide whether IR compilation runs at all and which
backend it targets.

| Var | Default | Semantics |
|---|---|---|
| `*ir-compile*` | `false` | Routes single-arity `defn`s through the IR-compile path instead of standard bytecode expansion (multi-arity and docstring-less edge cases still fall back). Enable with `(set! *ir-compile* true)` **after** `(require 'ir.passes.pipeline)` — it throws if the pipeline isn't loaded. There is no environment variable for it. `core.lg:963` |
| `*ir-compile-verbose*` | `false` | When true, the `defn` macro logs a diagnostic (name + error) each time a fn falls back to bytecode. `core.lg:968` |
| `*ir-compile-fallback-log*` | `(atom [])` | Vector of `[name error-msg]` fallback records, populated while `*ir-compile-verbose*` is true. `core.lg:973` |
| `*target*` | `:bytecode` | `:go` makes `compile-form*` route through `ir.lower-go` (native Go) instead of `ir.lower` (bytecode). Bind it before calling `compile-form`. `passes/pipeline.lg:545` |

**Cost of `*ir-compile*`** — it is the one knob with a real amortization
story, and turning it on is not free. Pipeline load plus per-fn compile is a
one-time cost paid up front, so it pays back only when amortized *and* only on
allocation-bound work:

- **Alloc-heavy workloads** (persistent-map, transducers, `reduce`): ~5–12×
  fewer allocations, converting to roughly −13% wall-clock per run once heap
  churn dominates — break-even around 64 runs on persistent-map.
- **Compute-bound code** (`fib`, `loop`/`recur`): a small net loss that never
  breaks even; the compile cost has nothing to amortize against.
- Most of the win is the pipeline cutting per-element boxing, not fusion:
  `reduce` over `range` drops ~78% of allocations with no fusion applicable.

Figures from @mparrett's measurement on #555, via a gctrace GC-cycle proxy on
a single machine — directional, not a contract. `BenchmarkIRCompile`
(`pkg/ir/ir_compile_bench_test.go`) is the instrument to re-measure with, and
`make ir-stress` is what tells you whether a change moved *coverage* rather
than just cost.

## Pass toggles & tuning

| Var | Default | Semantics |
|---|---|---|
| `*enable-fusion*` | `true` | Transducer / deforestation fusion, placed after `cse`. On by default — measured ~20% fewer allocations across the ClojureTestSuite with backend parity and no ir-stress regression (`make ir-stress-gate`). `passes/fusion.lg:27` |
| `*enable-inline*` | `false` | Master switch for the inline pass. Opt-in: inlining supersedes the #345 direct-call path and still has rough edges (deftype-devirt codegen), so it stays off outside the AOT combinator measurement harness. Flipping it on is a coverage change — gate it with `make ir-stress-gate` and `make parity-full`. `passes/inline.lg:25` |
| `*max-unroll*` | `32` | Cap on fold-over-rest unrolling (ITER-0034). A combinator call with more than this many flat rest operands is left as a runtime call with a logged skip — never silently truncated — rather than unrolled into an oversized branch chain. Raising it trades compile time for code size; watch `:stress/timeout` buckets via `make ir-stress`. `passes/inline.lg:30` |
| `*typeinfer-max-drains*` | `2000000` | Backstop bound on the typeinfer fixpoint for pathological inputs (never fires on real code). The bail is sound — every assigned type is monotone and `lower-go`'s `rt.<Op>Value` path handles `:any` operands. Bind `nil` for unbounded. `passes/typeinfer.lg:494` |
| `*strict-structured?*` | `false` (seeded from `LG_STRICT_STRUCTURED`) | When true, structured-control-flow drift throws (and the caller falls the whole fn back to bytecode) instead of emitting a possibly mis-lowered `goto` body. Default off: the non-strict path stays correct via the coalesce-map interference fix; this just forbids the path. `lower_go.lg:2477` |
| `*direct-calls-disabled?*` | `false` | Forces every call through the cached-var / `InvokeValue` trampoline (which re-reads the var root each call) so runtime `alter-var-root` / `intern` overrides are observed. A baked direct call — `corefns.Count`, a lowered sibling's Go func — would otherwise ignore them. `lower_go.lg:1863` |
| `*pass-trace*` | `nil` | Bind to an atom to capture per-pass instruction traces. `passes/trace.lg:39` |

## Cross-package / exported-wrapper control (`--target=go`)

Knobs for whole-program Go lowering, where one lowered package must call into
another. Defaults keep the committed lowered tree byte-identical until the
whole-program collector binds them.

| Var | Default | Semantics |
|---|---|---|
| `*emit-exported-wrappers*` | `false` | Emit an exported thin forwarding wrapper for each direct-callable lowered fn so it is reachable from another Go package. Off keeps bootstrap codegen byte-stable; flipped on by the collector and the T3 unit test. `passes/pipeline.lg:1065` |
| `*cross-pkg-registry*` | `{}` | Whole-program `{[internal-ns name arity] -> {:go-pkg <import> :go-name "LG_<go>" …}}` of every other lowered package's direct-callable exports. Merged into the per-ns registry so a cross-package call resolves to `pkg.LG_<go>(ec, …)`. `{}` ⇒ no cross-package entries. `passes/pipeline.lg:1073` |
| `*wrapper-target-names*` | `:all` | Which fns get an exported wrapper. `:all` = every direct-callable fn (the single-ns convenience); a concrete set = exactly its members; an empty set = none. `lower-all-ns-to-go` always binds the concrete set, so a whole-program build with no cross-package references emits no dead exported API. `passes/pipeline.lg:1086` |
| `*export-name-overrides*` | `nil` | Per-namespace `{source-name -> resolved exported Go name}` bound around a namespace's collect + lower passes. PascalCase is not injective, so this remaps the loser of any collision to a distinct name. `nil` = plain PascalCase (the collision-free case). `lower_go.lg:3387` |
| `*deftype-ctor-types*` | `nil` | `{constructor-name -> deftype-name-symbol}` bound around a `:go` typeinfer pass so a call to a known constructor — `(->Square 3)` — is typed `[:dtype Square]`, carrying the concrete receiver type to field access and devirtualized dispatch. `nil` = off, zero overhead. `passes/typeinfer.lg:50` |

## Per-compile state (not knobs)

These are `nil`/empty-initialized and rebound by the pipeline as it runs.
They are listed for discoverability; setting them by hand is not a supported
configuration surface.

| Var | Location | Role |
|---|---|---|
| `*current-fn*`, `*current-inst*`, `*current-zip*` | `passes.lg:23-25` | Current traversal cursor (fn / instruction / zipper). |
| `*inline-registry*` | `passes/inline.lg:20` | Inline-candidate registry for the inline pass. |
| `*lowered-registry*` | `lower_go.lg:1342` | Registry of lowered namespaces / fns for cross-ns direct-call lowering. |
| `*native-imports-used*` | `lower_go.lg:1352` | Go imports referenced by the fn currently being emitted. |
| `*cross-ns-vars-used*` | `lower_go.lg:1371` | Cross-ns var references collected during emission (feeds the cross-package collector). |
| `*call-err-used*` | `lower_go.lg:1645` | Whether the emitted fn body needs the `callErr` plumbing. |
| `*typed-call-temps*` | `lower_go.lg:1657` | Temp bindings for typed direct calls in the current fn. |
| `*closure-arg-prefix*` | `lower_go.lg:64` | Prefix disambiguating closure-local arg names (captured-name shadowing fix). |
| `*force-needs-error*` | `lower_go.lg:2465` | Forces error plumbing on for a body regardless of inference. |
| `*deftype-ctors*` | `lower_go.lg:1906` | Deftype constructors in scope for native ctor-call emission. |
| `*protocol-methods*` | `lower_go.lg:1935` | Protocol method table for devirtualized dispatch. |
| `*protocol-method-sigs*` | `lower_go.lg:1943` | Protocol method signatures. |
| `*defmulti-dispatchers*` | `lower_go.lg:2056` | Type-dispatched `defmulti`/`defmethod` tables. |
| `*ti-counters*` | `lattice.lg:110` | Typeinfer instrumentation counters. |

## Verifying a var change

The vars above decide which forms convert to IR and how they lower, so
flipping a default (or adding a knob) is a coverage change until proven
otherwise. `scripts/ir-stress.lg` is the harness that measures it: it drives a
corpus of `.lg` sources through a chosen IR path and reports per-defn buckets
(`:ok`, `:missing-form/set!`, `:validate/no-term`, `:stress/timeout`, …) so a
conversion regression shows up as a bucket that moved, not as a vague slowdown.
Full harness reference — modes, env vars, bucket meanings — is in
`scripts/ir-stress.md`.

The checks, cheapest first:

| Command | What it answers |
|---|---|
| `make ir-stress` | Native-lowering pass rate over the committed corpus allow-list (`scripts/ir-stress-corpus.edn`). The everyday "did my knob drop coverage?" run. |
| `make ir-stress-gate` | Same census, ratcheted against `docs/perf/ir-stress-baseline.edn` — exits non-zero if native-lowering failures grew. This is the gate; the ratchet only tightens. |
| `make jank-stress` | Coverage over the vendored jank Clojure-compat suite, which reaches Clojure surface the internal corpus doesn't (BigDecimal literals, multimethods). |
| `make parity-full` | Both ir-stress modes (`lower-go` and `ir-compile`) plus clojure-test-suite, run once untagged and once under `-tags gogen_ir`, comparing bucket-by-symbol. Catches a var whose effect differs between the two engines. |

Which mode maps to which claim:

- **`lower-go`** — AOT conversion coverage. Use for any var read by
  `lower_go.lg` or bound for `*target* :go` (`*emit-exported-wrappers*`,
  `*cross-pkg-registry*`, `*wrapper-target-names*`, `*export-name-overrides*`,
  `*deftype-ctor-types*`, `*strict-structured?*`, `*direct-calls-disabled?*`).
- **`ir-compile`** — eval-mode conversion, i.e. what users hit at load time
  under `*ir-compile*`. Slower, since it evals each defn.
- **`trace`** — one defn, per-pass timings. Reach for it when a tuning knob
  (`*typeinfer-max-drains*`, `*max-unroll*`) produces `:stress/timeout` and you
  need to know which pass is responsible before tuning blindly.

Practical recipe for a default flip: capture a census on both settings and diff
the bucket tallies rather than eyeballing pass counts.

```sh
LG_STRESS_LOG=/tmp/before.log make ir-stress
# flip the var's default (or bind it in the harness), rebuild
LG_STRESS_LOG=/tmp/after.log  make ir-stress
# bucket-level diff: any row that moved is a conversion change
diff <(cut -f3 /tmp/before.log | sort | uniq -c) \
     <(cut -f3 /tmp/after.log  | sort | uniq -c)
```

If the change intentionally moves coverage, rebaseline the ratchet with
`make ir-stress-rebaseline` (tool-maintained — never hand-edit the EDN), review
the diff, and commit it alongside the change that caused it. This is the
evidence `*enable-fusion*` cites above ("no ir-stress regression") and the
standard any new default should meet.

## Notes

- Only `*ir-compile*` and `*target*` are ordinarily set by callers; the rest
  are pass-internal defaults that tests and the whole-program driver rebind.
- Coverage claims about any of these vars should cite an ir-stress run — see
  [Verifying a var change](#verifying-a-var-change). `make ir-stress-gate` is
  the ratcheted form of that check.
- `*strict-structured?*` is the only var with an environment seed
  (`LG_STRICT_STRUCTURED`); everything else is `binding` / `set!` only.
- Line numbers are anchors, not contracts — grep the var name if a file has
  drifted since `last-verified`.

````

</details>

<details>
<summary>❌ failed &nbsp; <b>meta-llama/llama-3.3-70b-instruct</b> &nbsp; <code>$0.0011</code> &nbsp; 1/7 &nbsp; <code>sources/ir-dynamic-vars.md</code></summary>

**Validator rejected it:**
- `sources/ir-dynamic-vars.md: unknown tag 'ir' (not in taxonomy)`

````markdown
---
type: Source
category: source
title: "IR Dynamic Vars"
description: Index of IR pipeline dynamic variables controlling the let-go compiler
tags: [compiler, ir, bytecode]
resource: "https://github.com/mparrett/let-go/blob/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: mparrett/let-go docs/design/ir-dynamic-vars.md, 2026-09-11"]
created: "2026-09-11"
updated: "2026-09-11"
status: speculative
---
The `docs/design/ir-dynamic-vars.md` file documents the dynamic variables used in the IR pipeline of the let-go compiler. It provides an index of the variables, their defaults, and their semantics, as well as information on how to verify changes to these variables using the ir-stress harness. This page is a key resource for understanding the let-go compiler's configuration and behavior.

````

</details>

<details>
<summary>✅ passed &nbsp; <b>mistralai/mistral-small-3.2-24b-instruct</b> &nbsp; <code>$0.0011</code> &nbsp; 7/7 &nbsp; <code>concepts/ir-dynamic-vars.md</code></summary>

````markdown
---
type: Concept
category: concept
title: "IR Dynamic Variables"
description: "Dynamic variables controlling the IR compilation and lowering pipeline"
tags: [compiler, bytecode, vm, clojure, go]
resource: "https://github.com/nooga/let-go/blob/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: nooga/let-go docs/design/ir-dynamic-vars.md, 2026-09-11"]
created: "2026-09-11"
updated: "2026-09-11"
status: speculative
---

# IR Dynamic Variables

The IR compilation and lowering pipeline in let-go is controlled almost entirely through dynamic variables (`^:dynamic` vars) rather than flags or options. These variables are scattered across several namespaces and are used to configure the pipeline's behavior.

## Compilation-mode Control

The entry knobs that decide whether IR compilation runs at all and which backend it targets.

| Var | Default | Semantics |
|---|---|---|
| `*ir-compile*` | `false` | Routes single-arity `defn`s through the IR-compile path instead of standard bytecode expansion. |
| `*ir-compile-verbose*` | `false` | When true, the `defn` macro logs a diagnostic each time a fn falls back to bytecode. |
| `*ir-compile-fallback-log*` | `(atom [])` | Vector of `[name error-msg]` fallback records, populated while `*ir-compile-verbose*` is true. |
| `*target*` | `:bytecode` | `:go` makes `compile-form*` route through `ir.lower-go` (native Go) instead of `ir.lower` (bytecode). |

**Cost of `*ir-compile*`** — it is the one knob with a real amortization story, and turning it on is not free. Pipeline load plus per-fn compile is a one-time cost paid up front, so it pays back only when amortized and only on allocation-bound work.

## Pass Toggles & Tuning

| Var | Default | Semantics |
|---|---|---|
| `*enable-fusion*` | `true` | Transducer / deforestation fusion, placed after `cse`. |
| `*enable-inline*` | `false` | Master switch for the inline pass. |
| `*max-unroll*` | `32` | Cap on fold-over-rest unrolling. |
| `*typeinfer-max-drains*` | `2000000` | Backstop bound on the typeinfer fixpoint for pathological inputs. |
| `*strict-structured?*` | `false` (seeded from `LG_STRICT_STRUCTURED`) | When true, structured-control-flow drift throws instead of emitting a possibly mis-lowered `goto` body. |
| `*direct-calls-disabled?*` | `false` | Forces every call through the cached-var / `InvokeValue` trampoline. |
| `*pass-trace*` | `nil` | Bind to an atom to capture per-pass instruction traces. |

## Cross-package / Exported-wrapper Control

Knobs for whole-program Go lowering, where one lowered package must call into another.

| Var | Default | Semantics |
|---|---|---|
| `*emit-exported-wrappers*` | `false` | Emit an exported thin forwarding wrapper for each direct-callable lowered fn. |
| `*cross-pkg-registry*` | `{}` | Whole-program registry of every other lowered package's direct-callable exports. |
| `*wrapper-target-names*` | `:all` | Which fns get an exported wrapper. |
| `*export-name-overrides*` | `nil` | Per-namespace overrides for exported Go names. |
| `*deftype-ctor-types*` | `nil` | Bound around a `:go` typeinfer pass to type constructor calls. |

## Per-compile State

These are `nil`/empty-initialized and rebound by the pipeline as it runs. They are listed for discoverability; setting them by hand is not a supported configuration surface.

| Var | Location | Role |
|---|---|---|
| `*current-fn*`, `*current-inst*`, `*current-zip*` | `passes.lg:23-25` | Current traversal cursor (fn / instruction / zipper). |
| `*inline-registry*` | `passes/inline.lg:20` | Inline-candidate registry for the inline pass. |
| `*lowered-registry*` | `lower_go.lg:1342` | Registry of lowered namespaces / fns for cross-ns direct-call lowering. |
| `*native-imports-used*` | `lower_go.lg:1352` | Go imports referenced by the fn currently being emitted. |
| `*cross-ns-vars-used*` | `lower_go.lg:1371` | Cross-ns var references collected during emission. |
| `*call-err-used*` | `lower_go.lg:1645` | Whether the emitted fn body needs the `callErr` plumbing. |
| `*typed-call-temps*` | `lower_go.lg:1657` | Temp bindings for typed direct calls in the current fn. |
| `*closure-arg-prefix*` | `lower_go.lg:64` | Prefix disambiguating closure-local arg names. |
| `*force-needs-error*` | `lower_go.lg:2465` | Forces error plumbing on for a body regardless of inference. |
| `*deftype-ctors*` | `lower_go.lg:1906` | Deftype constructors in scope for native ctor-call emission. |
| `*protocol-methods*` | `lower_go.lg:1935` | Protocol method table for devirtualized dispatch. |
| `*protocol-method-sigs*` | `lower_go.lg:1943` | Protocol method signatures. |
| `*defmulti-dispatchers*` | `lower_go.lg:2056` | Type-dispatched `defmulti`/`defmethod` tables. |
| `*ti-counters*` | `lattice.lg:110` | Typeinfer instrumentation counters. |

## Verifying a Var Change

The vars above decide which forms convert to IR and how they lower, so flipping a default (or adding a knob) is a coverage change until proven otherwise. `scripts/ir-stress.lg` is the harness that measures it.

The checks, cheapest first:

| Command | What it answers |
|---|---|
| `make ir-stress` | Native-lowering pass rate over the committed corpus allow-list. |
| `make ir-stress-gate` | Same census, ratcheted against `docs/perf/ir-stress-baseline.edn`. |
| `make jank-stress` | Coverage over the vendored jank Clojure-compat suite. |
| `make parity-full` | Both ir-stress modes plus clojure-test-suite. |

## Notes

- Only `*ir-compile*` and `*target*` are ordinarily set by callers; the rest are pass-internal defaults that tests and the whole-program driver rebind.
- Coverage claims about any of these vars should cite an ir-stress run.
- `*strict-structured?*` is the only var with an environment seed (`LG_STRICT_STRUCTURED`); everything else is `binding` / `set!` only.
- Line numbers are anchors, not contracts — grep the var name if a file has drifted since `last-verified`.

````

</details>

<details>
<summary>❌ failed &nbsp; <b>google/gemma-3-12b-it</b> &nbsp; <code>$0.0011</code> &nbsp; 7/7 &nbsp; <code>concepts/ir-dynamic-vars.md</code></summary>

**Validator rejected it:**
- `concepts/ir-dynamic-vars.md: unknown tag 'ir' (not in taxonomy)`

````markdown
---
type: Concept
category: concept
title: "IR Pipeline Dynamic Vars"
description: "This page documents the dynamic variables that control the let-go IR pipeline."
tags: [ir, vm, compiler, concept]
resource: "https://github.com/nooga/let-go/tree/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: nooga/let-go docs/design/ir-dynamic-vars.md, 2026-09-11"]
created: "2026-09-11"
updated: "2026-09-11"
status: speculative
---

The IR compile/lowering pipeline is configured almost entirely through
`^:dynamic` vars rather than flags or options. They are scattered across
`core.lg`, `passes/pipeline.lg`, `passes/inline.lg`, `passes/fusion.lg`,
`passes/typeinfer.lg`, and `lower_go.lg`, and only one (`*strict-structured?*`)
has an environment seed — the rest are set with `binding` / `set!`. This page
is the single index.

Two kinds of var live here:

- **Knobs** — you set them (via `binding` or `set!`) to change what the
  pipeline does. Documented with defaults and semantics below.
- **Per-compile state** — `nil`/empty-initialized vars the pipeline rebinds
  as it runs. Listed at the end so they're discoverable, but they are not
  settings; binding them by hand will usually just break a compile.

For build tags, environment variables, and `lgbgen` CLI flags, see the
companion sections in the guide; this page covers the dynamic vars only.

Every knob here changes what the pipeline converts to IR (or how it lowers
it), so a change to one is only as good as its coverage evidence. Measure it
with the ir-stress harness — see [Verifying a var change](#verifying-a-var-change)
at the end of this page and `scripts/ir-stress.md` for the harness itself.

## Compilation-mode control

The entry knobs that decide whether IR compilation runs at all and which
backend it targets.

| Var | Default | Semantics |
|---|---|---|
| `*ir-compile*` | `false` | Routes single-arity `defn`s through the IR-compile path instead of standard bytecode expansion (multi-arity and docstring-less edge cases still fall back). Enable with `(set! *ir-compile* true)` **after** `(require 'ir.passes.pipeline)` — it throws if the pipeline isn’t loaded. There is no environment variable for it. `core.lg:963` |
| `*ir-compile-verbose*` | `false` | When true, the `defn` macro logs a diagnostic (name + error) each time a fn falls back to bytecode. `core.lg:968` |
| `*ir-compile-fallback-log*` | `(atom [])` | Vector of `[name error-msg]` fallback records, populated while `*ir-compile-verbose*` is true. `core.lg:973` |
| `*target*` | `:bytecode` | `:go` makes `compile-form*` route through `ir.lower-go` (native Go) instead of `ir.lower` (bytecode). Bind it before calling `compile-form`. `passes/pipeline.lg:545` |

**Cost of `*ir-compile*`** — it is the one knob with a real amortization
story, and turning it on is not free. Pipeline load plus per-fn compile is a
one-time cost paid up front, so it pays back only when amortized *and* only on
allocation-bound work:

- **Alloc-heavy workloads** (persistent-map, transducers, `reduce`): ~5–12×
  fewer allocations, converting to roughly −13% wall-clock per run once heap
  churn dominates — break-even around 64 runs on persistent-map.
- **Compute-bound code** (`fib`, `loop`/`recur`): a small net loss that never
  breaks even; the compile cost has nothing to amortize against.
- Most of the win is the pipeline cutting per-element boxing, not fusion:
  `reduce` over `range` drops ~78% of allocations with no fusion applicable.

Figures from @mparrett's measurement on #555, via a gctrace GC-cycle proxy on
a single machine — directional, not a contract. `BenchmarkIRCompile`
(`pkg/ir/ir_compile_bench_test.go`) is the instrument to re-measure with, and
`make ir-stress` is what tells you whether a change moved *coverage* rather
than just cost.

## Pass toggles & tuning

| Var | Default | Semantics |
|---|---|---|
| `*enable-fusion*` | `true` | Transducer / deforestation fusion, placed after `cse`. On by default — measured ~20% fewer allocations across the ClojureTestSuite with backend parity and no ir-stress regression (`make ir-stress-gate`). `passes/fusion.lg:27` |
| `*enable-inline*` | `false` | Master switch for the inline pass. Opt-in: inlining supersedes the #345 direct-call path and still has rough edges (deftype-devirt codegen), so it stays off outside the AOT combinator measurement harness. Flipping it on is a coverage change — gate it with `make ir-stress-gate` and `make parity-full`. `passes/inline.lg:25` |
| `*max-unroll*` | `32` | Cap on fold-over-rest unrolling (ITER-0034). A combinator call with more than this many flat rest operands is left as a runtime call with a logged skip — never silently truncated — rather than unrolled into an oversized branch chain. Raising it trades compile time for code size; watch `:stress/timeout` buckets via `make ir-stress`. `passes/inline.lg:30` |
| `*typeinfer-max-drains*` | `2000000` | Backstop bound on the typeinfer fixpoint for pathological inputs (never fires on real code). The bail is sound — every assigned type is monotone and `lower-go`'s `rt.<Op>Value` path handles `:any` operands. Bind `nil` for unbounded. `passes/typeinfer.lg:494` |
| `*strict-structured?*` | `false` (seeded from `LG_STRICT_STRUCTURED`) | When true, structured-control-flow drift throws (and the caller falls the whole fn back to bytecode) instead of emitting a possibly mis-lowered `goto` body. Default off: the non-strict path stays correct via the coalesce-map interference fix; this just forbids the path. `lower_go.lg:2477` |
| `*direct-calls-disabled?*` | `false` | Forces every call through the cached-var / `InvokeValue` trampoline (which re-reads the var root each call) so runtime `alter-var-root` / `intern` overrides are observed. A baked direct call — `corefns.Count`, a lowered sibling’s Go func — would otherwise ignore them. `lower_go.lg:1863` |
| `*pass-trace*` | `nil` | Bind to an atom to capture per-pass instruction traces. `passes/trace.lg:39` |

## Cross-package / exported-wrapper control (`--target=go`)

Knobs for whole-program Go lowering, where one lowered package must call into
another. Defaults keep the committed lowered tree byte-identical until the
whole-program collector binds them.

| Var | Default | Semantics |
|---|---|---|
| `*emit-exported-wrappers*` | `false` | Emit an exported thin forwarding wrapper for each direct-callable lowered fn so it is reachable from another Go package. Off keeps bootstrap codegen byte-stable; flipped on by the collector and the T3 unit test. `passes/pipeline.lg:1065` |
| `*cross-pkg-registry*` | `{}` | Whole-program `{[internal-ns name arity] -> {:go-pkg <import> :go-name "LG_<go>" …}}` of every other lowered package’s direct-callable exports. Merged into the per-ns registry so a cross-package call resolves to `pkg.LG_<go>(ec, …)`. `{}` ⇒ no cross-package entries. `passes/pipeline.lg:1073` |
| `*wrapper-target-names*` | `:all` | Which fns get an exported wrapper. `:all` = every direct-callable fn (the single-ns convenience); a concrete set = exactly its members; an empty set = none. `lower-all-ns-to-go` always binds the concrete set, so a whole-program build with no cross-package references emits no dead exported API. `passes/pipeline.lg:1086` |
| `*export-name-overrides*` | `nil` | Per-namespace `{source-name -> resolved exported Go name}` bound around a namespace’s collect + lower passes. PascalCase is not injective, so this remaps the loser of any collision to a distinct name. `nil` = plain PascalCase (the collision-free case). `lower_go.lg:3387` |
| `*deftype-ctor-types*` | `nil` | `{constructor-name -> deftype-name-symbol}` bound around a `:go` typeinfer pass so a call to a known constructor — `(->Square 3)` — is typed `[:dtype Square]`, carrying the concrete receiver type to field access and devirtualized dispatch. `nil` = off, zero overhead. `passes/typeinfer.lg:50` |

## Per-compile state (not knobs)

These are `nil`/empty-initialized and rebound by the pipeline as it runs.
They are listed for discoverability; setting them by hand is not a supported
configuration surface.

| Var | Location | Role |
|---|---|---|
| `*current-fn*`, `*current-inst*`, `*current-zip*` | `passes.lg:23-25` | Current traversal cursor (fn / instruction / zipper). |
| `*inline-registry*` | `passes/inline.lg:20` | Inline-candidate registry for the inline pass. |
| `*lowered-registry*` | `lower_go.lg:1342` | Registry of lowered namespaces / fns for cross-ns direct-call lowering. |
| `*native-imports-used*` | `lower_go.lg:1352` | Go imports referenced by the fn currently being emitted. |
| `*cross-ns-vars-used*` | `lower_go.lg:1371` | Cross-ns var references collected during emission (feeds the cross-package collector). |
| `*call-err-used*` | `lower_go.lg:1645` | Whether the emitted fn body needs the `callErr` plumbing. |
| `*typed-call-temps*` | `lower_go.lg:1657` | Temp bindings for typed direct calls in the current fn. |
| `*closure-arg-prefix*` | `lower_go.lg:64` | Prefix disambiguating closure-local arg names (captured-name shadowing fix). |
| `*force-needs-error*` | `lower_go.lg:2465` | Forces error plumbing on for a body regardless of inference. |
| `*deftype-ctors*` | `lower_go.lg:1906` | Deftype constructors in scope for native ctor-call emission. |
| `*protocol-methods*` | `lower_go.lg:1935` | Protocol method table for devirtualized dispatch. |
| `*protocol-method-sigs*` | `lower_go.lg:1943` | Protocol method signatures. |
| `*defmulti-dispatchers*` | `lower_go.lg:2056` | Type-dispatched `defmulti`/`defmethod` tables. |
| `*ti-counters*` | `lattice.lg:110` | Typeinfer instrumentation counters. |

## Verifying a var change

The vars above decide which forms convert to IR and how they lower, so
flipping a default (or adding a knob) is a coverage change until proven
otherwise. `scripts/ir-stress.lg` is the harness that measures it: it drives a
corpus of `.lg` sources through a chosen IR path and reports per-defn buckets
(`:ok`, `:missing-form/set!`, `:validate/no-term`, `:stress/timeout`, …) so a
conversion regression shows up as a bucket that moved, not as a vague slowdown.
Full harness reference — modes, env vars, bucket meanings — is in
`scripts/ir-stress.md`.

The checks, cheapest first:

| Command | What it answers |
|---|---|
| `make ir-stress` | Native-lowering pass rate over the committed corpus allow-list (`scripts/ir-stress-corpus.edn`). The everyday "did my knob drop coverage?" run. |
| `make ir-stress-gate` | Same census, ratcheted against `docs/perf/ir-stress-baseline.edn` — exits non-zero if native-lowering failures grew. This is the gate; the ratchet only tightens. |
| `make jank-stress` | Coverage over the vendored jank Clojure-compat suite, which reaches Clojure surface the internal corpus doesn’t (BigDecimal literals, multimethods). |
| `make parity-full` | Both ir-stress modes (`lower-go` and `ir-compile`) plus clojure-test-suite, run once untagged and once under `-tags gogen_ir`, comparing bucket-by-symbol. Catches a var whose effect differs between the two engines. |

Which mode maps to which claim:

- **`lower-go`** — AOT conversion coverage. Use for any var read by
  `lower_go.lg` or bound for `*target* :go` (`*emit-exported-wrappers*`,
  `*cross-pkg-registry*`, `*wrapper-target-names*`, `*export-name-overrides*`,
  `*deftype-ctor-types*`, `*strict-structured?*`, `*direct-calls-disabled?*`).
- **`ir-compile`** — eval-mode conversion, i.e. what users hit at load time
  under `*ir-compile*`. Slower, since it evals each defn.
- **`trace`** — one defn, per-pass timings. Reach for it when a tuning knob
  (`*typeinfer-max-drains*`, `*max-unroll*`) produces `:stress/timeout` and you
  need to know which pass is responsible before tuning blindly.

Practical recipe for a default flip: capture a census on both settings and diff
the bucket tallies rather than eyeballing pass counts.

```sh
LG_STRESS_LOG=/tmp/before.log make ir-stress
# flip the var's default (or bind it in the harness), rebuild
LG_STRESS_LOG=/tmp/after.log  make ir-stress
# bucket-level diff: any row that moved is a conversion change
diff <(cut -f3 /tmp/before.log | sort | uniq -c) \
     <(cut -f3 /tmp/after.log  | sort | uniq -c)
```

If the change intentionally moves coverage, rebaseline the ratchet with
`make ir-stress-rebaseline` (tool-maintained — never hand-edit the EDN), review
the diff, and commit it alongside the change that caused it. This is the
evidence `*enable-fusion*` cites above ("no ir-stress regression") and the
standard any new default should meet.

## Notes

- Only `*ir-compile*` and `*target*` are ordinarily set by callers; the rest
  are pass-internal defaults that tests and the whole-program driver rebind.
- Coverage claims about any of these vars should cite an ir-stress run — see
  [Verifying a var change](#verifying-a-var-change). `make ir-stress-gate` is
  the ratcheted form of that check.
- `*strict-structured?*` is the only var with an environment seed
  (`LG_STRICT_STRUCTURED`); everything else is `binding` / `set!` only.
- Line numbers are anchors, not contracts — grep the var name if a file has
  drifted since `last-verified`.

````

</details>

<details>
<summary>⚠️ model-proposed-nothing &nbsp; <b>qwen/qwen3-32b</b> &nbsp; <code>$0.0012</code> &nbsp; 0/7 &nbsp; no page</summary>

````markdown
*(this run produced no page)*
````

</details>

<details>
<summary>✅ passed &nbsp; <b>qwen/qwen3.5-9b</b> &nbsp; <code>$0.0017</code> &nbsp; 7/7 &nbsp; <code>sources/design-ir-dynamic-vars.md</code></summary>

````markdown
---
type: Concept
category: source
title: "IR Pipeline Dynamic Vars"
description: "Reference for dynamic variable knobs, per-compile state, and verification methods used to configure the IR compile/lowering pipeline"
tags: [compiler, bytecode, vm]
resource: "https://github.com/mparrett/let-go/blob/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: mparrett/let-go docs/design/ir-dynamic-vars.md, 2026-09-11"]
created: "2026-09-11"
updated: "2026-09-11"
status: speculative
---

# IR Pipeline Dynamic Vars

The IR compile/lowering pipeline is configured almost entirely through `^:dynamic` vars rather than flags or options. They are scattered across `core.lg`, `passes/pipeline.lg`, `passes/inline.lg`, `passes/fusion.lg`, `passes/typeinfer.lg`, and `lower_go.lg`, and only one (`*strict-structured?*`) has an environment seed — the rest are set with `binding` / `set!`. This page is the single index.

Two kinds of var live here:

- **Knobs** — you set them (via `binding` or `set!`) to change what the pipeline does. Documented with defaults and semantics below.
- **Per-compile state** — `nil`/empty-initialized vars the pipeline rebinds as it runs. Listed at the end so they're discoverable, but they are not settings; binding them by hand will usually just break a compile.

For build tags, environment variables, and `lgbgen` CLI flags, see the companion sections in the guide; this page covers the dynamic vars only.

Every knob here changes what the pipeline converts to IR (or how it lowers it), so a change to one is only as good as its coverage evidence. Measure it with the ir-stress harness — see [Verifying a var change](#verifying-a-var-change) at the end of this page and `scripts/ir-stress.md` for the harness itself.

## Compilation-mode control

The entry knobs that decide whether IR compilation runs at all and which backend it targets.

| Var | Default | Semantics |
|---|---|---|
| `*ir-compile*` | `false` | Routes single-arity `defn`s through the IR-compile path instead of standard bytecode expansion (multi-arity and docstring-less edge cases still fall back). Enable with `(set! *ir-compile* true)` **after** `(require 'ir.passes.pipeline)` — it throws if the pipeline isn't loaded. There is no environment variable for it. `core.lg:963` |
| `*ir-compile-verbose*` | `false` | When true, the `defn` macro logs a diagnostic (name + error) each time a fn falls back to bytecode. `core.lg:968` |
| `*ir-compile-fallback-log*` | `(atom [])` | Vector of `[name error-msg]` fallback records, populated while `*ir-compile-verbose*` is true. `core.lg:973` |
| `*target*` | `:bytecode` | `:go` makes `compile-form*` route through `ir.lower-go` (native Go) instead of `ir.lower` (bytecode). Bind it before calling `compile-form`. `passes/pipeline.lg:545` |

**Cost of `*ir-compile*`** — it is the one knob with a real amortization story, and turning it on is not free. Pipeline load plus per-fn compile is a one-time cost paid up front, so it pays back only when amortized *and* only on allocation-bound work:

- **Alloc-heavy workloads** (persistent-map, transducers, `reduce`): ~5–12× fewer allocations, converting to roughly −13% wall-clock per run once heap churn dominates — break-even around 64 runs on persistent-map.
- **Compute-bound code** (`fib`, `loop`/`recur`): a small net loss that never breaks even; the compile cost has nothing to amortize against.
- Most of the win is the pipeline cutting per-element boxing, not fusion: `reduce` over `range` drops ~78% of allocations with no fusion applicable.

Figures from @mparrett's measurement on #555, via a gctrace GC-cycle proxy on a single machine — directional, not a contract. `BenchmarkIRCompile` (`pkg/ir/ir_compile_bench_test.go`) is the instrument to re-measure with, and `make ir-stress` is what tells you whether a change moved *coverage* rather than just cost.

## Pass toggles & tuning

| Var | Default | Semantics |
|---|---|---|
| `*enable-fusion*` | `true` | Transducer / deforestation fusion, placed after `cse`. On by default — measured ~20% fewer allocations across the ClojureTestSuite with backend parity and no ir-stress regression (`make ir-stress-gate`). `passes/fusion.lg:27` |
| `*enable-inline*` | `false` | Master switch for the inline pass. Opt-in: inlining supersedes the #345 direct-call path and still has rough edges (deftype-devirt codegen), so it stays off outside the AOT combinator measurement harness. Flipping it on is a coverage change — gate it with `make ir-stress-gate` and `make parity-full`. `passes/inline.lg:25` |
| `*max-unroll*` | `32` | Cap on fold-over-rest unrolling (ITER-0034). A combinator call with more than this many flat rest operands is left as a runtime call with a logged skip — never silently truncated — rather than unrolled into an oversized branch chain. Raising it trades compile time for code size; watch `:stress/timeout` buckets via `make ir-stress`. `passes/inline.lg:30` |
| `*typeinfer-max-drains*` | `2000000` | Backstop bound on the typeinfer fixpoint for pathological inputs (never fires on real code). The bail is sound — every assigned type is monotone and `lower-go`'s `rt.<Op>Value` path handles `:any` operands. Bind `nil` for unbounded. `passes/typeinfer.lg:494` |
| `*strict-structured?*` | `false` (seeded from `LG_STRICT_STRUCTURED`) | When true, structured-control-flow drift throws (and the caller falls the whole fn back to bytecode) instead of emitting a possibly mis-lowered `goto` body. Default off: the non-strict path stays correct via the coalesce-map interference fix; this just forbids the path. `lower_go.lg:2477` |
| `*direct-calls-disabled?*` | `false` | Forces every call through the cached-var / `InvokeValue` trampoline (which re-reads the var root each call) so runtime `alter-var-root` / `intern` overrides are observed. A baked direct call — `corefns.Count`, a lowered sibling's Go func — would otherwise ignore them. `lower_go.lg:1863` |
| `*pass-trace*` | `nil` | Bind to an atom to capture per-pass instruction traces. `passes/trace.lg:39` |

## Cross-package / exported-wrapper control (`--target=go`)

Knobs for whole-program Go lowering, where one lowered package must call into another. Defaults keep the committed lowered tree byte-identical until the whole-program collector binds them.

| Var | Default | Semantics |
|---|---|---|
| `*emit-exported-wrappers*` | `false` | Emit an exported thin forwarding wrapper for each direct-callable lowered fn so it is reachable from another Go package. Off keeps bootstrap codegen byte-stable; flipped on by the collector and the T3 unit test. `passes/pipeline.lg:1065` |
| `*cross-pkg-registry*` | `{}` | Whole-program `{[internal-ns name arity] -> {:go-pkg <import> :go-name "LG_<go>" …}}` of every other lowered package's direct-callable exports. Merged into the per-ns registry so a cross-package call resolves to `pkg.LG_<go>(ec, …)`. `{}` ⇒ no cross-package entries. `passes/pipeline.lg:1073` |
| `*wrapper-target-names*` | `:all` | Which fns get an exported wrapper. `:all` = every direct-callable fn (the single-ns convenience); a concrete set = exactly its members; an empty set = none. `lower-all-ns-to-go` always binds the concrete set, so a whole-program build with no cross-package references emits no dead exported API. `passes/pipeline.lg:1086` |
| `*export-name-overrides*` | `nil` | Per-namespace `{source-name -> resolved exported Go name}` bound around a namespace's collect + lower passes. PascalCase is not injective, so this remaps the loser of any collision to a distinct name. `nil` = plain PascalCase (the collision-free case). `lower_go.lg:3387` |
| `*deftype-ctor-types*` | `nil` | `{constructor-name -> deftype-name-symbol}` bound around a `:go` typeinfer pass so a call to a known constructor — `(->Square 3)` — is typed `[:dtype Square]`, carrying the concrete receiver type to field access and devirtualized dispatch. `nil` = off, zero overhead. `passes/typeinfer.lg:50` |

## Per-compile state (not knobs)

These are `nil`/empty-initialized and rebound by the pipeline as it runs. They are listed for discoverability; setting them by hand is not a supported configuration surface.

| Var | Location | Role |
|---|---|---|
| `*current-fn*`, `*current-inst*`, `*current-zip*` | `passes.lg:23-25` | Current traversal cursor (fn / instruction / zipper). |
| `*inline-registry*` | `passes/inline.lg:20` | Inline-candidate registry for the inline pass. |
| `*lowered-registry*` | `lower_go.lg:1342` | Registry of lowered namespaces / fns for cross-ns direct-call lowering. |
| `*native-imports-used*` | `lower_go.lg:1352` | Go imports referenced by the fn currently being emitted. |
| `*cross-ns-vars-used*` | `lower_go.lg:1371` | Cross-ns var references collected during emission (feeds the cross-package collector). |
| `*call-err-used*` | `lower_go.lg:1645` | Whether the emitted fn body needs the `callErr` plumbing. |
| `*typed-call-temps*` | `lower_go.lg:1657` | Temp bindings for typed direct calls in the current fn. |
| `*closure-arg-prefix*` | `lower_go.lg:64` | Prefix disambiguating closure-local arg names (captured-name shadowing fix). |
| `*force-needs-error*` | `lower_go.lg:2465` | Forces error plumbing on for a body regardless of inference. |
| `*deftype-ctors*` | `lower_go.lg:1906` | Deftype constructors in scope for native ctor-call emission. |
| `*protocol-methods*` | `lower_go.lg:1935` | Protocol method table for devirtualized dispatch. |
| `*protocol-method-sigs*` | `lower_go.lg:1943` | Protocol method signatures. |
| `*defmulti-dispatchers*` | `lower_go.lg:2056` | Type-dispatched `defmulti`/`defmethod` tables. |
| `*ti-counters*` | `lattice.lg:110` | Typeinfer instrumentation counters. |

## Verifying a var change

The vars above decide which forms convert to IR and how they lower, so flipping a default (or adding a knob) is a coverage change until proven otherwise. `scripts/ir-stress.lg` is the harness that measures it: it drives a corpus of `.lg` sources through a chosen IR path and reports per-defn buckets (`:ok`, `:missing-form/set!`, `:validate/no-term`, `:stress/timeout`, …) so a conversion regression shows up as a bucket that moved, not as a vague slowdown. Full harness reference — modes, env vars, bucket meanings — is in `scripts/ir-stress.md`.

The checks, cheapest first:

| Command | What it answers |
|---|---|
| `make ir-stress` | Native-lowering pass rate over the committed corpus allow-list (`scripts/ir-stress-corpus.edn`). The everyday "did my knob drop coverage?" run. |
| `make ir-stress-gate` | Same census, ratcheted against `docs/perf/ir-stress-baseline.edn` — exits non-zero if native-lowering failures grew. This is the gate; the ratchet only tightens. |
| `make jank-stress` | Coverage over the vendored jank Clojure-compat suite, which reaches Clojure surface the internal corpus doesn't (BigDecimal literals, multimethods). |
| `make parity-full` | Both ir-stress modes (`lower-go` and `ir-compile`) plus clojure-test-suite, run once untagged and once under `-tags gogen_ir`, comparing bucket-by-symbol. Catches a var whose effect differs between the two engines. |

Which mode maps to which claim:

- **`lower-go`** — AOT conversion coverage. Use for any var read by `lower_go.lg` or bound for `*target* :go` (`*emit-exported-wrappers*`, `*cross-pkg-registry*`, `*wrapper-target-names*`, `*export-name-overrides*`, `*deftype-ctor-types*`, `*strict-structured?*`, `*direct-calls-disabled?*`).
- **`ir-compile`** — eval-mode conversion, i.e. what users hit at load time under `*ir-compile*`. Slower, since it evals each defn.
- **`trace`** — one defn, per-pass timings. Reach for it when a tuning knob (`*typeinfer-max-drains*`, `*max-unroll*`) produces `:stress/timeout` and you need to know which pass is responsible before tuning blindly.

Practical recipe for a default flip: capture a census on both settings and diff the bucket tallies rather than eyeballing pass counts.

```sh
LG_STRESS_LOG=/tmp/before.log make ir-stress
# flip the var's default (or bind it in the harness), rebuild
LG_STRESS_LOG=/tmp/after.log  make ir-stress
# bucket-level diff: any row that moved is a conversion change
diff <(cut -f3 /tmp/before.log | sort | uniq -c) \
     <(cut -f3 /tmp/after.log  | sort | uniq -c)
```

If the change intentionally moves coverage, rebaseline the ratchet with `make ir-stress-rebaseline` (tool-maintained — never hand-edit the EDN), review the diff, and commit it alongside the change that caused it. This is the evidence `*enable-fusion*` cites above ("no ir-stress regression") and the standard any new default should meet.

## Notes

- Only `*ir-compile*` and `*target*` are ordinarily set by callers; the rest are pass-internal defaults that tests and the whole-program driver rebind.
- Coverage claims about any of these vars should cite an ir-stress run — see [Verifying a var change](#verifying-a-var-change). `make ir-stress-gate` is the ratcheted form of that check.
- `*strict-structured?*` is the only var with an environment seed (`LG_STRICT_STRUCTURED`); everything else is `binding` / `set!` only.
- Line numbers are anchors, not contracts — grep the var name if a file has drifted since `last-verified`.

---

Added: `docs/design/ir-dynamic-vars.md` is arquitecturally meaningful — it documents a key configuration scheme for the IR pipeline, defines ~18 knobs with semantics and verification methods, and is referenced in the design docs index.

````

</details>

<details>
<summary>✅ passed &nbsp; <b>qwen/qwen3.6-35b-a3b</b> &nbsp; <code>$0.0093</code> &nbsp; 7/7 &nbsp; <code>concepts/ir-dynamic-vars.md</code></summary>

````markdown
---
type: Concept
category: concept
title: "IR Pipeline Dynamic Vars"
description: "Indexes the dynamic vars that configure and track state during the IR compile and lowering pipeline."
tags: [compiler, bytecode, tooling, concept]
resource: "https://github.com/nooga/let-go/tree/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: nooga/let-go docs/design/ir-dynamic-vars.md, 2026-09-11"]
created: "2026-09-11"
updated: "2026-09-11"
status: speculative
---

# IR Pipeline Dynamic Vars

The IR compile/lowering pipeline is configured almost entirely through `^:dynamic` vars rather than flags or environment variables. They are scattered across `core.lg`, `passes/pipeline.lg`, `passes/inline.lg`, `passes/fusion.lg`, `passes/typeinfer.lg`, and `lower_go.lg`, and only one (`*strict-structured?*`) has an environment seed — the rest are set with `binding` / `set!`. This page serves as the single index for these configuration and state variables.

Variables in this system fall into two categories:

- **Knobs** — settings you set (via `binding` or `set!`) to change what the pipeline does. Documented with defaults and semantics below.
- **Per-compile state** — `nil`/empty-initialized vars the pipeline rebinds as it runs. Listed at the end for discoverability; binding them by hand typically breaks a compile.

Every knob here changes what forms convert to IR (or how they lower it), so a change to one is only as good as its coverage evidence. The pipeline provides an `ir-stress` harness to measure these changes — see [Verifying a var change](#verifying-a-var-change) for the verification strategy.

## Compilation-mode control

These are the entry knobs that decide whether IR compilation runs at all and which backend it targets.

| Var | Default | Semantics |
|---|---|---|
| `*ir-compile*` | `false` | Routes single-arity `defn`s through the IR-compile path instead of standard bytecode expansion. Enable with `(set! *ir-compile* true)` **after** `(require 'ir.passes.pipeline)` — it throws if the pipeline isn't loaded. `core.lg:963` |
| `*ir-compile-verbose*` | `false` | When true, the `defn` macro logs a diagnostic (name + error) each time a fn falls back to bytecode. `core.lg:968` |
| `*ir-compile-fallback-log*` | `(atom [])` | Vector of `[name error-msg]` fallback records, populated while `*ir-compile-verbose*` is true. `core.lg:973` |
| `*target*` | `:bytecode` | `:go` makes `compile-form*` route through `ir.lower-go` (native Go) instead of `ir.lower` (bytecode). Bind it before calling `compile-form`. `passes/pipeline.lg:545` |

**Cost of `*ir-compile*`** — it is the one knob with a real amortization story. Pipeline load plus per-fn compile is a one-time cost paid up front, so it pays back only when amortized *and* only on allocation-bound work:

- **Alloc-heavy workloads** (persistent-map, transducers, `reduce`): ~5–12× fewer allocations, converting to roughly −13% wall-clock per run once heap churn dominates — break-even around 64 runs on persistent-map.
- **Compute-bound code** (`fib`, `loop`/`recur`): a small net loss that never breaks even; the compile cost has nothing to amortize against.
- Most of the win comes from the pipeline cutting per-element boxing, not fusion: `reduce` over `range` drops ~78% of allocations with no fusion applicable.

These figures represent directional measurements from a single machine via a GC-cycle proxy. `BenchmarkIRCompile` (`pkg/ir/ir_compile_bench_test.go`) is the instrument for re-measurement, and `make ir-stress` determines whether a change moved *coverage* rather than just cost.

## Pass toggles & tuning

| Var | Default | Semantics |
|---|---|---|
| `*enable-fusion*` | `true` | Transducer / deforestation fusion, placed after `cse`. On by default — measured ~20% fewer allocations across the ClojureTestSuite with backend parity and no ir-stress regression (`make ir-stress-gate`). `passes/fusion.lg:27` |
| `*enable-inline*` | `false` | Master switch for the inline pass. Opt-in: inlining supersedes the #345 direct-call path and still has rough edges (deftype-devirt codegen), so it stays off outside the AOT combinator measurement harness. Flipping it on is a coverage change — gate it with `make ir-stress-gate` and `make parity-full`. `passes/inline.lg:25` |
| `*max-unroll*` | `32` | Cap on fold-over-rest unrolling (ITER-0034). A combinator call with more than this many flat rest operands is left as a runtime call with a logged skip — never silently truncated — rather than unrolled into an oversized branch chain. Raising it trades compile time for code size; watch `:stress/timeout` buckets via `make ir-stress`. `passes/inline.lg:30` |
| `*typeinfer-max-drains*` | `2000000` | Backstop bound on the typeinfer fixpoint for pathological inputs (never fires on real code). The bail is sound — every assigned type is monotone and `lower-go`'s `rt.<Op>Value` path handles `:any` operands. Bind `nil` for unbounded. `passes/typeinfer.lg:494` |
| `*strict-structured?*` | `false` (seeded from `LG_STRICT_STRUCTURED`) | When true, structured-control-flow drift throws (and the caller falls the whole fn back to bytecode) instead of emitting a possibly mis-lowered `goto` body. Default off: the non-strict path stays correct via the coalesce-map interference fix; this just forbids the path. `lower_go.lg:2477` |
| `*direct-calls-disabled?*` | `false` | Forces every call through the cached-var / `InvokeValue` trampoline (which re-reads the var root each call) so runtime `alter-var-root` / `intern` overrides are observed. A baked direct call — `corefns.Count`, a lowered sibling's Go func — would otherwise ignore them. `lower_go.lg:1863` |
| `*pass-trace*` | `nil` | Bind to an atom to capture per-pass instruction traces. `passes/trace.lg:39` |

## Cross-package / exported-wrapper control (`--target=go`)

Knobs for whole-program Go lowering, where one lowered package must call into another. Defaults keep the committed lowered tree byte-identical until the whole-program collector binds them.

| Var | Default | Semantics |
|---|---|---|
| `*emit-exported-wrappers*` | `false` | Emit an exported thin forwarding wrapper for each direct-callable lowered fn so it is reachable from another Go package. Off keeps bootstrap codegen byte-stable; flipped on by the collector and the T3 unit test. `passes/pipeline.lg:1065` |
| `*cross-pkg-registry*` | `{}` | Whole-program `{[internal-ns name arity] -> {:go-pkg <import> :go-name "LG_<go>" …}}` of every other lowered package's direct-callable exports. Merged into the per-ns registry so a cross-package call resolves to `pkg.LG_<go>(ec, …)`. `{}` ⇒ no cross-package entries. `passes/pipeline.lg:1073` |
| `*wrapper-target-names*` | `:all` | Which fns get an exported wrapper. `:all` = every direct-callable fn (the single-ns convenience); a concrete set = exactly its members; an empty set = none. `lower-all-ns-to-go` always binds the concrete set, so a whole-program build with no cross-package references emits no dead exported API. `passes/pipeline.lg:1086` |
| `*export-name-overrides*` | `nil` | Per-namespace `{source-name -> resolved exported Go name}` bound around a namespace's collect + lower passes. PascalCase is not injective, so this remaps the loser of any collision to a distinct name. `nil` = plain PascalCase (the collision-free case). `lower_go.lg:3387` |
| `*deftype-ctor-types*` | `nil` | `{constructor-name -> deftype-name-symbol}` bound around a `:go` typeinfer pass so a call to a known constructor — `(->Square 3)` — is typed `[:dtype Square]`, carrying the concrete receiver type to field access and devirtualized dispatch. `nil` = off, zero overhead. `passes/typeinfer.lg:50` |

## Per-compile state (not knobs)

These are `nil`/empty-initialized and rebound by the pipeline as it runs. They are listed for discoverability; setting them by hand is not a supported configuration surface.

| Var | Location | Role |
|---|---|---|
| `*current-fn*`, `*current-inst*`, `*current-zip*` | `passes.lg:23-25` | Current traversal cursor (fn / instruction / zipper). |
| `*inline-registry*` | `passes/inline.lg:20` | Inline-candidate registry for the inline pass. |
| `*lowered-registry*` | `lower_go.lg:1342` | Registry of lowered namespaces / fns for cross-ns direct-call lowering. |
| `*native-imports-used*` | `lower_go.lg:1352` | Go imports referenced by the fn currently being emitted. |
| `*cross-ns-vars-used*` | `lower_go.lg:1371` | Cross-ns var references collected during emission (feeds the cross-package collector). |
| `*call-err-used*` | `lower_go.lg:1645` | Whether the emitted fn body needs the `callErr` plumbing. |
| `*typed-call-temps*` | `lower_go.lg:1657` | Temp bindings for typed direct calls in the current fn. |
| `*closure-arg-prefix*` | `lower_go.lg:64` | Prefix disambiguating closure-local arg names (captured-name shadowing fix). |
| `*force-needs-error*` | `lower_go.lg:2465` | Forces error plumbing on for a body regardless of inference. |
| `*deftype-ctors*` | `lower_go.lg:1906` | Deftype constructors in scope for native ctor-call emission. |
| `*protocol-methods*` | `lower_go.lg:1935` | Protocol method table for devirtualized dispatch. |
| `*protocol-method-sigs*` | `lower_go.lg:1943` | Protocol method signatures. |
| `*defmulti-dispatchers*` | `lower_go.lg:2056` | Type-dispatched `defmulti`/`defmethod` tables. |
| `*ti-counters*` | `lattice.lg:110` | Typeinfer instrumentation counters. |

## Verifying a var change

The vars above decide which forms convert to IR and how they lower, so flipping a default (or adding a knob) is a coverage change until proven otherwise. `scripts/ir-stress.lg` is the harness that measures it: it drives a corpus of `.lg` sources through a chosen IR path and reports per-defn buckets (`:ok`, `:missing-form/set!`, `:validate/no-term`, `:stress/timeout`, …) so a conversion regression shows up as a bucket that moved, not as a vague slowdown. See `scripts/ir-stress.md` for the harness reference.

The checks, cheapest first:

| Command | What it answers |
|---|---|
| `make ir-stress` | Native-lowering pass rate over the committed corpus allow-list (`scripts/ir-stress-corpus.edn`). The everyday "did my knob drop coverage?" run. |
| `make ir-stress-gate` | Same census, ratcheted against `docs/perf/ir-stress-baseline.edn` — exits non-zero if native-lowering failures grew. This is the gate; the ratchet only tightens. |
| `make jank-stress` | Coverage over the vendored jank Clojure-compat suite, which reaches Clojure surface the internal corpus doesn't (BigDecimal literals, multimethods). |
| `make parity-full` | Both ir-stress modes (`lower-go` and `ir-compile`) plus clojure-test-suite, run once untagged and once under `-tags gogen_ir`, comparing bucket-by-symbol. Catches a var whose effect differs between the two engines. |

Which mode maps to which claim:

- **`lower-go`** — AOT conversion coverage. Use for any var read by `lower_go.lg` or bound for `*target* :go` (`*emit-exported-wrappers*`, `*cross-pkg-registry*`, `*wrapper-target-names*`, `*export-name-overrides*`, `*deftype-ctor-types*`, `*strict-structured?*`, `*direct-calls-disabled?*`).
- **`ir-compile`** — eval-mode conversion, i.e. what users hit at load time under `*ir-compile*`. Slower, since it evals each `defn`.
- **`trace`** — one `defn`, per-pass timings. Reach for it when a tuning knob (`*typeinfer-max-drains*`, `*max-unroll*`) produces `:stress/timeout` and you need to know which pass is responsible before tuning blindly.

### Practical verification recipe

To verify a default flip, capture a census on both settings and diff the bucket tallies rather than eyeballing pass counts:

```sh
LG_STRESS_LOG=/tmp/before.log make ir-stress
# flip the var's default (or bind it in the harness), rebuild
LG_STRESS_LOG=/tmp/after.log  make ir-stress
# bucket-level diff: any row that moved is a conversion change
diff <(cut -f3 /tmp/before.log | sort | uniq -c) \
     <(cut -f3 /tmp/after.log  | sort | uniq -c)
```

If the change intentionally moves coverage, rebaseline the ratchet with `make ir-stress-rebaseline` (tool-maintained — never hand-edit the EDN), review the diff, and commit it alongside the change that caused it. This is the evidence `*enable-fusion*` cites above ("no ir-stress regression") and the standard any new default should meet.

## Notes

- Only `*ir-compile*` and `*target*` are ordinarily set by callers; the rest are pass-internal defaults that tests and the whole-program driver rebind.
- Coverage claims about any of these vars should cite an `ir-stress` run — see [Verifying a var change](#verifying-a-var-change). `make ir-stress-gate` is the ratcheted form of that check.
- `*strict-structured?*` is the only var with an environment seed (`LG_STRICT_STRUCTURED`); everything else is `binding` / `set!` only.
- Line numbers are anchors, not contracts — grep the var name if a file has drifted since `last-verified`.

````

</details>

<details>
<summary>✅ passed &nbsp; <b>anthropic/claude-sonnet-5</b> &nbsp; <code>$0.0648</code> &nbsp; 7/7 &nbsp; <code>sources/design-ir-dynamic-vars.md</code></summary>

````markdown
---
type: Reference
category: source
title: "Source: IR pipeline dynamic vars"
description: "Summary of docs/design/ir-dynamic-vars.md, the index of `^:dynamic` vars that configure the IR compile/lowering pipeline and the ir-stress harness used to verify a change to one"
tags: [compiler, bytecode, vm, runtime, reference]
resource: "https://github.com/mparrett/let-go/blob/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: mparrett/let-go docs/design/ir-dynamic-vars.md, 2026-09-11"]
created: "2026-09-11"
updated: "2026-09-11"
status: speculative
---

# Source: IR pipeline dynamic vars

`docs/design/ir-dynamic-vars.md` is the canonical index of the `^:dynamic`
vars that configure let-go's IR compile/lowering pipeline (see
[the IR pipeline](../concepts/ir-pipeline.md) and
[IR passes](../concepts/ir-passes.md)). The design choice it documents: the
pipeline exposes almost no flags or options structs — configuration lives in
scattered `^:dynamic` vars across `core.lg`, `passes/pipeline.lg`,
`passes/inline.lg`, `passes/fusion.lg`, `passes/typeinfer.lg`, and
`lower_go.lg`, set via `binding`/`set!`. Only one var,
`*strict-structured?*`, has an environment-variable seed
(`LG_STRICT_STRUCTURED`); everything else is process-local.

## Two kinds of var

- **Knobs** — user-facing settings with documented defaults and semantics:
  compilation-mode control (`*ir-compile*`, `*ir-compile-verbose*`,
  `*ir-compile-fallback-log*`, `*target*`), pass toggles/tuning
  (`*enable-fusion*`, `*enable-inline*`, `*max-unroll*`,
  `*typeinfer-max-drains*`, `*strict-structured?*`,
  `*direct-calls-disabled?*`, `*pass-trace*`), and cross-package/exported-
  wrapper control for `--target=go` whole-program lowering
  (`*emit-exported-wrappers*`, `*cross-pkg-registry*`,
  `*wrapper-target-names*`, `*export-name-overrides*`,
  `*deftype-ctor-types*`).
- **Per-compile state** — `nil`/empty-initialized vars the pipeline rebinds
  as it runs (traversal cursors, registries, `*ti-counters*`, etc.). Not a
  supported configuration surface; documented purely for discoverability.

## The `*ir-compile*` cost note

The source adds an amortization analysis for `*ir-compile*` (routing
single-arity `defn`s through IR-compile instead of bytecode expansion),
attributed to @mparrett's measurement on PR #555 via a gctrace GC-cycle
proxy on a single machine:

- Alloc-heavy workloads (persistent-map, transducers, `reduce`): ~5–12×
  fewer allocations, ~−13% wall-clock per run once heap churn dominates,
  break-even around 64 runs on persistent-map.
- Compute-bound code (`fib`, `loop`/`recur`): a small net loss that never
  breaks even.
- Most of the win is per-element boxing reduction, not fusion — `reduce`
  over `range` drops ~78% of allocations with no fusion applicable.

`BenchmarkIRCompile` (`pkg/ir/ir_compile_bench_test.go`) is named as the
instrument to re-measure this with; the figures are explicitly flagged as
directional, not a contract.

## Verifying a var change

Because every knob decides what converts to IR or how it lowers, the source
treats a change to any of them as a coverage change until measured with the
`scripts/ir-stress.lg` harness (per-defn buckets like `:ok`,
`:missing-form/set!`, `:validate/no-term`, `:stress/timeout`). It documents:

- `make ir-stress` — everyday native-lowering pass-rate check.
- `make ir-stress-gate` — same census ratcheted against
  `docs/perf/ir-stress-baseline.edn`; the gate, tightening-only.
- `make jank-stress` — coverage against the vendored jank Clojure-compat
  suite.
- `make parity-full` — both ir-stress modes (`lower-go`, `ir-compile`) plus
  clojure-test-suite, untagged and under `-tags gogen_ir`, diffed bucket by
  symbol.

It maps modes to claims — `lower-go` for AOT conversion coverage (the vars
read by `lower_go.lg` / `*target* :go`), `ir-compile` for eval-mode
conversion at load time, and `trace` for per-pass timing when a tuning knob
produces `:stress/timeout` — and gives a before/after bucket-diff recipe over
the `LG_STRESS_LOG` TSV. An intentional coverage move is rebaselined with
`make ir-stress-rebaseline` (tool-maintained; never hand-edited).

## Why this matters architecturally

This document formalizes a design pattern used throughout the compiler: pass
behavior is controlled by dynamic-var binding rather than a config object
threaded through the pipeline, which lets `binding`/`set!` scope a knob to a
single compile without touching call signatures — but it also means every
knob is a silent, global, coverage-affecting switch with no static
enforcement, which is why the source pairs each knob with an explicit
verification recipe (`make ir-stress[-gate]`) rather than trusting the
default alone.

See also: [IR pipeline](../concepts/ir-pipeline.md), [IR passes](../concepts/ir-passes.md), [IR optimizations](../concepts/ir-optimizations.md), [type inference](../concepts/type-inference.md).

````

</details>

## Caveats

One run per model, no temperature or seed control. An earlier sweep at a lower
output cap gave different verdicts for three of the nine, so treat any single
page as one sample of that model's behaviour rather than its standard. Marker
counts and validator verdicts are proxies; the prose above is the evidence.

Raw logs and both sweeps: [`devlog/model-sweep/`](.) in the trial repository.
