---
name: algo-craft
description: In-place algorithmic work on a narrow target — add a new algorithm, improve an existing one, refactor to the same intent, or evaluate that it should be left alone, scoped to a function, class, or single file. Twin creeds — performance does not exist until a benchmark says it does; correctness does not exist until it survives adversarial refutation. Linear spine — SCOPE (job, contract with guarantees and sync contract, correctness-lens risk scan, tags HOT/COLD CONCURRENT TRUST-BOUNDARY SUSPECT) → PIN (pin the contract before touching anything: PINNED/FLEXIBLE/SUSPECT behavior split, differential + property harness against the current code while it still exists, SUSPECT behaviors adjudicated so a bug is never enshrined as expected) → BASELINE (measured noise: interleaved repetitions with recorded spread, realistic plus worst-case shapes, contended when concurrent, allocation counts, machine profile) → DESIGN (at most 3 candidates, refuted on paper) → IMPLEMENT (in place, red-first, no dual paths) → VERIFY (risk-proportional adversarial ladder, hard cap 2 blind agents, "inline" in the request drops to zero agents with oracles unchanged: edge-hunter on hot or complexity-adding results, race oracle + interleaving-hunter on concurrent ones, the complexity class assigned by whichever hunter runs, boundary enforcement at trust boundaries, zero agents on cold simple paths) → MEASURE (adoption by proportionality; improve reverts on a miss, refactor adopts only if intent is preserved AND nothing regresses AND the code got simpler) → REPORT. Hot paths obey realtime rules — zero steady-state allocations, no hidden allocators, bounded worst case, no blocking. Testing method loads from the test-practice skill; language specifics from a <lang>-toolkit skill (e.g. msvcpp-toolkit) when one exists. Edits land in the existing files; no research fan-out, no parallel implementations. Trigger on "algo-craft", "/algo-craft", "add an algorithm", "implement an algorithm here", "improve this algorithm", "refactor this algorithm", "tune this function", "optimize this function in place", "in-place algorithmic work".
---

# algo-craft

Benchmark-gated, adversarially-proven algorithmic work on one named target, edited in place. Four jobs, one discipline:

- **Add** — implement a new algorithm where none exists.
- **Improve** — make an existing one measurably better.
- **Refactor** — preserve the intent, simplify the code; the internals are free to become a different algorithm.
- **Evaluate** — conclude it should be left alone; "no work needed" is a first-class outcome, not a failure.

Twin creeds: **performance does not exist until a benchmark says it does; correctness does not exist until it survives adversarial refutation** — an adversary that tried to kill it and failed. On the cold, sequential, simplicity-adding path that adversary is your own paper refutation; everywhere riskier, it is an oracle or a blind agent. Nothing is adopted on its author's word.

**Scope rule:** the target is the function / class / file the user names (or the code under discussion). If the work reveals a whole-subsystem redesign — many interacting components, a data flow that must change end to end — stop and report that the scope has outgrown this skill (algo-rewrite territory); never grow the run silently.

**Cost rule:** linear, no research fan-out, no parallel implementations, and a hard cap of **two blind verification agents** per run — dispatched only when the risk ladder below demands them. The ladder is the in-place sibling of algo-rewrite's `minimal` agent-economy level (paper refutation at DESIGN, hunters only where tags demand, mandates never merged under the 2-cap); **"inline"** (or "solo"/"agentless") in the request drops to **zero agents** — same oracles, same differential and boundary enforcement, main-thread self-audit instead of hunters, the complexity class self-assigned, and the report states plainly that no judgment was blind.

## The intent-oracle rule

Pin the **contract, not the algorithm**. The current implementation witnesses the contract's spirit — capabilities, invariants, guarantees, tolerances — never its own internals. Split every observable behavior at PIN:

- **PINNED** — the contract fixes the exact output → the differential harness compares exact.
- **FLEXIBLE** — the contract fixes a predicate only (±tolerance, any valid ordering, error model free to change) → compare by equivalence predicate or property.
- **SUSPECT** — the correctness scan doubts the current behavior → **adjudicate before pinning** (defect vs load-bearing quirk callers rely on). A defect gets a failing repro red-first and is fixed or recorded — it never enters the harness as expected behavior; bug-for-bug compatibility is refused.

