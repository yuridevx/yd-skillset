---
name: algo-rewrite
description: Whole-codebase algorithmic rewrite pipeline for C++ — inventory every algorithm, data flow, and feature with CPU/memory complexity, triage through SIMD / concurrency / library-first / hot-cold lenses, ideate greenfield replacements under a no-compatibility rule, adversarially refute ideas in independent agents, settle survivors by A/B/C benchmark shootout (red-first TDD, zero-alloc hot paths, two-axis speed-vs-simplicity adoption), assemble winners into a fresh-file sibling v2 with class-map tests + fuzzing, audit feature parity with a gap-hunter agent that can force full re-iteration, and finish with cpp-harden. Runs as a chain of named gates, each with a hard exit condition — one user question batch at the ASK gate, fully autonomous end-to-end when "autonomous" appears in the request. Trigger on "algo-rewrite", "/algo-rewrite", "algorithmic rewrite", "rewrite the algorithms", "optimize algorithms into v2", "make this algorithmically optimal", "rewrite for performance".
---

# algo-rewrite

Benchmark-gated algorithmic rewrite. Core principle: **an optimization does not exist until a benchmark says it does** — the twin of cpp-harden's "a defect does not exist until a failing test says it does."

The skill is a chain of **gates**. A gate is not a phase to schedule work into — it is a checkpoint with a hard exit condition; you are always standing at exactly one gate, doing its work, and you may not step past it until its exit condition is objectively green. There is no "later": work either happens at its gate or is killed at its gate.

The main thread runs linearly and owns all state. Independent agents (same model as the main thread) are used for exactly three jobs: killing ideas at REFUTE, correctness-triaging prototypes at SHOOTOUT, and hunting missing features at PARITY. Agents receive only the artifact under judgment plus its checklist — never the generating thread's reasoning — so they cannot inherit its optimism.

## Flow map

```
SETUP → SCAN → TRIAGE → ASK ══ only user stop (skipped if "autonomous")
                          │
      ┌───────────────────▼────────────────────────────────────┐
      │ REWRITE LOOP                                           │
      │ IDEATE → REFUTE → BASELINE → SHOOTOUT → ASSEMBLE       │
      │    ▲                                        │          │
      │    │                                     PROVE         │
      │    │                                        │          │
      │    └── CONFIRMED-GAP features ◄────────  PARITY        │
      │        (full loop, never a band-aid)        │          │
      └─────────────────────────────────────────────┼──────────┘
                        zero design-defining claims ▼
                                       HARDEN → REPORT
```

## Autonomy contract

- **Exactly one user interaction: the ASK gate.** Every question the run will ever need is batched there.
- Past ASK, **never** ask "shall I proceed", "say go", "want me to continue", or present intermediate results as questions. The only stops are a broken baseline at SETUP and completion at REPORT.
- If the invocation contains "autonomous", "auto", "no questions", or equivalent intent: **skip ASK entirely**, self-decide, run end to end. Either way every ASK-class decision lands in the ledger (user-answered or self-decided) so the report shows what was chosen and why.

## Universal rules (govern every gate)

### Red-first

**No code exists before its oracle does.** Every unit of work opens red — a failing test or a recorded baseline benchmark — and closes green:

- Harness self-check before the harness measures anything (SETUP).
- Per prototype: tests → baseline confirmed → implement to green → benchmark (SHOOTOUT). Tests written after code invalidate the prototype.
- Per v2 component: class map → 1:1 tests failing red against empty v2 → port to green → fuzz (ASSEMBLE).
- Per fix (triage failure, fuzz finding, regression, parity gap): failing repro/intent test → minimal fix to green → test kept as regression guard.
- Benchmark-first is the performance twin: no optimization is implemented before the number it must beat is in the ledger.

The report includes each component's red→green trail.

### No-copy

**Never copy existing code and edit it — rewrite into new files as the work needs them.** Applies to prototypes, benchmarks, tests, harnesses, and the v2 tree alike. A "port" is a rewrite from the component's contract and behavior (original open as reference), never a file copy: copies smuggle in old structure, dead code, and compatibility residue — exactly what the no-compat rule exists to kill.

The new file tree is designed, not accreted: one clear responsibility per file, header/implementation split per the project's conventions, directories mirroring the ledger's component structure. Never dump everything into one file — a v2 that wins every benchmark but reads like a heap is a failed rewrite under the simplicity rule.

### Realtime (hot paths only)

