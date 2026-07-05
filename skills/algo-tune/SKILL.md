---
name: algo-tune
description: Scaled-down, in-place sibling of algo-rewrite for small targets — a function, class, or single file. Evaluate the algorithm (complexity, allocations, realistic N), record a baseline benchmark, design at most a few candidates, self-refute them on paper, implement the survivor in place behind red-first tests, and keep it only if the re-measured benchmark beats the baseline outside noise — otherwise revert. Linear, no subagents, no research fan-out, no v2 sibling tree, no gate chain; edits land in the existing files. Same creed as algo-rewrite — an optimization does not exist until a benchmark says it does. Trigger on "algo-tune", "/algo-tune", "tune this algorithm", "tune this function", "optimize this function in place", "small algorithmic rewrite", "in-place algorithm rewrite", "evaluate and rewrite this algorithm".
---

# algo-tune

Scaled-down algo-rewrite for one small, named target. Same creed: **an optimization does not exist until a benchmark says it does.** Everything else is proportionally smaller — in-place edits, one linear thread, no agents, no pipeline.

**Scope rule:** the target is the function / class / file the user names (or the code under discussion). If evaluation reveals it is really whole-subsystem work — many interacting components, a data flow that must be redesigned end to end — stop and recommend `/algo-rewrite`; never grow this run into one.

## Steps

1. **BASELINE.** Build green, tests touching the target pass. Write a micro-benchmark with realistic shapes and sizes — use a benchmark harness already in the project, else a minimal timing loop; capture median + max, and allocation counts where the language exposes them cheaply. No candidate exists before this number does.
2. **EVAL.** Read the target; state its algorithm, CPU + memory complexity, allocation pattern, realistic N, hot or cold. If realistic N is small enough that nothing can matter, stop and say so — "no rewrite needed" is a first-class outcome, not a failure.
3. **DESIGN.** At most 3 candidates, one line each: idea, expected win, complexity cost. "Use a library already installed in the project" is a valid candidate; adding a new dependency is out of scope here. Self-refute each against realistic N, constant factors (cache, branches, allocations), and simplicity; drop anything that can't plausibly pay for its complexity.
4. **REWRITE.** For the survivor: tests pinning the target's contract must exist or be written first (main flow + boundary values, watched red where the behavior was untested), then implement **in place** to green. No dual paths, no `use_old_impl` flags.
5. **MEASURE.** Rerun the benchmark. Keep only a win outside run-to-run noise and proportional to the complexity added — doubt goes to the original. Anything else → revert; a reverted rewrite is a successful evaluation.
6. **REPORT.** Before/after numbers, what changed, candidates considered and why the rest died, and the exact command to rerun the bench.

## Rules

Carried from algo-rewrite: red-first (test or baseline before the code it judges); proportionality (added complexity must be bought by a measured win); on hot paths, new steady-state allocations and max-latency regressions count as losses regardless of median.

Deliberately dropped: the gate chain, ASK batch, refute/edge/parity agents, v2 sibling tree, four-lens triage, library research, and toolkit derivation. If a `<lang>-toolkit` skill exists, borrow its Benchmarking and Hot-path sections; if not, do not research one — proceed with the minimal harness.

Questions: at most one, and only if the target or the realistic input shapes are genuinely ambiguous. "Autonomous" in the request → zero questions.
