---
name: algo-rewrite
description: Whole-codebase algorithmic rewrite pipeline for any language — inventory every algorithm, data flow, and feature with CPU/memory complexity, triage through SIMD / concurrency / library-first / hot-cold lenses, ideate greenfield replacements under a no-compatibility rule, adversarially refute ideas in independent agents, settle survivors by A/B/C benchmark shootout (red-first TDD, zero-alloc hot paths, proportional speed-vs-simplicity adoption judgment), assemble winners into a fresh-file sibling v2 with class-map tests, a blind adversarial edge audit, and deterministic boundary enforcement at every trust boundary, audit feature parity with a gap-hunter agent that can force full re-iteration. Testing method (red-first TDD, class map + BVA, boundary enforcement standard, edge-hunter protocol) loads from the test-practice skill; language specifics (benchmark harness, allocation tracking, hidden allocators, SIMD/concurrency facilities, library shortlist, boundary-enforcement mechanisms) load from a <lang>-toolkit skill (e.g. msvcpp-toolkit) or are derived at setup when none exists. Runs as a chain of named gates, each with a hard exit condition — one user question batch at the ASK gate, fully autonomous end-to-end when "autonomous" appears in the request. Trigger on "algo-rewrite", "/algo-rewrite", "algorithmic rewrite", "rewrite the algorithms", "optimize algorithms into v2", "make this algorithmically optimal", "rewrite for performance".
---

# algo-rewrite

Benchmark-gated algorithmic rewrite. Core principle: **an optimization does not exist until a benchmark says it does** — the twin of harden's "a defect does not exist until a failing test says it does."

The skill is a chain of **gates**. A gate is not a phase to schedule work into — it is a checkpoint with a hard exit condition; you are always standing at exactly one gate, doing its work, and you may not step past it until its exit condition is objectively green. There is no "later": work either happens at its gate or is killed at its gate.

The main thread runs linearly and owns all state. Independent agents (same model as the main thread) are used for exactly four jobs: killing ideas at REFUTE, correctness-triaging prototypes at SHOOTOUT, edge-hunting at ASSEMBLE, and hunting missing features at PARITY. Agents receive only the artifact under judgment plus its checklist — never the generating thread's reasoning — so they cannot inherit its optimism.

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
                                            REPORT
