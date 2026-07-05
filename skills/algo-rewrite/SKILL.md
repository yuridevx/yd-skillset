---
name: algo-rewrite
description: Whole-codebase algorithmic rewrite pipeline for C++ — inventory every algorithm and data flow with CPU/memory complexity, triage through SIMD / concurrency / library-first / hot-cold lenses, ideate greenfield replacements under a no-compatibility rule, adversarially refute ideas in independent agents, then settle survivors by A/B/C benchmark shootout (red-first TDD, zero-alloc hot paths, two-axis speed-vs-simplicity adoption), assemble winners into a sibling v2 folder with class-map tests + fuzzing, and finish with cpp-harden. Single user gate after triage; fully autonomous end-to-end when "autonomous" appears in the request. Trigger on "algo-rewrite", "/algo-rewrite", "algorithmic rewrite", "rewrite the algorithms", "optimize algorithms into v2", "make this algorithmically optimal", "rewrite for performance".
---

# algo-rewrite

Benchmark-gated algorithmic rewrite. Core principle: **an optimization does not exist until a benchmark says it does** — the twin of cpp-harden's "a defect does not exist until a failing test says it does." Every phase is a kill-filter; code lands in v2 only after surviving all of them.

The main thread runs **linearly** and owns all state. Independent agents (same model as the main thread) are used for exactly two jobs: adversarial proof of ideas (Phase 4) and correctness triage of prototypes (Phase 6). Agents receive only the artifact under judgment plus its checklist — never the generator's reasoning — so they cannot inherit its optimism.

## Autonomy contract

- **Exactly one user interaction: the Gate (Phase 2.5).** Every question the run will ever need is batched there.
- Beyond the gate, **never** ask "shall I proceed", "say go", "do you want me to continue", or present intermediate results as questions. The only stop conditions are a broken baseline at Phase 0 and completion at Phase 10.
- If the invocation contains "autonomous", "auto", "no questions", or equivalent intent: **skip the gate entirely**, decide everything solo, run Phase 0 → 10 end to end. Either way, every gate-class decision is recorded in the ledger (user-answered or self-decided) so the report shows what was chosen and why.

## Red-first rule (governs every phase that writes code)

**No code exists before its oracle does.** Every unit of work opens red — a failing test or a recorded baseline benchmark — and closes green:

- Harness self-check before the harness measures anything (Phase 0).
- Per prototype: tests → baseline number confirmed → implement to green → benchmark (Phase 6). Tests written after the code make the prototype invalid.
- Per v2 component: class map → 1:1 test set failing red against empty v2 → port to green → fuzz (Phase 7).
- Per fix (triage failure, fuzz finding, integration regression): failing repro test → minimal fix to green → repro kept as regression guard.
- Benchmark-first is the performance twin: no optimization is implemented before the number it must beat is in the ledger.

The report includes each component's red→green trail.

## No-copy rule (governs all work in every phase)

**Never copy existing code and edit it — rewrite into new files as the work needs them.** This applies to prototypes, benchmarks, tests, harnesses, and the v2 tree alike. A "port" is a rewrite from the component's contract and behavior (with the original open as reference), not a file copy: copied files smuggle in the old structure, dead code, and compatibility residue — exactly what the no-compat rule exists to kill.

The new file tree is designed, not accreted: one clear responsibility per file, headers/implementation split as the project's conventions dictate, directories that mirror the component structure from the ledger. Never dump everything into one file — a v2 that wins every benchmark but reads like a heap is a failed rewrite under the simplicity rule.

## Realtime rules (hot paths only)

Hot path = any code on the per-item / per-frame / per-request path, tagged in Phase 2. Cold paths are exempt and judged on simplicity + throughput only.

1. **Zero allocations steady-state.** Memory acquired at init: preallocation, arenas/pools, SBO, reused scratch. First-call warmup allocation is fine; per-iteration allocation is a defect.
2. **No hidden allocators:** `std::function` beyond SBO, `std::string` temporaries, `std::vector` growth, `unordered_map` rehash, `shared_ptr` control blocks, iostreams, `std::regex`, throw/catch.
3. **Bounded worst case beats better average.** No amortized-only guarantees where the spike lands on the hot path. If a container cannot pre-reserve to a known bound, the design is wrong.
4. **No blocking:** no mutex waits, syscalls, I/O, or logging (ring-buffer deferred logging if needed). Concurrency via lock-free structures, per-thread ownership, or SPSC/MPSC stage handoff.
5. **Errors by value** (`expected`/status codes); exceptions are setup/teardown territory.
6. **Layout matches access pattern:** contiguous over node-based, SoA where the SIMD lens demands, padding against false sharing.