Hot path = any code on the per-item / per-frame / per-request path, tagged at TRIAGE. Cold paths are exempt and judged on simplicity + throughput only.

1. **Zero allocations steady-state.** Memory acquired at init: preallocation, arenas/pools, SBO, reused scratch. Warmup allocation fine; per-iteration allocation is a defect.
2. **No hidden allocators:** `std::function` beyond SBO, `std::string` temporaries, `std::vector` growth, `unordered_map` rehash, `shared_ptr` control blocks, iostreams, `std::regex`, throw/catch.
3. **Bounded worst case beats better average.** No amortized-only guarantees where the spike lands on the hot path; a container that cannot pre-reserve to a known bound is the wrong design.
4. **No blocking:** no mutex waits, syscalls, I/O, logging (ring-buffer deferred logging if needed). Concurrency via lock-free structures, per-thread ownership, or SPSC/MPSC stage handoff.
5. **Errors by value** (`expected`/status codes); exceptions are setup/teardown territory.
6. **Layout matches access pattern:** contiguous over node-based, SoA where the SIMD lens demands, padding against false sharing.

Enforcement: a counting allocator is wired into every benchmark; **hot-path benchmarks assert 0 steady-state allocations** — nonzero fails regardless of speed. Latency is **median + p99 + max**; winning median while regressing max on a hot path is a loss.

## State: the dossier

```
algo-rewrite/            (scratchpad by default; in-repo if chosen at ASK)
├── ledger.md            append/update only; re-read at every gate
├── bench/               harness, algo_bench.h, raw results (kept for reproducibility)
└── <target>-v2/         the rewrite — nearest sibling folder to the original target
```

Ledger state machines:

- Component: `INVENTORIED → TRIAGED → BASELINED → REWRITTEN | KEPT-AS-IS`
- Idea: `IDEA → REFUTED | PROTOTYPED → TRIAGE-FAILED(→ REFIXED | LOST) | LOST | WON → INTEGRATED | REGRESSED-REVERTED`
- Feature: `FEATURE → COVERED | SMALL-DIFF | DROPPED-BY-DESIGN | CONFIRMED-GAP → RE-ENTERED(iter N) → COVERED`
- Per component: `Path: hot|cold`, `Classes:` (equivalence map), `Fuzz:` (harness, runtime, findings), `Trail:` (red→green log)
- Header: `Assumptions:` (every self-made call), target, toolchain, machine profile, iteration counter

The ledger is memory across context compaction; never delete entries.

## The gates

### SETUP

1. Determine target (diff, given paths, or whole repo). Ambiguity → take the reasonable reading, record under `Assumptions:`.
2. Verify the project **builds and existing tests pass** — broken baseline → stop and report; nothing can be baselined on red.
3. Detect toolchain: compiler, SIMD ISA (SSE/AVX2/AVX-512/NEON), threads, package manager (vcpkg/conan/FetchContent).
4. Install the benchmark harness **library-first**: Google Benchmark → nanobench → hand-rolled timing only if neither installs.
5. Generate `bench/algo_bench.h`: `ALGO_BENCH_SCOPE(name)`, `ALGO_BENCH_COUNT(name, n)`, `ALGO_BENCH_ALLOC_GUARD(name)` gated on `ALGO_BENCH=1`; without the flag they expand to **nothing** — no branch, no atomic, no string. They may live in product code (v1 and v2) permanently. Red-first self-check: under `ALGO_BENCH=1` the alloc guard catches a deliberate allocation; the same TU in release compiles to a no-op — assert both.

**Exit:** build green, tests pass, harness self-check red→green, ledger header written (target, toolchain, CPU/cores/caches — numbers are meaningless without the machine profile).

### SCAN

Read the whole target. Record every **algorithmic component** (code that loops over data, searches, sorts, hashes, parses, allocates in a pattern, converts representations), every **data flow** (entry shape, each transformation, each copy, each intermediate), and the **feature map** — what the target does *for its users* at intent level: capability, why a user needs it, observable behavior, feature branches (modes, options, config-driven behaviors, documented edge semantics), where v1 implements it. **Guarantees are features too:** ordering stability, thread-safety promises, documented tolerances, complexity bounds callers rely on — the easiest things to lose silently.

Per component: location, role, algorithm, **CPU + memory complexity, allocation behavior**, data structures, hot-path weight. Inventory only — no judging, no ideas.