```

## Autonomy contract

- **Exactly one user interaction: the ASK gate.** Every question the run will ever need is batched there.
- Past ASK, **never** ask "shall I proceed", "say go", "want me to continue", or present intermediate results as questions. The only stops are a broken baseline at SETUP and completion at REPORT.
- If the invocation contains "autonomous", "auto", "no questions", or equivalent intent: **skip ASK entirely**, self-decide, run end to end. Either way every ASK-class decision is recorded in the dialog (user-answered or self-decided) so the report shows what was chosen and why.

## The language toolkit

Every language-specific choice in this skill is delegated to a **toolkit**: at SETUP, detect the target language and invoke the matching `<lang>-toolkit` skill (e.g. `msvcpp-toolkit`). It supplies, in named sections: **Toolchain** (what to detect), **Benchmarking** (harness install order, instrumentation pattern, allocation tracking), **Hot-path rules** (hidden allocation sources, error model), **SIMD / data-parallel** and **Concurrency** facilities, the **Library shortlist** for the library-first lens, and **Boundary enforcement** (mechanisms, defaults).

No toolkit skill for this language → derive the same sections yourself at SETUP (web research encouraged) and state them in the dialog; the pipeline is identical either way.

## The testing rulebook

All test work follows the **test-practice** skill, invoked at SETUP. Its three named sections govern: **TDD** (red-first protocol, class map + boundary-value analysis, main-flow-first 1:1 test set, fix protocol), **Boundary enforcement** (standard mechanisms enabled per the toolkit; hand-written trust-boundary validation gets its red/green pair), **Adversarial** (blind edge-hunter agent protocol). This skill's gates say *when* those sections run and what gets recorded; test-practice says *how*; the toolkit says *with what*.

## Universal rules (govern every gate)

### Red-first

**No code exists before its oracle does** — full protocol in test-practice's TDD section. Every unit of work opens red — a failing test or a recorded baseline benchmark — and closes green:

- Harness self-check before the harness measures anything (SETUP).
- Per prototype: main-flow + edge tests must pass before its benchmark counts (SHOOTOUT). Prototypes are throwaways — the independent correctness gate, not test ordering, is what validates them.
- Per v2 component: class map → 1:1 tests failing red against empty v2 → port to green → edge audit → boundary enforcement (ASSEMBLE).
- Per fix (triage failure, boundary-enforcement finding, edge-audit gap, regression, parity gap): failing repro/intent test → minimal fix to green → test kept as regression guard.
- Benchmark-first is the performance twin: no optimization is implemented before the number it must beat is recorded.

### No-copy

**Never copy existing code and edit it — rewrite into new files as the work needs them.** Applies to prototypes, benchmarks, tests, harnesses, and the v2 tree alike. A "port" is a rewrite from the component's contract and behavior (original open as reference), never a file copy: copies smuggle in old structure, dead code, and compatibility residue — exactly what the no-compat rule exists to kill.

The new file tree is designed, not accreted: one clear responsibility per file, module layout per the project's conventions, directories mirroring the component inventory from SCAN. Never dump everything into one file — a v2 that wins every benchmark but reads like a heap is a failed rewrite under the simplicity rule.

### Realtime (hot paths only)

Hot path = any code on the per-item / per-frame / per-request path, tagged at TRIAGE. Cold paths are exempt and judged on simplicity + throughput only.

1. **Zero allocations steady-state.** Memory acquired at init: preallocation, arenas/pools, buffer reuse, reused scratch. Warmup allocation fine; per-iteration allocation (including GC pressure in managed languages) is a defect.
2. **No hidden allocators.** The toolkit's Hot-path rules list the language's silent allocation sources (closures/boxing, string temporaries, container growth/rehash, exception machinery, …); hot paths avoid all of them.
3. **Bounded worst case beats better average.** No amortized-only guarantees where the spike lands on the hot path; a container that cannot pre-reserve to a known bound is the wrong design. In managed languages a GC pause triggered by hot-path garbage is a worst-case spike you caused.
4. **No blocking:** no lock waits, syscalls, I/O, logging (ring-buffer deferred logging if needed). Concurrency via lock-free structures, per-thread ownership, or SPSC/MPSC stage handoff.
5. **Errors without unwinding or allocating** on the hot path — value/status returns per the toolkit's error model; exceptions/panics are setup/teardown territory.
6. **Layout matches access pattern:** contiguous over node-based, SoA where the SIMD lens demands, padding against false sharing where the language exposes layout.

Enforcement: allocation tracking (the toolkit's mechanism — counting allocator, GC allocation counters, allocation profiler hooks) is wired into every benchmark; **hot-path benchmarks assert 0 steady-state allocations** — nonzero fails regardless of speed. Latency is **median + p99 + max**; winning median while regressing max on a hot path is a loss.

## State

Two real artifacts on disk; everything else lives in the conversation:

```
algo-rewrite/            (scratchpad by default; in-repo if chosen at ASK)
├── bench/               harness, instrumentation, raw results (kept for reproducibility)
└── <target>-v2/         the rewrite — nearest sibling folder to the original target
```

Working state is an in-dialog status table — one line per component, idea, and feature: id, hot/cold tag, current status, and the numbers or one-line reason that justify it. Status vocabulary: components end `REWRITTEN | KEPT-AS-IS`; ideas end `REFUTED | LOST | WON | REGRESSED-REVERTED`; features end `COVERED | SMALL-DIFF | DROPPED-BY-DESIGN` (via `CONFIRMED-GAP` when PARITY forces re-entry). Header: `Assumptions:` (every self-made call), target, language + toolkit (or derived sections), toolchain, machine profile, iteration counter.

Restate the current table compactly at every gate exit — never drop a line, whatever its status; the restatement is what carries the record through context compaction. Anything an agent needs (idea, code, checklist, class map, feature map, `DROPS:` list) is passed explicitly in its prompt; its verdict comes back in its reply and lands in the table.

## The gates

### SETUP

1. Determine target (diff, given paths, or whole repo). Ambiguity → take the reasonable reading, record under `Assumptions:`.
2. Verify the project **builds and existing tests pass** — broken baseline → stop and report; nothing can be baselined on red.
3. **Resolve the language toolkit** (see above) and **invoke the test-practice skill**; record both — or the derived toolkit sections — in the dialog.
4. Detect the toolchain per the toolkit's Toolchain section (compiler/runtime, SIMD ISA where relevant, threads, package manager).
5. Install the benchmark harness **library-first** in the toolkit's Benchmarking order; hand-rolled timing only if nothing installs.
6. Set up bench instrumentation per the toolkit: scope/count/alloc-guard hooks gated on a build flag; without the flag they compile to **nothing** — no branch, no atomic, no string — so they may live in product code (v1 and v2) permanently. Red-first self-check: with the flag on, the alloc guard catches a deliberate allocation; with it off, the same code builds to a no-op — assert both.

**Exit:** build green, tests pass, harness self-check red→green, setup summary stated (target, language/toolkit, toolchain, CPU/cores/caches — numbers are meaningless without the machine profile).

### SCAN

Read the whole target. Record every **algorithmic component** (code that loops over data, searches, sorts, hashes, parses, allocates in a pattern, converts representations), every **data flow** (entry shape, each transformation, each copy, each intermediate), and the **feature map** — what the target does *for its users* at intent level: capability, why a user needs it, observable behavior, feature branches (modes, options, config-driven behaviors, documented edge semantics), where v1 implements it. **Guarantees are features too:** ordering stability, thread-safety promises, documented tolerances, complexity bounds callers rely on — the easiest things to lose silently.

Every component gets one status-table line (location, role, algorithm). Full detail — **CPU + memory complexity, allocation behavior, data structures** — only for components plausibly hot or plausibly rewritable; a cold config-parser doesn't need its complexity derived to be tagged `KEPT-AS-IS`. Inventory only — no judging, no ideas.

**Exit:** every component, flow, and feature in the status table; rewrite candidates with complexity recorded.

### TRIAGE

Four lenses, one pass each:

1. **SIMD / data-parallel:** per the toolkit's facilities — vectorizable (contiguous, branch-light, no cross-iteration deps) / vectorizable-after-restructure (AoS→SoA etc.) / not. If the toolkit says the language has no practical SIMD story, this lens narrows to vectorization-friendly layout and bulk operations.
2. **Concurrency:** current threading costs (contention, false sharing, oversubscription) + parallelizable-but-serial components, judged against the toolkit's concurrency facilities.
3. **Library** (dedicated pass — web research explicitly encouraged): does a proven public library already do this? Start from the toolkit's Library shortlist, extend by research. A hit becomes the idea *adopt library X*; hand-rolled competitors must beat it head-to-head at SHOOTOUT. Ties go to the library.
4. **Hot/cold:** tag every component; hot → realtime rules apply.

**Exit:** every component tagged on all four lenses.

### ASK — the only user stop

Present in one batch: assumptions taken, inventory summary with hot/cold tags, library candidates, planned feature drops, and every genuine either-way call (artifact location, disputed tags, scope). One question round; answers are recorded. In autonomous mode: skip, self-decide, record.

**Exit:** every ASK-class decision recorded. From here to REPORT, zero questions — every later decision is resolved by the recorded decisions, the adoption judgment, or a benchmark.

### IDEATE (rewrite loop entry — re-entered by PARITY)

Per component, generate alternatives under **no-compatibility, no-fallback** rules: only the **spirit** of the public API must survive — same capabilities, free to change signatures, ownership, error model. No shims, no `#ifdef USE_OLD_PATH`-style dual paths. Killing a component by changing the upstream representation is a first-class idea — the best optimization is deletion. Web research for state-of-the-art approaches encouraged.