Signatures, internals, data structures, error models, complexity: **FREE** — judged only by correctness (oracles and hunters), proof (the paper refutation), benchmark (MEASURE), and simplicity (adoption). Nothing else has a vote. Pinning an incidental behavior is a defect: it freezes the algorithm. Because pinned tests are contract-level, they outlive this change and stay valid across future algorithm swaps.

## Steps

1. **SCOPE.** State the job (add / improve / refactor / evaluate), the target and its contract — inputs, outputs, invariants callers rely on, guarantees (ordering, tolerances, thread-safety promises), and the **synchronization contract** if state is shared (per test-practice's Concurrent correctness). Run the **correctness lens**: a quick defect-risk scan with harden's four classes (boundary, concurrency, lifetime/resource, garbage data) → `SUSPECT` tags. Tag the target: `HOT/COLD`, `CONCURRENT` (shared mutable state now, or the work will introduce it), `TRUST-BOUNDARY`, `SUSPECT`. State CPU + memory complexity and allocation pattern of any existing code, realistic input shapes and sizes. If realistic N is small enough that nothing can matter, say so and stop. Invoke **test-practice**; if a `<lang>-toolkit` exists, borrow its **Benchmarking**, **Hot-path rules**, **Boundary enforcement**, and **Concurrency oracles** sections — if not, do not research one; proceed with the minimal harness and derive only what the tags require. At most one clarifying question, and only if the target or the realistic input shapes are genuinely ambiguous; "autonomous" in the request → zero questions.
2. **PIN.** Before any design: build green; tests touching the target pass — nothing can be pinned on red. Split behaviors PINNED / FLEXIBLE / SUSPECT per the intent-oracle rule; adjudicate every SUSPECT now (defect → failing repro red-first, then fix or record — harden-style, never enshrined). Write the **class map** (equivalence classes + BVA, test-practice TDD) for every job — it shapes the differential generator and is what any hunter later receives. Improve/refactor: build the **differential harness + property tests against the current implementation now, green** (test-practice Properties & differential) — it defines the intent, so capture the oracle while it exists. Add: write the class map's 1:1 tests **red** (test-practice TDD).
3. **BASELINE.** Micro-benchmark under **measured noise**: the project's harness if it has one, else a minimal timing loop — either way the harness must **defeat dead-code elimination**: every result is consumed/sunk (the toolkit's DoNotOptimize-equivalent when one exists, a volatile/checksum sink when hand-rolled), or the numbers are fiction; baseline runs interleaved with the repetition count the toolkit prescribes (or ≥5 hand-rolled), and the run-to-run **spread is recorded next to the numbers** — a win must later clear that spread, not an asserted one. Shapes: realistic small / typical / large **plus one worst-case shape** for the algorithm class at hand; `CONCURRENT` → measured contended per the toolkit's pattern. Capture median + p99 + max, allocation counts and peak memory where the language exposes them cheaply, and the machine profile — numbers are meaningless without it. Improve: this is the bar to beat. Refactor: the bar not to regress. Add: record the performance expectation instead — promised complexity at realistic N and any hot-path budget; the bench exists before the code it will judge. No candidate exists before this step is done.
4. **DESIGN.** At most 3 candidates, one line each: idea, expected win or behavior, complexity cost. A library already installed in the project is a valid candidate; adding a new dependency is out of scope here. **Refute each on paper** — complexity at realistic N, constant factors (cache, branches, allocations), the realtime rules where the path is hot, implementability of the sync contract where `CONCURRENT`, simplicity; drop anything that can't plausibly pay for its complexity. Adding: the simplest candidate that meets the requirement is the default winner. Evaluate may end here: every candidate refuted → a reasoned "leave it alone" verdict, reported as a success — the refutation record *is* the deliverable.
5. **IMPLEMENT.** Land the surviving candidate **in place**, in the existing files, red-first for any new behavior, to green. No dual paths, no flags preserving an old implementation.
6. **VERIFY — the risk ladder** (adversarial proof of correctness; cap 2 agents):
   - **Always:** full suite green, plus the differential + property harness from PIN (contract-level, so a refactor that swapped the algorithm still passes).
   - **`TRUST-BOUNDARY`:** boundary enforcement per test-practice/toolkit — standard mechanisms enabled; hand-written validation gets its red/green pair.
   - **`HOT`, or the result adds complexity (more-complex / hack):** one blind **edge-hunter** (test-practice Adversarial) on the contract + class map + code. Accepted claims become red tests fixed per the fix protocol.
   - **Whichever hunter runs assigns the complexity class** (the edge-hunter when dispatched, else the interleaving-hunter) — self-grading survives only on the zero-agent path, where by construction the result is simpler/same.
   - **`CONCURRENT`:** race oracle + bounded contended stress (test-practice Concurrent correctness), plus one blind **interleaving-hunter** on the sync contract + code. No race oracle on this machine → the claim ends `UNPROVEN`: improve/refactor default to **revert**; add reports plainly that concurrency is unproven — never adopted on suspicion.
   - **Cold + sequential + simpler:** zero agents — the paper refutation was the proportionate adversary; as light as it looks.
