---
name: algo-craft
description: In-place algorithmic work on a narrow target — add a new algorithm, improve an existing one, or evaluate that it should be left alone, scoped to a function, class, or single file. Linear and self-contained — state the target's contract, complexity, allocation pattern, realistic N, and hot/cold; record a baseline benchmark (realistic shapes at small/typical/large sizes, median + p99 + max, allocation counts, machine profile) before any candidate exists; design at most a few candidates and self-refute them on paper; implement the survivor in place behind red-first tests; keep an improvement only if it beats the baseline outside noise, accept an addition only when its measured numbers match its promised complexity. Hot paths obey realtime rules — zero steady-state allocations, no hidden allocators, bounded worst case, no blocking. No subagents, no research fan-out, no parallel implementations; edits land in the existing files. Trigger on "algo-craft", "/algo-craft", "add an algorithm", "implement an algorithm here", "improve this algorithm", "tune this function", "optimize this function in place", "in-place algorithmic work".
---

# algo-craft

Benchmark-gated algorithmic work on one named target, edited in place. Three jobs, one discipline:

- **Add** — implement a new algorithm where none exists.
- **Improve** — make an existing one measurably better.
- **Evaluate** — conclude it should be left alone; "no work needed" is a first-class outcome, not a failure.

Creed: **an algorithm's performance does not exist until a benchmark says it does.** Every claim in the run — "this is faster", "this meets the budget", "this allocates nothing" — is either backed by a recorded number or does not get made.

**Scope rule:** the target is the function / class / file the user names (or the code under discussion). If the work reveals a whole-subsystem redesign — many interacting components, a data flow that must change end to end — stop and report that the scope has outgrown this skill; never grow the run silently.

## Steps

1. **SCOPE.** State the job (add / improve), the target and its contract (inputs, outputs, invariants callers rely on), CPU + memory complexity and allocation pattern of any existing code, realistic input shapes and sizes, and hot or cold. If realistic N is small enough that nothing can matter, say so and stop. At most one clarifying question, and only if the target or the realistic input shapes are genuinely ambiguous; "autonomous" in the request → zero questions.
2. **BASELINE.** Build green; tests touching the target pass — nothing can be baselined on red. Set up a micro-benchmark: the project's harness if it has one, else a minimal timing loop, driving realistic shapes at small / typical / large sizes. Capture median + p99 + max latency, allocation counts and peak memory where the language exposes them cheaply, and the machine profile (CPU, cores, caches) — numbers are meaningless without it. Improving: benchmark the current implementation; that number is the bar every candidate must beat. Adding: record the performance expectation instead — promised complexity at realistic N and any hot-path budget; the bench exists before the code it will judge. No candidate exists before this step is done.
3. **DESIGN.** At most 3 candidates, one line each: idea, expected win or behavior, complexity cost. A library already installed in the project is a valid candidate; adding a new dependency is out of scope here. Self-refute each on paper against realistic N, constant factors (cache, branches, allocations), the realtime rules where the path is hot, and simplicity; drop anything that can't plausibly pay for its complexity. Adding: the simplest candidate that meets the requirement is the default winner.
4. **TEST.** Tests pinning the contract — main flow plus boundary values — exist or are written now, and are watched red wherever the behavior is new or untested. No implementation before its tests.
5. **IMPLEMENT.** Land the surviving candidate **in place**, in the existing files, to green. No dual paths, no flags preserving an old implementation.
6. **MEASURE.** Rerun the benchmark and judge by the Adoption rules below. Improving: no adoption → revert; a reverted improvement is a successful evaluation. Adding: the measured numbers must match the promised complexity and any hot-path budget; a miss → redesign, or report plainly what the code actually delivers.
7. **REPORT.** The numbers (before/after, or measured curve against the promise), what changed, candidates considered and why the rest died, and the exact command to rerun the bench.

## Realtime rules (hot paths only)

Hot = on the per-item / per-frame / per-request path, tagged at SCOPE. Cold paths are exempt and judged on simplicity + throughput only.

1. **Zero allocations steady-state.** Memory acquired at init — preallocation, pools, reused scratch. Warmup allocation is fine; per-iteration allocation (including GC pressure in managed languages) is a defect. Where the language exposes allocation counts, the hot-path benchmark asserts 0 — nonzero fails regardless of speed.
2. **No hidden allocators:** closures/boxing, string temporaries, container growth/rehash, exception machinery — the toolkit's Hot-path list when one exists.
3. **Bounded worst case beats better average:** pre-reserve to known bounds; no amortized-only guarantees where the spike lands on the hot path.
4. **No blocking:** no lock waits, syscalls, I/O, or logging on the hot path.
5. **Errors without unwinding or allocating** — value/status returns; exceptions/panics are setup/teardown territory.
6. **Layout matches access pattern:** contiguous over node-based.

Latency is judged median + p99 + max — winning median while regressing max is a loss.

## Adoption rules

One principle: the burden of proof grows with the complexity the code adds. Assign the complexity class yourself — simpler/same, more complex, or hack — and record it with the verdict and a one-line rationale; the rationale is the gate.

- Below run-to-run noise is not a win — that's measurement, not judgment.
- Simpler or same complexity: any real win adopts; a tie adopts only if the code got simpler. A library already in the project counts as simpler — complexity we don't maintain doesn't count against us.
- More complex: the win must be worth maintaining the complexity — a modest win can justify modest complexity on a hot path; a cold path almost never earns added complexity.
- Hack (fragile cleverness, UB-adjacent tricks, layout gymnastics): only an exceptional, measured win on a genuinely hot path, contained behind a clean interface and commented as such.
- Hot paths: a max/p99 regression weighs heavily against adoption; new steady-state allocations are a loss regardless of median.
- Doubt → the original wins (or, when adding, the simpler candidate does). A rationale you'd be embarrassed to read back is a rejection.

The same proportionality picks between candidates for an addition: extra complexity must buy a measured, needed win over the simpler option.

## Toolkit

If a `<lang>-toolkit` skill exists for the target language (for example msvcpp-toolkit), borrow its Benchmarking and Hot-path sections; if not, do not research one — proceed with the minimal harness.