An idea that deletes or narrows a capability must declare `DROPS: <feature> — <rationale>` (surfaced at ASK, or self-decided and recorded). This is what lets PARITY distinguish deliberate removal from accidental loss.

Simplicity at the door: every idea states its complexity cost. Hot-path ideas must obey the realtime rules; obvious violations die here. So does an idea whose plausible win is clearly too small to carry its cost (a marginal speedup that adds a whole dependency) — the same proportionality the adoption judgment applies at SHOOTOUT.

**Exit:** every live component has its ideas recorded, each with complexity cost and any `DROPS:` declared.

### REFUTE (independent agents)

One agent per idea, given only the idea, the component's code, and the kill-checklist:

- Complexity math holds at **realistic N**? (O(n log n)→O(n) is noise at n ≤ 64.)
- Constant factors: cache behavior, branch predictability, allocation count — not just big-O.
- SIMD/data-parallel: layout actually amenable, or does gather/scatter eat the win?
- Concurrency: enough work per task to amortize synchronization?
- Library: actually covers the use case (read the docs), maintained, installs on this toolchain?
- Realtime compliance for hot-path ideas.

Verdict rule — **doubt flows forward** (the benchmark downstream is a cheap objective judge); certainty-it-cannot-win or a simplicity violation → `REFUTED` with a one-line reason.