7. **MEASURE.** Rerun the benchmark interleaved against the recorded baseline and judge by the Adoption rules below. Improve: no adoption → revert; a reverted improvement is a successful evaluation. Refactor: intent preserved (differential green) **and** no regression outside the recorded spread **and** the code got simpler — any leg fails → revert. Add: the measured numbers must match the promised complexity and any hot-path budget; a miss → redesign, or report plainly what the code actually delivers.
8. **REPORT.** The numbers with their spread (before/after, or measured curve against the promise), what changed, candidates considered and why the rest died, SUSPECT adjudications, which ladder rungs ran and what the adversaries claimed vs what was actioned, and the exact command to rerun the bench.

## Realtime rules (hot paths only)

Hot = on the per-item / per-frame / per-request path, tagged at SCOPE. Cold paths are exempt and judged on simplicity + throughput only.

1. **Zero allocations steady-state.** Memory acquired at init — preallocation, pools, reused scratch. Warmup allocation is fine; per-iteration allocation (including GC pressure in managed languages) is a defect. Where the language exposes allocation counts, the hot-path benchmark asserts 0 — nonzero fails regardless of speed.
2. **No hidden allocators:** closures/boxing, string temporaries, container growth/rehash, exception machinery — the toolkit's Hot-path list when one exists.
3. **Bounded worst case beats better average:** pre-reserve to known bounds; no amortized-only guarantees where the spike lands on the hot path.
4. **No blocking:** no lock waits, syscalls, I/O, or logging on the hot path. Anything lock-free introduced here is `CONCURRENT` and owes the full ladder — a fast structure that races is not fast, it is wrong.
5. **Errors without unwinding or allocating** — value/status returns; exceptions/panics are setup/teardown territory.
6. **Layout matches access pattern:** contiguous over node-based.

Latency is judged median + p99 + max — winning median while regressing max is a loss.

## Adoption rules

The four judges are **correctness, proof, benchmark, simplicity** — nothing else has a vote. One principle: the burden of proof grows with the complexity the code adds. The complexity class — simpler/same, more complex, or hack — is assigned by whichever hunter runs (edge-hunter first, else interleaving-hunter), self-assigned only on the zero-agent path; record it with the verdict and a one-line rationale; the rationale is the gate.

- A result inside the recorded run-to-run spread is not a win — that's measurement, not judgment.
- Simpler or same complexity: any real win adopts; a tie adopts only if the code got simpler. A library already in the project counts as simpler — complexity we don't maintain doesn't count against us.
- More complex: the win must be worth maintaining the complexity — a modest win can justify modest complexity on a hot path; a cold path almost never earns added complexity.
- Hack (fragile cleverness, UB-adjacent tricks, layout gymnastics): only an exceptional, measured win on a genuinely hot path, contained behind a clean interface and commented as such.
- Hot paths: a max/p99 regression weighs heavily against adoption; new steady-state allocations are a loss regardless of median.
- An `UNPROVEN` correctness claim vetoes adoption regardless of the numbers — fast-but-unproven loses to slow-but-proven.
- Refactor adopts on its own three-legged rule (intent preserved, no regression, simpler) — a refactor that wins a benchmark but loses the differential is a defect, not a win.
- Doubt → the original wins (or, when adding, the simpler candidate does). A rationale you'd be embarrassed to read back is a rejection.

The same proportionality picks between candidates for an addition: extra complexity must buy a measured, needed win over the simpler option.