**Exit:** every component, flow, and feature in the ledger with complexity recorded.

### TRIAGE

Four lenses, one pass each:

1. **SIMD:** vectorizable (contiguous, branch-light, no cross-iteration deps) / vectorizable-after-restructure (AoS→SoA etc.) / not.
2. **Concurrency:** current threading costs (contention, false sharing, oversubscription) + parallelizable-but-serial components.
3. **Library** (dedicated pass — web research explicitly encouraged): does a proven public library already do this (abseil/folly, simdjson, xxhash, TBB/taskflow, EVE/xsimd/highway, fmt, …)? A hit becomes the idea *adopt library X*; hand-rolled competitors must beat it head-to-head at SHOOTOUT. Ties go to the library.
4. **Hot/cold:** tag every component; hot → realtime rules apply.

**Exit:** every component tagged on all four lenses.

### ASK — the only user stop

Present in one batch: assumptions taken, inventory summary with hot/cold tags, library candidates, planned feature drops, and every genuine either-way call (dossier location, disputed tags, scope). One question round; answers go into the ledger. In autonomous mode: skip, self-decide, record.

**Exit:** every ASK-class decision in the ledger. From here to REPORT, zero questions — every later decision is resolved by the ledger, the adoption matrix, or a benchmark.

### IDEATE (rewrite loop entry — re-entered by PARITY)

Per component, generate alternatives under **no-compatibility, no-fallback** rules: only the **spirit** of the public API must survive — same capabilities, free to change signatures, ownership, error model. No shims, no `#ifdef USE_OLD_PATH`. Killing a component by changing the upstream representation is a first-class idea — the best optimization is deletion. Web research for state-of-the-art approaches encouraged.

An idea that deletes or narrows a capability must declare `DROPS: <feature> — <rationale>` (surfaced at ASK, or self-decided and recorded). This is what lets PARITY distinguish deliberate removal from accidental loss.

Simplicity at the door: every idea states its complexity cost. Hot-path ideas must obey the realtime rules; obvious violations die here. A 1.05× win that adds a dependency dies here.

**Exit:** every live component has its ideas recorded, each with complexity cost and any `DROPS:` declared.

### REFUTE (independent agents)

One agent per idea, given only the idea, the component's code, and the kill-checklist:

- Complexity math holds at **realistic N**? (O(n log n)→O(n) is noise at n ≤ 64.)
- Constant factors: cache behavior, branch predictability, allocation count — not just big-O.
- SIMD: layout actually amenable, or does gather/scatter eat the win?
- Concurrency: enough work per task to amortize synchronization?
- Library: actually covers the use case (read the docs), maintained, installs on this toolchain?
- Realtime compliance for hot-path ideas.

Verdict rule — **doubt flows forward** (the benchmark downstream is a cheap objective judge); certainty-it-cannot-win or a simplicity violation → `REFUTED` with a one-line reason.

**Exit:** every idea dispositioned; main thread has recorded all verdicts.

### BASELINE

For every component with a surviving idea: benchmark **current v1** — realistic shapes at small/typical/large sizes; median + p99 + max latency, allocation count, peak memory, counting allocator wired in. Also capture in-situ macro baselines on v1 — these are PROVE's bar.

**Exit:** the number every candidate must beat is in the ledger. No prototype exists before its baseline does.

### SHOOTOUT

Per component: minimal prototypes of each surviving idea (A = v1, B/C/… = candidates), same harness, same data. Prototype only the algorithmic core — no productionizing losers. Order per candidate (red-first): main-flow + edge tests written failing → implement to green → benchmark.

**Correctness gate before any benchmark counts:** an independent triage agent per prototype checks breakage, edge cases, safe coding (boundaries, overflow, lifetime, UB), realtime compliance, and that tests predate code (ledger trail). Fail → fix and **re-benchmark** (the fix may eat the win), or `LOST`. The agent also assigns the **complexity class**: simpler/same, more complex, or hack (fragile cleverness, UB-adjacent tricks, layout gymnastics). Libraries replacing hand-rolled code count as **simpler** — complexity we don't maintain doesn't count against us.

**Adoption matrix:**

| | simpler / same | more complex | hack |
| --- | --- | --- | --- |
| real win (above run-to-run noise) | **ADOPT** — slightly faster beats simple | only if **≥2× on a measured hot path** | REJECT — simple-but-slower wins |
| tie / loss | only if simpler | REJECT | REJECT |