**Exit:** every idea dispositioned; main thread has recorded all verdicts.

### BASELINE

For every component with a surviving idea: benchmark **current v1** — realistic shapes at small/typical/large sizes; median + p99 + max latency, allocation count, peak memory, allocation tracking wired in. Also capture in-situ macro baselines on v1 — these are PROVE's bar.

**Exit:** the number every candidate must beat is recorded. No prototype exists before its baseline does.

### SHOOTOUT

Per component: minimal prototypes of each surviving idea (A = v1, B/C/… = candidates), same harness, same data. Prototype only the algorithmic core — no productionizing losers. Order per candidate (red-first): main-flow + edge tests written failing → implement to green → benchmark.

**Correctness gate before any benchmark counts:** an independent triage agent per prototype checks breakage, edge cases, safe coding (boundaries, overflow, lifetime, undefined behavior where the language has it), and realtime compliance. Fail → fix and **re-benchmark** (the fix may eat the win), or `LOST`. The agent also assigns the **complexity class**: simpler/same, more complex, or hack (fragile cleverness, UB-adjacent tricks, layout gymnastics). Libraries replacing hand-rolled code count as **simpler** — complexity we don't maintain doesn't count against us.

**Adoption judgment — proportionality, not gates.** One principle: the burden of proof grows with the complexity a candidate adds. Weigh the measured win (its size, and how hot the path is) against the complexity class the triage agent assigned, and decide per candidate; the verdict plus a one-line rationale goes in the status table — the rationale is the gate. Anchors:

- A result below run-to-run noise is not a win — that's measurement, not judgment.
- Simpler or same complexity: any real win adopts; a tie adopts only if simpler (library adoptions usually land here).
- More complex: the win must be worth maintaining the complexity — a modest win can justify modest complexity on a hot path; a cold path almost never earns added complexity.
- Hack: only an exceptional, measured win on a genuinely hot path, contained behind a clean interface and commented as such.
- Hot-path regressions in max/p99 latency weigh heavily against adoption; steady-state allocations on a realtime-tagged path remain a defect per the Realtime rules.
- Doubt → simple wins. A rationale you'd be embarrassed to read back is a rejection.

**Exit:** every candidate `WON` or `LOST`, with the rationale and actual numbers in the status table.

### ASSEMBLE

