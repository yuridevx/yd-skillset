---
name: algo-rewrite
description: Whole-codebase greenfield algorithmic rewrite pipeline for any language — inventory every algorithm, data flow, and feature; extract per-component contract cards that split observable behavior into PINNED / FLEXIBLE / SUSPECT under the intent-oracle rule (v1 witnesses the contract's spirit, never the algorithm; bug-for-bug compatibility is refused); triage through correctness / SIMD / concurrency / library-first / hot-cold lenses; then run each component through a pipeline — ideate greenfield replacements under a no-compatibility rule, adversarially refute each component's ideas in an independent comparative refuter that nominates an adversarial benchmark shape per survivor, settle survivors by benchmark shootout under measured noise (interleaved repetitions with recorded spread, contended runs for concurrent code, zero-alloc hot paths, proportional speed-vs-simplicity adoption), build fresh-file v2 components red-first with class-map tests, contract-level differential tests against v1, and property tests, then verify each through the adversarial spine — risk-gated blind edge-hunter, race oracle plus interleaving-hunter on concurrent components, deterministic boundary enforcement at every trust boundary, and a hardening-oracle pass — prove in situ against macro baselines, and audit feature parity with a gap-hunter agent that can force full re-iteration. Twin creeds — performance does not exist until a benchmark says it does; correctness does not exist until it survives adversarial refutation. Testing method loads from the test-practice skill (TDD, properties & differential, boundary enforcement, concurrent correctness, adversarial); language specifics load from a <lang>-toolkit skill (e.g. msvcpp-toolkit) or are derived at setup when none exists. Agent dispatch is leveled by a run-wide tag named for its policy — per-claim / batched (default — comparative per-component refuters, per-shootout triage, risk-gated hunters) / minimal / inline — chosen in the invocation or at ASK; oracles, differential suites, and the UNPROVEN veto are never leveled away, and inline runs record every not-blind judgment. Runs as a chain of named gates, each with a hard exit condition — one user question batch at the ASK gate, fully autonomous end-to-end when "autonomous" appears in the request. Trigger on "algo-rewrite", "/algo-rewrite", "algorithmic rewrite", "rewrite the algorithms", "optimize algorithms into v2", "make this algorithmically optimal", "rewrite for performance".
---

# algo-rewrite

Greenfield rewrite of a whole target into a sibling v2, gated by benchmarks and adversarial proof. Twin creeds:

- **Performance does not exist until a benchmark says it does.**
- **Correctness does not exist until it survives adversarial refutation** — an oracle-backed adversary that tried to kill it and failed.

Nothing is adopted on its author's word. The skill is a chain of **gates**: a gate is not a phase to schedule work into — it is a checkpoint with a hard exit condition; you are always standing at exactly one gate, doing its work, and you may not step past it until its exit condition is objectively green. There is no "later": work either happens at its gate or is killed at its gate.

The main thread runs linearly and owns all state. Independent agents (same model as the main thread) are the adversaries of the spine below, dispatched per the **Agent economy** level. Every agent is **blind** — it receives only the artifact under judgment (contract card, code, checklist), never the generating thread's reasoning — and follows test-practice's **Adversarial** calibration standard (claim bar, verbatim false-positive list, confidence labels).

## The adversarial spine

Every positive claim has a named adversary that must fail to kill it before the claim counts:

| Claim | Adversary | Where |
| --- | --- | --- |
| "this idea can win" | refuter agent with kill-checklist | REFUTE |
| "this prototype is correct" | triage agent + oracles (incl. race oracle) | SHOOTOUT |
| "this component is correct" | edge-hunter (inputs); interleaving-hunter (schedules) | VERIFY |
| "this boundary is safe" | boundary enforcement, red/green pair | VERIFY |
| "this is faster" | interleaved benchmark vs measured noise | SHOOTOUT / PROVE |
| "v2 serves every v1 intent" | gap-hunter | PARITY |
| "v1's behavior here is right" | correctness lens; SUSPECT adjudicated before pinning | TRIAGE / CONTRACT |

## Agent economy

Agent dispatch is leveled by one run-wide tag, named for its dispatch policy: **`per-claim` / `batched` (default) / `minimal` / `inline`**. The level rides the invocation ("per-claim", "minimal agents", "inline"; aliases: "lean"/"budget" → `minimal`, "solo"/"agentless" → `inline`) or is settled at ASK, and is recorded in the status-table header like every ASK-class decision.

| Level | REFUTE | SHOOTOUT triage | VERIFY hunters | PARITY |
| --- | --- | --- | --- | --- |
| `per-claim` | 1 agent per idea | 1 agent per prototype | edge-hunter on every built component; interleaving-hunter on every `CONCURRENT` one; always split | 1 agent/pass |
| `batched` (default) | 1 agent per component — judges its ideas **comparatively**: can rank, cannot approve contradictory ideas | 1 agent per shootout — A/B/C judged together on **one complexity scale** | edge-hunter only on `WON` / `HOT` / `TRUST-BOUNDARY` / `SUSPECT` components (a faithful port with a green contract-level differential already faced v1 as its edge adversary); interleaving-hunter on every `CONCURRENT`, split | 1 agent/pass |
| `minimal` | agents only for `HOT` / `CONCURRENT` / `SUSPECT` components; the rest refuted on paper by the main thread | as `batched` | one merged dual-mandate adversary (inputs + schedules) per risky component; split only where `HOT` and `CONCURRENT` coincide | 1 agent/pass |
| `inline` | all paper | main-thread triage | main-thread self-audit against the contract card + class map | main-thread sweep of the feature map + `DROPS:` |

Invariant at every level: oracles, sanitizers, boundary enforcement, differential + property suites, measured noise, red-first, and the UNPROVEN veto are agent-free and are **never leveled away**. **Escalation:** tags outrank the level — the main thread may raise a single component one level, recorded in the table; it may never lower one. `inline` is qualitatively different and the report must say so: **blindness is gone** — every not-blind judgment is recorded `NOT-BLIND` in the ledger, and the only true adversaries left are the oracles, the differential suite, and the benchmark. Use `inline` for tiny targets, agent-less environments, or a cheap first iteration to be re-verified at `batched` later.

## Flow map

```
SETUP → SCAN → CONTRACT → TRIAGE → ASK ══ only user stop (skipped if "autonomous")
                                    │
      ┌─────────────────────────────▼─────────────────────────────┐
      │ PER-COMPONENT PIPELINE (components flow independently)    │
      │ IDEATE → REFUTE → BASELINE → SHOOTOUT → BUILD → VERIFY    │
      │    ▲                                                      │
      └────┼─────────────────────────────────────┬────────────────┘
           │                        all components done
           │                                     ▼
           └── CONFIRMED-GAP features ◄────── PARITY ◄── PROVE
               (full pipeline, never a band-aid) │
                             zero design-defining claims
                                                 ▼
                                              REPORT
```

## Autonomy contract

- **Exactly one user interaction: the ASK gate.** Every question the run will ever need is batched there.
- Past ASK, **never** ask "shall I proceed", "say go", "want me to continue", or present intermediate results as questions. The only stops are a broken baseline at SETUP and completion at REPORT.
- If the invocation contains "autonomous", "auto", "no questions", or equivalent intent: **skip ASK entirely**, self-decide, run end to end. Either way every ASK-class decision is recorded in the dialog (user-answered or self-decided) so the report shows what was chosen and why.

## The intent-oracle rule

v1 is the **intent oracle, never the algorithm oracle**. It serves three roles until retired: baseline benchmark, contract-level differential oracle, parity reference. The CONTRACT gate splits every observable behavior:

- **PINNED** — the contract fixes the exact output → differential tests compare exact.
- **FLEXIBLE** — the contract fixes a predicate only (±tolerance, any valid ordering, error model free to change) → differential tests compare by equivalence predicate or property assertion.
- **SUSPECT** — the correctness lens doubts v1 here → **adjudicated before it may be pinned** (defect vs load-bearing quirk callers rely on). A defect is never pinned — bug-for-bug compatibility is refused; v2 asserts the adjudicated behavior and the adjudication is recorded.

Signatures, internals, data structures, error models, complexity: **FREE** — judged only by correctness (adversarial oracles), proof (REFUTE), benchmark (SHOOTOUT/PROVE), and simplicity (adoption judgment). Nothing else has a vote. Pinning an incidental behavior is a defect: it freezes the algorithm. Pinning a SUSPECT behavior is worse: it enshrines a bug as contract.

## The language toolkit

Every language-specific choice is delegated to a **toolkit**: at SETUP, detect the target language and invoke the matching `<lang>-toolkit` skill (e.g. `msvcpp-toolkit`). It supplies, in named sections: **Toolchain**, **Benchmarking** (harness order, instrumentation, allocation tracking), **Hot-path rules**, **SIMD / data-parallel** and **Concurrency** facilities, **Concurrency oracles** (race detector, interleaving forcing, contended-benchmark pattern), the **Library shortlist**, **Boundary enforcement**, and **Hardening oracles**.

No toolkit skill for this language → derive the same sections yourself at SETUP (web research encouraged) and state them in the dialog; the pipeline is identical either way.

## The testing rulebook

All test work follows the **test-practice** skill, invoked at SETUP. Its five named sections govern: **TDD** (red-first, class map + BVA, main-flow-first 1:1 set, fix protocol), **Properties & differential** (property tests, contract-level differential against v1, PINNED/FLEXIBLE/SUSPECT comparison rules), **Boundary enforcement**, **Concurrent correctness** (synchronization contract, race-oracle-as-red-test, contended stress, UNPROVEN rule), **Adversarial** (blind hunter protocol + calibration standard). This skill's gates say *when* those sections run and what gets recorded; test-practice says *how*; the toolkit says *with what*.

## Universal rules (govern every gate)

### Red-first

**No code exists before its oracle does** — full protocol in test-practice's TDD section. Every unit of work opens red — a failing test or a recorded baseline benchmark — and closes green:

- Harness self-check before the harness measures anything (SETUP).
- Per prototype: main-flow + edge tests must pass before its benchmark counts (SHOOTOUT). Prototypes are throwaways — the independent correctness gate, not test ordering, is what validates them.
- Per v2 component: class map → 1:1 tests failing red against empty v2 → port to green → differential + property tests green → adversarial verification (BUILD, VERIFY).
- Per fix (triage failure, boundary-enforcement finding, hunter claim, regression, parity gap): failing repro/intent test → minimal fix to green → test kept as regression guard.
- Benchmark-first is the performance twin: no optimization is implemented before the number it must beat is recorded.

### No-copy

**Never copy existing code and edit it — rewrite into new files as the work needs them.** Applies to prototypes, benchmarks, tests, harnesses, and the v2 tree alike. A "port" is a rewrite from the component's contract card and behavior (original open as reference), never a file copy: copies smuggle in old structure, dead code, and compatibility residue — exactly what the no-compat rule exists to kill.

The new file tree is designed, not accreted: one clear responsibility per file, module layout per the project's conventions, directories mirroring the component inventory from SCAN. Never dump everything into one file — a v2 that wins every benchmark but reads like a heap is a failed rewrite under the simplicity rule.

### Realtime (hot paths only)

Hot path = any code on the per-item / per-frame / per-request path, tagged at TRIAGE. Cold paths are exempt and judged on simplicity + throughput only.

1. **Zero allocations steady-state.** Memory acquired at init: preallocation, arenas/pools, buffer reuse, reused scratch. Warmup allocation fine; per-iteration allocation (including GC pressure in managed languages) is a defect.
2. **No hidden allocators.** The toolkit's Hot-path rules list the language's silent allocation sources (closures/boxing, string temporaries, container growth/rehash, exception machinery, …); hot paths avoid all of them.
3. **Bounded worst case beats better average.** No amortized-only guarantees where the spike lands on the hot path; a container that cannot pre-reserve to a known bound is the wrong design. In managed languages a GC pause triggered by hot-path garbage is a worst-case spike you caused.
4. **No blocking:** no lock waits, syscalls, I/O, logging (ring-buffer deferred logging if needed). Concurrency via lock-free structures, per-thread ownership, or SPSC/MPSC stage handoff — all of which are `CONCURRENT`-tagged and owe the full Concurrent-correctness treatment; a fast structure that races is not fast, it is wrong.
5. **Errors without unwinding or allocating** on the hot path — value/status returns per the toolkit's error model; exceptions/panics are setup/teardown territory.
6. **Layout matches access pattern:** contiguous over node-based, SoA where the SIMD lens demands, padding against false sharing where the language exposes layout.

Enforcement: allocation tracking (the toolkit's mechanism) is wired into every benchmark; **hot-path benchmarks assert 0 steady-state allocations** — nonzero fails regardless of speed. Latency is **median + p99 + max**; winning median while regressing max on a hot path is a loss.

### Measured noise

Noise is measured, not assumed. Every benchmark comparison runs baseline and candidate **interleaved** (the harness's random-interleaving mode where it has one), with the repetition count the toolkit's Benchmarking section prescribes, and the **run-to-run spread is recorded in the status table next to the numbers** — a win must clear the recorded spread, not an asserted one. The harness must defeat dead-code elimination per the toolkit. Bench shapes: realistic small / typical / large **plus every adversarial shape nominated at REFUTE**. `CONCURRENT` components benchmark **contended** per the toolkit's Concurrency-oracles pattern — uncontended numbers for a concurrent structure are not numbers.

## State

Two real artifacts on disk; everything else lives in the conversation:

```
algo-rewrite/            (scratchpad by default; in-repo if chosen at ASK)
├── bench/               harness, instrumentation, raw results (kept for reproducibility)
└── <target>-v2/         the rewrite — nearest sibling folder to the original target
```

Working state is an in-dialog status table — one line per component, idea, and feature: id, tags (`HOT/COLD`, `CONCURRENT`, `TRUST-BOUNDARY`, `SUSPECT`), current status, the numbers (with recorded spread) or one-line reason that justify it, and — once VERIFY has run — the component's **verification ledger**: which adversaries ran and claims-vs-actioned. Status vocabulary: components end `REWRITTEN | KEPT-AS-IS`; ideas end `REFUTED | LOST | WON | REGRESSED-REVERTED`; features end `COVERED | SMALL-DIFF | DROPPED-BY-DESIGN` (via `CONFIRMED-GAP` when PARITY forces re-entry); SUSPECT behaviors end `ADJUDICATED-DEFECT | ADJUDICATED-QUIRK`. Header: `Assumptions:` (every self-made call), target, language + toolkit (or derived sections), toolchain, machine profile, oracle availability, **agent-economy level**, iteration counter.

Restate the current table compactly at every gate exit — never drop a line, whatever its status; the restatement is what carries the record through context compaction. Anything an agent needs (idea, code, contract card, checklist, class map, feature map, `DROPS:` list) is passed explicitly in its prompt; its verdict comes back in its reply and lands in the table.

## The gates

### SETUP

1. Determine target (diff, given paths, or whole repo). Ambiguity → take the reasonable reading, record under `Assumptions:`.
2. Verify the project **builds and existing tests pass** — broken baseline → stop and report; nothing can be baselined on red.
3. **Resolve the language toolkit** and **invoke the test-practice skill**; record both — or the derived toolkit sections — in the dialog.
4. Detect the toolchain per the toolkit's Toolchain section (compiler/runtime, SIMD ISA where relevant, threads, package manager).
5. **Confirm oracle availability on this machine** — especially the race oracle from the toolkit's Concurrency oracles. A missing oracle never blocks the run; it dooms specific claims: a `CONCURRENT` design whose race oracle is unavailable here ends `UNPROVEN` and defaults to the simple/sequential alternative — never adopted on suspicion.
6. Install the benchmark harness **library-first** in the toolkit's Benchmarking order; hand-rolled timing only if nothing installs.
7. Set up bench instrumentation per the toolkit: scope/count/alloc-guard hooks gated on a build flag; without the flag they compile to **nothing** — no branch, no atomic, no string — so they may live in product code (v1 and v2) permanently. Red-first self-check: with the flag on, the alloc guard catches a deliberate allocation; with it off, the same code builds to a no-op — assert both.

**Exit:** build green, tests pass, harness self-check red→green, setup summary stated (target, language/toolkit, toolchain, oracle availability, CPU/cores/caches — numbers are meaningless without the machine profile).

### SCAN

Read the whole target. Record every **algorithmic component** (code that loops over data, searches, sorts, hashes, parses, allocates in a pattern, converts representations), every **data flow** (entry shape, each transformation, each copy, each intermediate), and the **feature map** — what the target does *for its users* at intent level: capability, why a user needs it, observable behavior, feature branches (modes, options, config-driven behaviors, documented edge semantics), where v1 implements it. **Guarantees are features too:** ordering stability, thread-safety promises, documented tolerances, complexity bounds callers rely on — the easiest things to lose silently.

Every component gets one status-table line (location, role, algorithm). Full detail — **CPU + memory complexity, allocation behavior, data structures** — only for components plausibly hot or plausibly rewritable; a cold config-parser doesn't need its complexity derived to be tagged `KEPT-AS-IS`. Inventory only — no judging, no ideas.

**Exit:** every component, flow, and feature in the status table; rewrite candidates with complexity recorded.

### CONTRACT

Per rewrite-candidate component (a cold `KEPT-AS-IS` component needs no card), extract one **contract card** — the single artifact every downstream adversary receives:

- Public intent: what callers get, in one paragraph.
- Invariants and guarantees callers rely on (ordering, tolerances, complexity bounds, thread-safety promises).
- **Synchronization contract** where state is shared (per test-practice's Concurrent correctness): what's shared, who owns it, ordering/visibility assumptions, what may block.
- Trust boundaries the component sits on.
- Statable properties (permutation, round-trip, invariant preservation — per test-practice's Properties & differential).
- The **PINNED / FLEXIBLE / SUSPECT** split of every observable behavior, per the intent-oracle rule.

The card is contract, not implementation: it must not name v1's algorithm or data structures — that is exactly the freedom the FREE rule protects.

**Exit:** every rewrite candidate has a card; every SUSPECT behavior is adjudicated or queued for ASK.

### TRIAGE

Five lenses, one pass each:

1. **Correctness:** defect-risk scan with harden's four classes as the checklist — boundary, concurrency, lifetime/resource, garbage data. Emits `SUSPECT` tags on doubted behaviors (feeding CONTRACT's adjudication) and on components, and owns the `TRUST-BOUNDARY` tag — its boundary class names every component sitting on external input (parsers, network data, file formats); **a suspected-wrong component is a first-class rewrite candidate — rewrite-because-wrong ranks with rewrite-because-slow.**
2. **SIMD / data-parallel:** per the toolkit's facilities — vectorizable / vectorizable-after-restructure (AoS→SoA etc.) / not. If the toolkit says the language has no practical SIMD story, this lens narrows to vectorization-friendly layout and bulk operations.
3. **Concurrency:** current threading costs (contention, false sharing, oversubscription) + parallelizable-but-serial components, judged against the toolkit's concurrency facilities — **and the `CONCURRENT` tag**, applied to everything that shares mutable state today *and* everything this lens proposes to parallelize; the tag is what routes a component into the Concurrent-correctness treatment downstream.
4. **Library** (dedicated pass — web research explicitly encouraged): does a proven public library already do this? Start from the toolkit's Library shortlist, extend by research. A hit becomes the idea *adopt library X*; hand-rolled competitors must beat it head-to-head at SHOOTOUT. Ties go to the library.
5. **Hot/cold:** tag every component; hot → realtime rules apply.

**Exit:** every component tagged on all five lenses (`HOT/COLD`, `CONCURRENT`, `TRUST-BOUNDARY`, `SUSPECT` where earned).

### ASK — the only user stop

Present in one batch: assumptions taken, inventory summary with tags, **SUSPECT adjudications** (defect vs quirk — with the recommended verdict each), the **agent-economy level** (when the invocation didn't set it), library candidates, planned feature drops, and every genuine either-way call (artifact location, disputed tags, scope). One question round; answers are recorded. In autonomous mode: skip, self-decide, record.

**Exit:** every ASK-class decision recorded, every SUSPECT behavior `ADJUDICATED-DEFECT` or `ADJUDICATED-QUIRK`. From here to REPORT, zero questions — every later decision is resolved by the recorded decisions, the adoption judgment, or a benchmark.

### IDEATE (pipeline entry — re-entered by PARITY)

Per component, generate alternatives under **no-compatibility, no-fallback** rules: only the **spirit** of the public API must survive — same capabilities, free to change signatures, ownership, error model. No shims, no dual paths. Killing a component by changing the upstream representation is a first-class idea — the best optimization is deletion. Web research for state-of-the-art approaches encouraged.

An idea that deletes or narrows a capability must declare `DROPS: <feature> — <rationale>` (surfaced at ASK, or self-decided and recorded). This is what lets PARITY distinguish deliberate removal from accidental loss.

Simplicity at the door: every idea states its complexity cost. Hot-path ideas must obey the realtime rules; obvious violations die here. So does an idea whose plausible win is clearly too small to carry its cost — the same proportionality the adoption judgment applies at SHOOTOUT.

**Exit:** every live component has its ideas recorded, each with complexity cost and any `DROPS:` declared.

### REFUTE (independent adversaries, dispatched per the Agent economy)

One blind refuter per **component**, given only that component's ideas, the contract card, the code, and the kill-checklist; it judges the ideas **comparatively** — it may rank them, and it may not approve two mutually contradictory ones. (`per-claim`: one refuter per idea; `minimal`: agents only for `HOT` / `CONCURRENT` / `SUSPECT` components, the main thread refutes the rest on paper; `inline`: all paper.) The kill-checklist:

- Complexity math holds at **realistic N**? (O(n log n)→O(n) is noise at n ≤ 64.)
- Constant factors: cache behavior, branch predictability, allocation count — not just big-O.
- SIMD/data-parallel: layout actually amenable, or does gather/scatter eat the win?
- Concurrency: enough work per task to amortize synchronization? **Is the idea's synchronization contract actually implementable** — no ABA-shaped design, no ordering assumption the platform doesn't grant?
- Library: actually covers the use case (read the docs), maintained, installs on this toolchain?
- Realtime compliance for hot-path ideas.

Whoever refutes — agent or main thread — also **nominates one adversarial benchmark shape per surviving idea** — the input on which that idea should be weakest; it joins the component's bench shapes for BASELINE and SHOOTOUT.

Verdict rule — **doubt flows forward** (the benchmark downstream is a cheap objective judge); certainty-it-cannot-win or a simplicity violation → `REFUTED` with a one-line reason.

**Exit:** every idea dispositioned; surviving ideas each carry a nominated adversarial shape; main thread has recorded all verdicts.

### BASELINE

For every component with a surviving idea: benchmark **current v1** under the Measured-noise rule — realistic shapes at small/typical/large sizes plus the nominated adversarial shapes; median + p99 + max latency with recorded spread, allocation count, peak memory; contended where `CONCURRENT`. Also capture in-situ macro baselines on v1 — these are PROVE's bar.

**Exit:** the number (and spread) every candidate must beat is recorded. No prototype exists before its baseline does.

### SHOOTOUT

Per component: minimal prototypes of each surviving idea (A = v1, B/C/… = candidates), same harness, same data. Prototype only the algorithmic core — no productionizing losers. Order per candidate (red-first): main-flow + edge tests written failing → implement to green → benchmark.

**Correctness gate before any benchmark counts:** one independent triage agent per component's shootout judges **all its prototypes together** — one complexity scale for every candidate (`per-claim`: one agent per prototype; `inline`: main-thread triage, recorded `NOT-BLIND`) — checking breakage, edge cases, safe coding (boundaries, overflow, lifetime, undefined behavior where the language has it), realtime compliance — **and, for `CONCURRENT` prototypes, race-freedom: the race oracle must be silent under bounded contended stress before the benchmark counts** (per test-practice's Concurrent correctness; no oracle available → the prototype's correctness ends `UNPROVEN` and it may not win). Fail → fix and **re-benchmark** (the fix may eat the win), or `LOST`. The triage adversary (at `inline`: the main thread — the class is then self-graded and flagged as such) also assigns the **complexity class**: simpler/same, more complex, or hack (fragile cleverness, UB-adjacent tricks, layout gymnastics). Libraries replacing hand-rolled code count as **simpler** — complexity we don't maintain doesn't count against us.

**Adoption judgment — proportionality, not gates.** The four judges are correctness, proof, benchmark, simplicity — nothing else has a vote. One principle: the burden of proof grows with the complexity a candidate adds. Weigh the measured win (its size, its spread, and how hot the path is) against the complexity class the triage agent assigned, and decide per candidate; the verdict plus a one-line rationale goes in the status table — the rationale is the gate. Anchors:

- A result inside the recorded run-to-run spread is not a win — that's measurement, not judgment.
- Simpler or same complexity: any real win adopts; a tie adopts only if simpler (library adoptions usually land here).
- More complex: the win must be worth maintaining the complexity — a modest win can justify modest complexity on a hot path; a cold path almost never earns added complexity.
- Hack: only an exceptional, measured win on a genuinely hot path, contained behind a clean interface and commented as such.
- Hot-path regressions in max/p99 latency weigh heavily against adoption; steady-state allocations on a realtime-tagged path remain a defect per the Realtime rules.
- An `UNPROVEN` correctness claim vetoes adoption regardless of the numbers — fast-but-unproven loses to slow-but-proven.
- Doubt → simple wins. A rationale you'd be embarrassed to read back is a rejection.

**Exit:** every candidate `WON` or `LOST`, with the rationale and actual numbers (plus spread) in the status table.

### BUILD

Build the component's place in `<target>-v2/` from its `WON` entry — or, with no winner, as the **simplest faithful port**; fresh files per the no-copy rule, into a deliberately designed tree. Per component, in order (per test-practice's TDD and Properties & differential sections):

1. **Class map:** equivalence classes from the contract card plus BVA classes at every boundary. Contract-impossible inputs are **out of scope**: no breaking things under unrealistic expectations.
2. **Minimal test set:** main-flow tests first, then exactly one test per class — written **red against the empty v2**.
3. **Port/integrate to green.**
4. **Differential vs v1, at contract level:** PINNED behaviors compare exact; FLEXIBLE behaviors compare by equivalence predicate; SUSPECT behaviors assert the **adjudicated** behavior, not v1's — an `ADJUDICATED-DEFECT` dies here, by design, and its fix is recorded.
5. **Property tests** green for every property the card states.

**Exit:** component in v2, class-map tests 1:1 green, differential + property suites green against the card.

### VERIFY — adversarial proof of correctness

Per built component, depth keyed by its tags:

1. **Edge-hunter — risk-gated:** components that are `WON`, `HOT`, `TRUST-BOUNDARY`, or `SUSPECT` get one blind edge-hunter (test-practice Adversarial), receiving the contract card, class map, and code only. A faithful port whose contract-level differential suite is green skips the hunter — v1 already served as its edge adversary — and the skip is recorded in the ledger. (`per-claim`: every built component gets the hunter; `inline`: main-thread self-audit, recorded `NOT-BLIND`.) Accepted claims become new classes with red tests, fixed per the fix protocol; rejections get a one-line reason.
2. **`TRUST-BOUNDARY` — boundary enforcement** (test-practice Boundary enforcement): standard mechanisms enabled per the toolkit; hand-written validation gets its red/green pair. Findings block the component until fixed red-first.
3. **`CONCURRENT` — race oracle + interleaving-hunter:** contended stress under the race oracle (test-practice Concurrent correctness), then one blind interleaving-hunter receiving the synchronization contract and code. (`minimal`: the edge and interleaving mandates merge into one dual-mandate adversary — except a component both `HOT` and `CONCURRENT` keeps them split; `inline`: self-audit, recorded `NOT-BLIND`.) No race oracle on this machine → the component's concurrent claim ends `UNPROVEN` and the design reverts to the simple/sequential port — never shipped on suspicion.
4. **All — hardening pass:** run the toolkit's Hardening-oracles builds (address/UB sanitizers and equivalents) over the component's test suite; any finding is a defect, fixed per the fix protocol.

The component's verification ledger (adversaries run, claims vs actioned) lands in the status table.

**Exit:** every hunter pass triaged, every boundary enforced, oracles silent (or `UNPROVEN` recorded and the design reverted), ledger recorded.

### PROVE

Re-measure every integrated part **in situ** via the gated instrumentation inside real v2 flows, plus end-to-end vs the v1 macro baselines — same Measured-noise rule. If anything is `CONCURRENT`, one whole-system contended run. Microbenches judged the shootout; in-situ numbers judge integration. Won-isolated-but-regressed-integrated → `REGRESSED-REVERTED`, replaced by the simple port.

**Exit:** every adopted win confirmed in place; regressions reverted; tests green, boundary enforcement in place.

### PARITY (one gap-hunter pass — an agent at every level above `inline` — can force a new pipeline iteration)

The gap-hunter (one blind agent; at `inline`, the main thread, recorded `NOT-BLIND`) receives v1, v2, the feature map, the `DROPS:` list, and the SUSPECT adjudication record — not the rewrite thread's reasoning. Mandate: find features present in v1 whose **intent** is absent in v2, and **triage severity itself**:

- **Design-defining** — a capability, mode, or guarantee users build around; losing it forces callers to hand-roll logic or restructure usage. Only these come back as `PARITY-CLAIM`s.
- **Small** — convenience overloads, incidental output details, precision nuances, one-line-recoverable behaviors. Returned as `SMALL-DIFF` notes: recorded, never actioned. Unsure → must argue "usage would restructure without it"; can't → small.

The **spirit rule** for judging: changed signatures/error models, different internals, float drift within reasonable tolerance, renamed/merged concepts = fine. Missing capability, silently narrowed input domain, dropped mode, lost guarantee = gap. **A fixed `ADJUDICATED-DEFECT` is not a gap** — it matches the adjudication record. "Line X has no equivalent" and "differs at the 5th decimal" are not claims.

Main thread adjudicates each claim: **refute with a trace** of how v2 serves the intent / **DROPPED-BY-DESIGN** (matches a declared drop) / **CONFIRMED-GAP**. An undeclared drop is always a confirmed gap — even if dropping was right, the decision must become explicit, never accidental.

**Confirmed gaps get the full pipeline, not band-aids:** each is registered in the status table and re-enters at IDEATE for that feature set — ideate how the capability fits v2's design natively, refute, baseline against v1's implementation, shoot out alternatives, build red-first, verify through the full spine, prove in situ. The result is indistinguishable from first-iteration work: same rules, same gates, same evidence.

**Exit:** a full gap-hunter pass (a fresh adversary) returns **zero new design-defining claims**. Cost per pass: exactly 1 agent call at every level above `inline` — the gap-hunter is never batched, merged, or risk-gated away. At `inline` the main thread sweeps the feature map + `DROPS:` itself, checklist-driven, and the pass is recorded `NOT-BLIND`.

### REPORT

From the status table, cumulative:

- **Adopted** — per part: before/after numbers (median/p99/max with spread, allocs) and complexity class.
- **Library adoptions** — what replaced hand-rolled code and why it won or tied.
- **Refuted / lost / reverted** — one line each ("31 ideas, 6 adopted, here's why the rest died").
- **SUSPECT adjudications** — every defect fixed-by-design and every quirk deliberately preserved.
- **Dropped features** — every `DROPPED-BY-DESIGN` with its rationale; gaps that were confirmed, re-entered, and covered.
- **Verification ledger** — per component: which adversaries ran (and at what agent-economy level), claims vs actioned, every `NOT-BLIND` judgment, anything left `UNPROVEN` and what was done about it.
- **Machine profile** and exact commands to re-run `bench/`.

Never dress up an interrupted run as complete; report the actual last status table as-is — that is the record of where the run stopped.

**Exit:** the report delivered from the final status table, nothing omitted — every component, idea, feature, and adjudication accounted for.