Benchmark enforcement: a counting allocator is wired into every benchmark; **hot-path benchmarks assert 0 steady-state allocations** — nonzero fails regardless of speed. Latency reported as **median + p99 + max**; winning median while regressing max on a hot path is a loss.

## State: the dossier

```
algo-rewrite/            (scratchpad by default; in-repo if chosen at the gate)
├── ledger.md            append/update only; re-read at the start of every phase
├── bench/               harness, algo_bench.h, raw results (kept for reproducibility)
└── <target>-v2/         the rewrite — nearest sibling folder to the original target
```

Ledger state machines:

- Component: `INVENTORIED → TRIAGED → BASELINED → REWRITTEN | KEPT-AS-IS`
- Idea: `IDEA → REFUTED | PROTOTYPED → TRIAGE-FAILED(→ REFIXED | LOST) | LOST | WON → INTEGRATED | REGRESSED-REVERTED`
- Per component fields: `Path: hot|cold`, `Classes:` (equivalence map), `Fuzz:` (harness, runtime, findings), `Trail:` (red→green log)
- Header: `Assumptions:` (every self-made call from Phases 0–2), target, toolchain, machine profile

The ledger is memory across context compaction; never delete entries.

## Phase 0 — Setup gate (once, no questions)

1. Determine target (diff, given paths, or whole repo). Ambiguity → take the reasonable reading, record under `Assumptions:`. Verify the project **builds and existing tests pass** — broken baseline → stop and report; nothing can be baselined on red.
2. Detect toolchain: compiler, SIMD ISA (SSE/AVX2/AVX-512/NEON), thread facilities, package manager (vcpkg/conan/FetchContent).
3. Install the benchmark harness **library-first**: Google Benchmark → nanobench → hand-rolled timing only if neither installs.
4. Generate `bench/algo_bench.h`: `ALGO_BENCH_SCOPE(name)`, `ALGO_BENCH_COUNT(name, n)`, `ALGO_BENCH_ALLOC_GUARD(name)` gated on `ALGO_BENCH=1`; without the flag they expand to **nothing** — no branch, no atomic, no string. They may be placed in product code (v1 and v2) and left there permanently. **Self-check first (red-first):** under `ALGO_BENCH=1` the alloc guard catches a deliberate allocation; the same TU under release compiles to a no-op — assert both before the harness measures anything.
5. Write the ledger header: target, toolchain, machine profile (CPU, cores, cache sizes) — numbers are meaningless without it.

## Phase 1 — Scan & inventory

Read the whole target. Record every **algorithmic component** — code that loops over data, searches, sorts, hashes, parses, allocates in a pattern, or converts representations — and every **data flow**: entry shape, each transformation, each copy, each intermediate representation.

Per component: location, role, current algorithm, **CPU complexity, memory complexity, allocation behavior**, data structures, hot-path weight (called from where, how often). Inventory only — no judging, no ideas yet.

## Phase 2 — Triage, four lenses (one pass per lens)

1. **SIMD:** vectorizable (contiguous, branch-light, no cross-iteration deps) / vectorizable-after-restructure (AoS→SoA etc.) / not vectorizable.
2. **Concurrency:** current threading and its costs (lock contention, false sharing, oversubscription) + parallelizable-but-serial components.
3. **Library** (dedicated pass — web research explicitly encouraged): for every component, does a proven public library already do this (abseil/folly containers, simdjson, xxhash, TBB/taskflow, EVE/xsimd/highway, fmt, …)? A hit becomes the idea *adopt library X*; any hand-rolled competitor must beat it head-to-head in Phase 6 to exist. Ties go to the library.
4. **Hot/cold:** tag each component. Hot → realtime rules apply; cold → exempt.

## Phase 2.5 — The Gate (the only user interaction)

Present in one batch: assumptions taken so far, component inventory summary with hot/cold tags, library candidates found, and every genuine either-way call (dossier location, disputed hot/cold tags, scope questions). One question round. Answers go into the ledger; **Phases 3–10 then run end to end with zero further questions** — every later decision is resolved by the ledger, the adoption matrix, or a benchmark. In autonomous mode this phase is skipped and the same decisions are self-made and recorded.

## Phase 3 — Ideate (greenfield)

Per component, generate alternatives under **no-compatibility, no-fallback** rules: only the **spirit** of the public API must survive — same capabilities, free to change signatures, ownership, error model. No shims, no `#ifdef USE_OLD_PATH`. Killing a component entirely by changing the upstream representation is a first-class idea — the best optimization is deletion. Web research for state-of-the-art approaches is encouraged.

Simplicity gate at the door: every idea states its complexity cost. Hot-path ideas must obey the realtime rules; obvious violations are refuted on the spot. A 1.05× win that adds a dependency dies here.

## Phase 4 — Adversarial proof (independent agents)

One agent per surviving `IDEA`, given only: the idea, the component's code, and this kill-checklist —

