---
type: Reference
category: source
title: "IR pipeline dynamic vars"
description: "Reference index of the ^:dynamic vars that configure the IR compile/lowering pipeline, plus the ir-stress recipe for verifying a knob change"
tags: [compiler, bytecode, vm, reference]
resource: "https://github.com/nooga/let-go/blob/main/docs/design/ir-dynamic-vars.md"
sources: ["repo: nooga/let-go docs/design/ir-dynamic-vars.md, 2026-09-10"]
created: "2026-09-10"
updated: "2026-09-10"
status: speculative
---

# IR pipeline dynamic vars

This is a wiki-side pointer to `docs/design/ir-dynamic-vars.md`, the
in-repo reference for how let-go's IR compile/lowering pipeline is
configured. The pipeline has almost no flags or options object — it's
driven by `^:dynamic` vars scattered across `core.lg`, `passes/pipeline.lg`,
`passes/inline.lg`, `passes/fusion.lg`, `passes/typeinfer.lg`, and
`lower_go.lg`. Only `*strict-structured?*` has an environment seed
(`LG_STRICT_STRUCTURED`); everything else is set via `binding` / `set!`.

For the pipeline's architecture and pass ordering, see
[ir-pipeline](../concepts/ir-pipeline.md) and
[ir-passes](../concepts/ir-passes.md); this source page is about the
*configuration surface* of that pipeline, not its structure.

## Two kinds of var

- **Knobs** — set to change pipeline behavior: whether IR compilation runs
  at all (`*ir-compile*`), which backend it targets (`*target*`), pass
  toggles (`*enable-fusion*`, `*enable-inline*`), tuning bounds
  (`*max-unroll*`, `*typeinfer-max-drains*`), and cross-package /
  exported-wrapper controls for `--target=go` whole-program builds.
- **Per-compile state** — `nil`/empty-initialized vars the pipeline rebinds
  as it runs (traversal cursors, registries, per-fn flags). Not a supported
  configuration surface; documented only for discoverability.

## Notable claims from the source

- **`*ir-compile*` has a real cost/amortization story.** Turning it on pays
  a one-time pipeline-load-plus-compile cost that only amortizes on
  allocation-bound workloads: ~5–12× fewer allocations and roughly −13%
  wall-clock per run on alloc-heavy code (persistent-map, transducers,
  `reduce`), break-even around 64 runs — versus a small net loss on
  compute-bound code (`fib`, `loop`/`recur`) that never breaks even. Most of
  the win is per-element boxing reduction, not fusion. Figures are from a
  single-machine gctrace GC-cycle proxy (directional, not a contract);
  `BenchmarkIRCompile` (`pkg/ir/ir_compile_bench_test.go`) is named as the
  instrument to re-measure with.
- **Every knob is a coverage decision, not just a behavior toggle.** Because
  each var decides what converts to IR or how it lowers, the source
  document treats a default flip as something to be measured, not assumed
  safe.

## Verifying a var change (ir-stress)

The source documents `scripts/ir-stress.lg` as the instrument: it drives a
corpus of `.lg` sources through a chosen IR path and reports per-defn
buckets (`:ok`, `:missing-form/set!`, `:validate/no-term`,
`:stress/timeout`, …), so a coverage regression shows up as a bucket that
moved rather than a vague slowdown.

| Command | What it answers |
|---|---|
| `make ir-stress` | Pass rate over the committed corpus allow-list — the everyday check. |
| `make ir-stress-gate` | Same census, ratcheted against `docs/perf/ir-stress-baseline.edn`; the ratchet only tightens. |
| `make jank-stress` | Coverage over the vendored jank Clojure-compat suite. |
| `make parity-full` | Both `lower-go` and `ir-compile` modes plus clojure-test-suite, tagged and untagged, diffed bucket-by-symbol. |

Mode-to-claim mapping: `lower-go` = AOT conversion coverage (vars read by
`lower_go.lg` or bound under `*target* :go`); `ir-compile` = eval-mode
conversion coverage (what users hit at load time under `*ir-compile*`);
`trace` = per-pass timings for one defn, useful when a tuning knob produces
`:stress/timeout`.

The recipe for a default flip is a before/after bucket diff over the
`LG_STRESS_LOG` TSV, and an intentional coverage move is rebaselined with
`make ir-stress-rebaseline` (tool-maintained, never hand-edited) and
committed alongside the change.

## Related pages

- [ir-pipeline](../concepts/ir-pipeline.md) — pipeline architecture these vars configure
- [ir-passes](../concepts/ir-passes.md) — the passes several knobs toggle (fusion, inline)
- [ir-optimizations](../concepts/ir-optimizations.md) — what fusion/inline buy in practice
- [type-inference](../concepts/type-inference.md) — the typeinfer pass bounded by `*typeinfer-max-drains*`