Build `<target>-v2/` from `WON` entries only; everything else is the **simplest faithful port** — fresh files per the no-copy rule, into a deliberately designed tree. Per component, in order (steps 1–2 and 4–5 per test-practice's TDD / Adversarial / Boundary enforcement sections):

1. **Class map:** equivalence classes from the public API contract plus BVA classes at every boundary (test-practice TDD). Contract-impossible inputs are **out of scope**: no breaking things under unrealistic expectations.
2. **Minimal test set:** main-flow tests first, then exactly one test per class — the smallest set where every class appears once; redundant tests are refused like redundant candidates. Written **red against the empty v2**.
3. **Port/integrate to green.**
4. **Edge audit:** one blind edge-hunter agent per component (test-practice Adversarial) — contract + class map + code only. Accepted claims become new classes with red tests, fixed per the fix protocol; rejections get a one-line reason.
5. **Boundary enforcement:** per trust boundary (test-practice Boundary enforcement) — standard mechanisms (bounds-checked types, hardening flags, sanitizer builds) enabled per the toolkit's Boundary enforcement section; hand-written validation gets its red/green pair. Findings block that part until fixed red-first.

**Exit:** v2 complete, class-map tests 1:1 green, edge audit triaged, every boundary enforced.

### PROVE

Re-measure every integrated part **in situ** via the gated instrumentation inside real v2 flows, plus end-to-end vs the v1 macro baselines. Microbenches judged the shootout; in-situ numbers judge integration. Won-isolated-but-regressed-integrated → `REGRESSED-REVERTED`, replaced by the simple port.

**Exit:** every adopted win confirmed in place; regressions reverted; tests green, boundary enforcement in place.

### PARITY (1 gap-hunter agent per pass — can force a new loop iteration)

One agent receives v1, v2, the feature map, and the `DROPS:` list — not the rewrite thread's reasoning. Mandate: find features present in v1 whose **intent** is absent in v2, and **triage severity itself**:

- **Design-defining** — a capability, mode, or guarantee users build around; losing it forces callers to hand-roll logic or restructure usage. Only these come back as `PARITY-CLAIM`s.
- **Small** — convenience overloads, incidental output details, precision nuances, one-line-recoverable behaviors. Returned as `SMALL-DIFF` notes: recorded, never actioned. Unsure → must argue "usage would restructure without it"; can't → small.

The **spirit rule** for judging: changed signatures/error models, different internals, float drift within reasonable tolerance, renamed/merged concepts = fine. Missing capability, silently narrowed input domain, dropped mode, lost guarantee = gap. "Line X has no equivalent" and "differs at the 5th decimal" are not claims.

Main thread adjudicates each claim: **refute with a trace** of how v2 serves the intent / **DROPPED-BY-DESIGN** (matches a declared drop) / **CONFIRMED-GAP**. An undeclared drop is always a confirmed gap — even if dropping was right, the decision must become explicit, never accidental.

**Confirmed gaps get the full pipeline, not band-aids:** each is registered in the status table and the rewrite loop **re-enters at IDEATE** for that feature set — ideate how the capability fits v2's design natively, refute, baseline against v1's implementation, shoot out alternatives, assemble red-first with class map + boundary enforcement, prove in situ. The result is indistinguishable from first-iteration work: same rules, same gates, same evidence.

**Exit:** a full gap-hunter pass (fresh agent) returns **zero new design-defining claims**. Cost per pass: exactly 1 agent call.

### REPORT

From the status table, cumulative:

- **Adopted** — per part: before/after numbers (median/p99/max, allocs) and complexity class.
- **Library adoptions** — what replaced hand-rolled code and why it won or tied.
- **Refuted / lost / reverted** — one line each ("31 ideas, 6 adopted, here's why the rest died").
- **Dropped features** — every `DROPPED-BY-DESIGN` with its rationale; gaps that were confirmed, re-entered, and covered.
- **Machine profile** and exact commands to re-run `bench/`.

Never dress up an interrupted run as complete; report the actual last status table as-is — that is the record of where the run stopped.