Hot-path hard gates on top: 0 steady-state allocations; max-latency regression = loss.

**Exit:** every candidate `WON` or `LOST` via the matrix, with actual numbers in the ledger.

### ASSEMBLE

Build `<target>-v2/` from `WON` entries only; everything else is the **simplest faithful port** — fresh files per the no-copy rule, into a deliberately designed tree. Per component, in order:

1. **Class map:** enumerate representable input classes from the public API contract — empty, one element, typical, max-size, boundaries, malformed-only-if-reachable. Contract-impossible inputs are **out of scope**: no breaking things under unrealistic expectations.
2. **Minimal test set:** exactly one test per class plus main flows — the smallest set where every class appears once; redundant tests are refused like redundant candidates. Written **red against the empty v2**; ledger records `Classes: N → Tests: N (1:1)`.
3. **Port/integrate to green.**
4. **Fuzz:** structure-aware harness per trust boundary, contract-valid inputs (bytes into parsers, arrays into sorters — never garbage into internal APIs). libFuzzer (clang / clang-cl on MSVC), time-boxed: 60s/component default, longer for parsers/decoders. Findings block that part until fixed red-first.

**Exit:** v2 complete, class-map tests 1:1 green, fuzz clean.

### PROVE

Re-measure every integrated part **in situ** via the gated macros inside real v2 flows, plus end-to-end vs the v1 macro baselines. Microbenches judged the shootout; in-situ numbers judge integration. Won-isolated-but-regressed-integrated → `REGRESSED-REVERTED`, replaced by the simple port.

**Exit:** every adopted win confirmed in place; regressions reverted; tests green + fuzz clean.

### PARITY (1 gap-hunter agent per pass — can force a new loop iteration)

One agent receives v1, v2, the feature map, and the `DROPS:` list — not the rewrite thread's reasoning. Mandate: find features present in v1 whose **intent** is absent in v2, and **triage severity itself**:

- **Design-defining** — a capability, mode, or guarantee users build around; losing it forces callers to hand-roll logic or restructure usage. Only these come back as `PARITY-CLAIM`s.
- **Small** — convenience overloads, incidental output details, precision nuances, one-line-recoverable behaviors. Returned as `SMALL-DIFF` notes: recorded, never actioned. Unsure → must argue "usage would restructure without it"; can't → small.

The **spirit rule** for judging: changed signatures/error models, different internals, float drift within reasonable tolerance, renamed/merged concepts = fine. Missing capability, silently narrowed input domain, dropped mode, lost guarantee = gap. "Line X has no equivalent" and "differs at the 5th decimal" are not claims.

Main thread adjudicates each claim: **refute with a trace** of how v2 serves the intent / **DROPPED-BY-DESIGN** (matches a declared drop) / **CONFIRMED-GAP**. An undeclared drop is always a confirmed gap — even if dropping was right, the decision must become explicit, never accidental.

**Confirmed gaps get the full pipeline, not band-aids:** each is registered in the ledger and the rewrite loop **re-enters at IDEATE** for that feature set — ideate how the capability fits v2's design natively, refute, baseline against v1's implementation, shoot out alternatives, assemble red-first with class map + fuzz, prove in situ. The result is indistinguishable from first-iteration work: same rules, same gates, same evidence.

**Exit:** a full gap-hunter pass (fresh agent) returns **zero new design-defining claims**. Cost per pass: exactly 1 agent call.

### HARDEN

Invoke the **cpp-harden** skill on `<target>-v2/`. It runs its own full convergence loop with its own ledger; the fuzz harnesses from ASSEMBLE are handed over as extra oracles.

**Exit:** cpp-harden converged.

### REPORT

From the ledger, cumulative:

- **Adopted** — per part: before/after numbers (median/p99/max, allocs), complexity class, red→green trail.
- **Library adoptions** — what replaced hand-rolled code and why it won or tied.
- **Refuted / lost / reverted** — one line each: the noise receipt ("31 ideas, 6 adopted, here's why the rest died").
- **Parity matrix** — `N features → covered / dropped-by-design (rationale) / migrated-after-audit`, plus refuted claims — the receipt that the audit had teeth.
- **Coverage** — per component `Classes: N → Tests: N`, fuzz runtime + findings.
- **Machine profile** and exact commands to re-run `bench/`.

Never dress up an interrupted run as complete; report the actual last ledger state — the ledger lets a later session resume exactly where it stopped.