- Complexity math holds at **realistic N**? (O(n log n)→O(n) is noise at n ≤ 64.)
- Constant factors: cache behavior, branch predictability, allocation count — not just big-O.
- SIMD: is the layout actually amenable, or does gather/scatter eat the win?
- Concurrency: enough work per task to amortize synchronization?
- Library: actually covers the use case (read its docs), maintained, installs on this toolchain?
- Realtime compliance for hot-path ideas.

Verdict rule — **doubt flows forward** (the benchmark is a cheap objective judge downstream); certainty-it-cannot-win or a simplicity violation → `REFUTED` with a one-line reason. The main thread records all dispositions.

## Phase 5 — Baseline benchmark

For every component with ≥1 surviving idea: benchmark **current v1** — realistic data shapes at small/typical/large sizes, measuring median + p99 + max latency, allocation count, peak memory, with the counting allocator wired in. Also capture in-situ macro baselines on v1 (these are Phase 8's bar). **No prototype exists before its baseline does.**

## Phase 6 — A/B/C shootout + correctness gate

Per component: minimal prototypes of each surviving idea (A = v1 baseline, B/C/… = candidates), same harness, same data. Prototype only the algorithmic core — no productionizing losers.

Order per candidate (red-first): main-flow + edge tests written first (failing — nothing implements them) → implement to green → benchmark.

**Correctness gate before any benchmark counts:** an independent triage agent per prototype checks breakage, edge cases, safe coding (boundaries, overflow, lifetime, UB), realtime compliance, and that tests predate code (ledger trail). Fail → fix and **re-benchmark** (the fix may eat the win), or `LOST`. The agent also assigns the candidate's **complexity class**: simpler/same, more complex, or hack (fragile cleverness, UB-adjacent tricks, layout gymnastics). Libraries replacing hand-rolled code count as **simpler** — complexity we don't maintain doesn't count against us.

**Adoption matrix:**

| | simpler / same | more complex | hack |
| --- | --- | --- | --- |
| real win (above run-to-run noise) | **ADOPT** — slightly faster beats simple | only if **≥2× on a measured hot path** | REJECT — simple-but-slower wins |
| tie / loss | only if simpler | REJECT | REJECT |

Hot-path hard gates apply on top: 0 steady-state allocations, and max-latency regression = loss. All results land in the ledger with actual numbers.

## Phase 7 — Build v2: tests first, then port, then fuzz

Assemble `<target>-v2/` from `WON` entries only; everything else is the **simplest faithful port** — written fresh per the no-copy rule, into a deliberately designed file tree. Per component, in this order:

1. **Class map:** enumerate the representable input classes from the public API contract — empty, one element, typical, max-size, boundary values, malformed-only-if-reachable. Inputs the contract makes impossible are **out of scope**: no breaking things under unrealistic expectations.
2. **Minimal test set:** exactly one test per class plus main flows — the optimal set is the smallest where every class appears once; redundant tests are noise and get refused like redundant candidates. Written **red against the empty v2**, recorded in the ledger as `Classes: N → Tests: N (1:1)`.
3. **Port/integrate to green.**
4. **Fuzz:** structure-aware harness at each trust boundary, generating contract-valid inputs (bytes into parsers; arrays into sorters — never garbage memory into internal APIs). libFuzzer (clang / clang-cl on MSVC), time-boxed: 60s/component default, longer for parsers/decoders. Findings become ledger entries and **block adoption of that part** until fixed red-first.

## Phase 8 — Integrated benchmark & adopt

Re-measure every integrated part **in situ** via the gated macros inside real v2 flows, plus end-to-end runs vs the v1 macro baselines from Phase 5. Microbenches judged the shootout; in-situ numbers judge integration. A part that won isolated but regresses integrated → `REGRESSED-REVERTED`, replaced by the simple port. Exit criteria: v2 tests green + fuzz clean + every adopted win confirmed in situ.

## Phase 9 — Harden

Invoke the **cpp-harden** skill on `<target>-v2/`. It runs its own full convergence loop with its own ledger; the fuzz harnesses from Phase 7 are handed over as extra oracles.

## Phase 10 — Report (once)

From the ledger, cumulative:

- **Adopted** — per part: before/after numbers (median/p99/max, allocs), complexity class, red→green trail.
- **Library adoptions** — what replaced hand-rolled code and why it won or tied.
- **Refuted / lost / reverted** — one line each; this is the noise receipt ("31 ideas, 6 adopted, here's why the rest died").
- **Coverage** — per component `Classes: N → Tests: N`, fuzz runtime + findings.
- **Machine profile** and exact commands to re-run `bench/`.

Never dress up an interrupted run as complete; report the actual last ledger state — the ledger lets a later session resume exactly where it stopped.
