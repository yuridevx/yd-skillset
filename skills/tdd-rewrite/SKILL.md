---
name: tdd-rewrite
description: Whole-project greenfield TDD rewrite pipeline for any language — tests are the spec, not the guard: study the original implementation and design, inventory every feature, flow, and use case; extract per-use-case cards that split observable behavior into PINNED / FLEXIBLE / SUSPECT under the intent-oracle rule (v1 witnesses intent, never the implementation; bug-for-bug compatibility is refused); triage through correctness / algorithmic-depth / concurrency / scale / hot-cold lenses; settle the v2 architecture by multi-candidate design — two or three greenfield sketches judged together by one blind comparative refuter with testability as the first judge, the winner becoming the run-wide design brief every v2 module is built on; then run each use case through a pipeline — author a tiered test plan first (T1 happy paths, T2 algorithmic edges via equivalence partitioning and boundary-value analysis with property and contract-level differential items, T3 concurrency plus stress only where tagged), adversarially refute the plan with an independent coverage-hunting refuter that nominates the nastiest missed inputs and schedules as new plan items, capture expected values by executing the plan's differential items against v1 (executed, never asserted — v1 failing its own item is adjudicated, never papered over), build strictly red-first up the tier ladder into fresh v2 files with every mid-build discovery recorded as a counted plan amendment, verify each through the adversarial spine — risk-gated blind edge-hunter, race oracle plus interleaving-hunter on concurrent use cases, deterministic boundary enforcement at every trust boundary, and a hardening-oracle pass — prove in situ with the full suite, cross-use-case flows, a final differential sweep before v1 retires, and a crude v1-vs-v2 timing note that warns but never gates, and audit feature parity with a gap-hunter agent that can force full re-iteration. Twin creeds — behavior does not exist until a red test demands it; coverage does not exist until it survives adversarial refutation. Testing method loads from the test-practice skill (TDD, properties & differential, boundary enforcement, concurrent correctness, adversarial); language specifics load from a <lang>-toolkit skill (e.g. msvcpp-toolkit) resolved on the target language, or are derived at setup when none exists. Agent dispatch is leveled by a run-wide tag named for its policy — per-claim / batched (default: per-use-case plan refuters, risk-gated hunters) / minimal / inline — chosen in the invocation or at ASK; red-first, the tier ladder, executed capture, differential and property suites, boundary enforcement, oracles, the UNPROVEN veto, the design brief, and the parity audit are never leveled away, and inline runs record every not-blind judgment. Runs as a chain of named gates, each with a hard exit condition — one user question batch at the ASK gate (where the design brief is confirmed), fully autonomous end-to-end when "autonomous" appears in the request. Trigger on "tdd-rewrite", "/tdd-rewrite", "TDD rewrite", "test-driven rewrite", "test-first rewrite", "rewrite test-first", "rewrite with tests first", "greenfield TDD rewrite".
---

# tdd-rewrite

Greenfield rewrite of a whole project into a sibling v2, driven by adversarially-refuted test plans: every upfront study — implementation, design, feature triage — converges into a tiered plan per use case, and implementation is TDD to green. The tests are the spec, not the guard. Twin creeds:

- **Behavior does not exist until a red test demands it** — v2 contains no line of code a failing test didn't force into existence.
- **Coverage does not exist until it survives adversarial refutation** — a plan is complete when a blind adversary hunting holes fails to find one, not when its author feels done.

Nothing is adopted on its author's word. The skill is a chain of **gates**: a gate is not a phase to schedule work into — it is a checkpoint with a hard exit condition; you are always standing at exactly one gate, doing its work, and you may not step past it until its exit condition is objectively green. There is no "later": work either happens at its gate or is killed at its gate.

The main thread runs linearly and owns all state. Independent agents (same model as the main thread) are the adversaries of the spine below, dispatched per the **Agent economy** level. Every agent is **blind** — it receives only the artifact under judgment (use-case card, class map, test plan, design sketch, code), never the generating thread's reasoning — and follows test-practice's **Adversarial** calibration standard (claim bar, verbatim false-positive list, confidence labels).

## The adversarial spine

Every positive claim has a named adversary that must fail to kill it before the claim counts:

| Claim | Adversary | Where |
| --- | --- | --- |
| "this architecture is buildable test-first" | comparative design-refuter with kill-checklist | DESIGN |
| "this plan covers the use case" | plan-refuter + nominated adversarial cases | REFUTE |
| "this expected value is real" | execution against v1 — captured, never asserted | CAPTURE |
| "this code implements the contract" | the red suite that demanded it + differential vs v1 | BUILD |
| "this use case survives hostile inputs" | edge-hunter | VERIFY |
| "this boundary is safe" | boundary enforcement, red/green pair | VERIFY |
| "this concurrent design is sound" | race oracle + interleaving-hunter | VERIFY |
| "v2 serves every v1 intent" | gap-hunter | PARITY |
| "v1's behavior here is right" | correctness lens; SUSPECT adjudicated before pinning | TRIAGE / CASE |

## Agent economy

Agent dispatch is leveled by one run-wide tag, named for its dispatch policy: **`per-claim` / `batched` (default) / `minimal` / `inline`**. The level rides the invocation ("per-claim", "minimal agents", "inline"; aliases: "lean"/"budget" → `minimal`, "solo"/"agentless" → `inline`) or is settled at ASK, and is recorded in the status-table header like every ASK-class decision.

| Level | DESIGN | REFUTE | VERIFY hunters | PARITY |
| --- | --- | --- | --- | --- |
| `per-claim` | 1 comparative refuter | 1 refuter per tier | edge-hunter on every built use case; interleaving-hunter on every `CONCURRENT` one; always split | 1 agent/pass |
| `batched` (default) | 1 comparative refuter | 1 refuter per use case — judges its tiers **together** on one coverage scale | edge-hunter only on `DEEP` / `HOT` / `TRUST-BOUNDARY` / `SUSPECT` use cases (an untagged plan already survived a blind refuter, and its differential faced v1); interleaving-hunter on every `CONCURRENT`, split | 1 agent/pass |
| `minimal` | 1 comparative refuter | refuters only for `DEEP` / `HOT` / `CONCURRENT` / `TRUST-BOUNDARY` / `SUSPECT` use cases; the rest refuted on paper by the main thread | one merged dual-mandate adversary (inputs + schedules) per risky use case; split only where `HOT` and `CONCURRENT` coincide | 1 agent/pass |
| `inline` | main-thread paper comparison | all paper | main-thread self-audit against the card + class map | main-thread sweep of the feature map + `DROPS:` |

Invariant at every level: red-first, the tier ladder, executed capture, differential + property suites, boundary enforcement, oracles, the UNPROVEN veto, the design brief, and the perf note are agent-free and are **never leveled away**. **Escalation:** tags outrank the level — the main thread may raise a single use case one level, recorded in the table; it may never lower one. `inline` is qualitatively different and the report must say so: **blindness is gone** — every not-blind judgment is recorded `NOT-BLIND` in the ledger, and the only true adversaries left are the red suites, the captured differential, and the oracles. Use `inline` for tiny targets, agent-less environments, or a cheap first iteration to be re-verified at `batched` later.

## Flow map

```
SETUP → SCAN → CASE → TRIAGE → DESIGN → ASK ══ only user stop (skipped if "autonomous")
                                         │
      ┌──────────────────────────────────▼───────────────────────┐
      │ PER-USE-CASE PIPELINE (runs in the brief's build order)  │
      │ PLAN → REFUTE → CAPTURE → BUILD → VERIFY                 │
      │   ▲                                                      │
      └───┼───────────────────────────────────┬──────────────────┘
          │                      all use cases done
          │                                   ▼
          └── CONFIRMED-GAP features ◄───── PARITY ◄── PROVE
              (full pipeline, never a band-aid) │
                            zero design-defining claims
                                                ▼
                                             REPORT
```

## Autonomy contract

- **Exactly one user interaction: the ASK gate.** Every question the run will ever need — including the design-brief confirmation — is batched there.
- Past ASK, **never** ask "shall I proceed", "say go", "want me to continue", or present intermediate results as questions. The only stops are a dead v1 at SETUP (absent an explicit rewrite-anyway) and completion at REPORT.
- If the invocation contains "autonomous", "auto", "no questions", or equivalent intent: **skip ASK entirely**, self-decide (including the design brief), run end to end. Either way every ASK-class decision is recorded in the dialog so the report shows what was chosen and why.

## The intent-oracle rule

v1 is the **intent oracle, never the implementation oracle**. It serves four roles until retired: expected-value source (CAPTURE), contract-level differential oracle (BUILD, PROVE), parity reference (PARITY), and the crude timing reference for the perf note. The CASE gate splits every observable behavior:

- **PINNED** — the contract fixes the exact output → differential items compare exact, against captured values.
- **FLEXIBLE** — the contract fixes a predicate only (±tolerance, any valid ordering, error model free to change) → differential items compare by equivalence predicate or property assertion.
- **SUSPECT** — the correctness lens doubts v1 here → **adjudicated before it may be pinned** (defect vs load-bearing quirk callers rely on). A defect is never pinned — bug-for-bug compatibility is refused; v2's tests assert the adjudicated behavior and the adjudication is recorded.

Signatures, internals, data structures, error models, module layout, complexity: **FREE** — judged only by the refuted plan, the suites it becomes, the design brief, and simplicity. Nothing else has a vote. Pinning an incidental behavior is a defect: it freezes the design. Pinning a SUSPECT behavior is worse: it enshrines a bug as contract.

## The design brief — one run-wide artifact

Every v2 module is built on **one design brief** — the winner of DESIGN's multi-candidate refutation, confirmed at ASK (self-decided in autonomous mode). One page:

- **Module map** — one responsibility per module, acyclic dependency direction, foundation first.
- **Public surfaces** — the API each use case enters through; these are the surfaces test plans target and tests drive.
- **Error model** — how failures surface: observable at the surfaces, consistent across modules, never swallowed.
- **Concurrency posture** — what is concurrent in v2, under what synchronization contract, and the named **sequential fallback** for any posture whose oracle turns out unavailable (the UNPROVEN veto's landing zone).
- **Build order** — which use case drives which module into existence first; the schedule the pipeline follows.

The brief is contract for BUILD: a module that greens its tests but breaks the brief's layout is a failed build. Drift found later is a defect against the brief, not a reason to grow it — the brief only changes by explicit decision recorded in the status table.

## The tiered test plan

The settle mechanism — this skill's analog of algo-rewrite's benchmark. Per use case, derived from the card (never from v1's code), three tiers:

- **T1 — happy paths.** The main flows end-to-end through the public surfaces with realistic data, per test-practice's main-flow-first rule. Every use case, always.
- **T2 — algorithmic edges.** The class map (equivalence partitioning + BVA per test-practice's TDD), exactly one test per class; property tests for every statable invariant; **differential items** against v1 — PINNED compare exact, FLEXIBLE compare by predicate, SUSPECT assert the adjudicated behavior. Depth scales with `DEEP`; existence is never waived.
- **T3 — concurrency + stress.** Risk-gated: `CONCURRENT` → synchronization-contract tests, bounded contended stress under the race oracle, deterministic interleaving forcing (test-practice's Concurrent correctness); `SCALE` → volume at contract maxima, endurance, resource-ceiling assertions. Neither tag → `SKIPPED-UNTAGGED`, recorded.

Every item carries: id, tier, the class/property/behavior it covers, kind (main-flow / class / BVA / property / differential / boundary / race / interleaving / stress), its oracle, and its expected-value source (**contract | v1-capture | adjudication**). The plan's **coverage claims** — what REFUTE attacks: every class exactly once (minimal 1:1), every stated property an item, every PINNED behavior a differential item, every synchronization contract T3 items, every trust boundary boundary items.

v1's own tests are mined at SCAN as intent evidence — classes they encode, regressions they remember, tolerances they assert — and rewritten at contract level. Never copied: copied tests pin incidentals (the no-copy rule applies to tests).

## The language toolkit

Every language-specific choice is delegated to a **toolkit**: at SETUP, detect the **target** language and invoke the matching `<lang>-toolkit` skill (e.g. `msvcpp-toolkit`) — the target's, so cross-language rewrites work; the CAPTURE adapter bridges to v1's language at contract level. Consumed sections: **Toolchain**, **Boundary enforcement**, **Concurrency oracles** (race detector, interleaving forcing, contended-stress pattern), **Hardening oracles**, and the **Library shortlist** (test frameworks first). The Benchmarking and SIMD sections are not consumed — PROVE's perf note is crude wall-clock by design; speed campaigns belong to perf-tune and algo-rewrite.

No toolkit skill for this language → derive the same sections yourself at SETUP (web research encouraged) and state them in the dialog; the pipeline is identical either way.

## The testing rulebook

All test work follows the **test-practice** skill, invoked at SETUP. Its five named sections govern: **TDD** (red-first, class map + BVA, main-flow-first 1:1 set, fix protocol), **Properties & differential** (property tests, contract-level differential against v1, PINNED/FLEXIBLE/SUSPECT comparison rules, green-before-deletion), **Boundary enforcement**, **Concurrent correctness** (synchronization contract, race-oracle-as-red-test, contended stress, UNPROVEN rule), **Adversarial** (blind hunter protocol + calibration standard). This skill's gates say *when* those sections run and what gets recorded; test-practice says *how*; the toolkit says *with what*.

## Universal rules (govern every gate)

### Red-first — the tier ladder

**No v2 code exists before a red plan item demands it** — full protocol in test-practice's TDD section. Per use case the ladder is strict: T1 written red against the empty v2 → implemented to green → T2 red → green → T3 red → green; a tier opens only when the previous one is green. Per fix (refuter nomination, hunter claim, boundary finding, regression, parity gap): failing repro first → minimal fix to green → test kept as regression guard. The harness itself opens red: one deliberately failing test observed red at SETUP before anything else counts.

**Plan amendments:** implementation may reveal a behavior no item demanded. The item is added to the plan first (red), then the code — never code without an item. Every amendment is recorded and counted per use case, and the count is reported: a high amendment count is a PLAN/REFUTE failure surfaced, never hidden.

### No-copy

**Never copy v1 code or v1 tests — rewrite into fresh files from the card, the plan, and the design brief.** Copies smuggle in old structure, dead code, pinned incidentals, and compatibility residue — exactly what the intent-oracle rule frees you from. Applies to product code, tests, fixtures, and harnesses alike. The v2 tree follows the design brief's module map, never v1's layout; v1 stays open as intent reference only.

### Executed, not asserted

**An expected value nobody ran is an assertion, not an oracle.** Every expected value claiming v1 heritage is harvested at CAPTURE by executing v1 — fixtures or a live adapter — never transcribed from reading v1's code or guessed from its docs. Degraded capture (v1 cannot run and the invocation explicitly accepted that at SETUP) flags every such value `ASSERTED` in the table, and the report leads with that fact.

### Depth by tags

Plan depth is bought by evidence, not anxiety: T1 always; T2 always, depth scaling with `DEEP`; T3 only where `CONCURRENT` or `SCALE` earned it — a cold config reader does not get an endurance suite. Skipped tiers are recorded with their reason, never silently absent. The **UNPROVEN veto**: no race oracle on this machine → the concurrent posture ends `UNPROVEN` and BUILD takes the design brief's sequential fallback — never shipped on suspicion (test-practice's Concurrent correctness).

## State

Artifacts on disk; everything else lives in the conversation:

```
tdd-rewrite/             (scratchpad by default; in-repo if chosen at ASK)
├── design/              architecture sketches + the winning design brief
├── plans/               use-case cards, tiered test plans, capture fixtures
└── <target>-v2/         the rewrite — tests and code, nearest sibling to the original target
```

Working state is an in-dialog status table — one line per use case, design sketch, and feature: id, tags (`HOT/COLD`, `DEEP`, `CONCURRENT`, `SCALE`, `TRUST-BOUNDARY`, `SUSPECT`), per-tier status, amendment count, and — once VERIFY has run — the use case's **verification ledger**: which adversaries ran and claims-vs-actioned. Status vocabulary: use cases end `SERVED | DROPPED-BY-DESIGN` (via `CONFIRMED-GAP` when PARITY forces re-entry); tiers end `GREEN | SKIPPED-<reason> | UNPROVEN`; design sketches end `WON | LOST`; SUSPECT behaviors end `ADJUDICATED-DEFECT | ADJUDICATED-QUIRK`; features end `COVERED | SMALL-DIFF | DROPPED-BY-DESIGN`; capture values are `EXECUTED | ASSERTED`. Header: `Assumptions:` (every self-made call), target, language + toolkit (or derived sections), toolchain, oracle availability, **agent-economy level**, design-brief status, iteration counter.

Restate the current table compactly at every gate exit — never drop a line, whatever its status; the restatement is what carries the record through context compaction. Anything an agent needs (card, class map, plan, sketch, feature map, `DROPS:` list) is passed explicitly in its prompt; its verdict comes back in its reply and lands in the table.

## The gates

### SETUP

1. Determine target (whole repo or given scope). Ambiguity → take the reasonable reading, record under `Assumptions:`.
2. Verify **v1 builds and runs** — it is the oracle CAPTURE will harvest. v1's own tests failing is **evidence, not a stop** (record the failures; they feed TRIAGE's correctness lens — a red or missing v1 suite is often why the rewrite was ordered). v1 not building → stop and report — unless the invocation explicitly accepts a dead v1 ("v1 doesn't run, rewrite anyway"), which enters **degraded capture**: every expected value will be `ASSERTED`, flagged in the table and the report.
3. **Resolve the language toolkit on the target language** and **invoke the test-practice skill**; record both — or the derived toolkit sections — in the dialog.
4. Detect the toolchain per the toolkit's Toolchain section (compiler/runtime, threads, package manager).
5. **Confirm oracle availability on this machine** — especially the race oracle from the toolkit's Concurrency oracles. A missing oracle never blocks the run; it dooms specific claims per the UNPROVEN veto.
6. Install the test framework **library-first** (the toolkit's shortlist; ties go to what the project already uses), wire it, and run the harness self-check red-first: one deliberately failing test observed red, then removed.

**Exit:** v1 runs (or degraded capture recorded), toolkit + test-practice resolved, framework wired with the self-check seen red, setup summary stated (target, language/toolkit, toolchain, oracle availability).

### SCAN

Read the whole target — the **implementation study**. Record every **feature** at intent level (capability, why a user needs it, observable behavior, feature branches — modes, options, config-driven behaviors, documented edge semantics), every **flow** (how use cases chain: entry shape, transformations, side effects), and the **use-case map** — what callers/users accomplish: actor, entry point, intrinsic inputs, observable outcome, frequency. **Guarantees are features too:** ordering stability, thread-safety promises, documented tolerances, resource ceilings callers rely on — the easiest things to lose silently.

Harvest v1's existing tests as **intent evidence**: classes they encode, regressions they remember, tolerances they assert — candidate plan material, rewritten at contract level later, never copied. Inventory only — no judging, no plans.

**Exit:** every use case, feature, and flow in the status table; v1's test evidence recorded.

### CASE

Per use case, extract one **use-case card** — the single artifact every downstream adversary receives:

- Caller/user intent: what is accomplished, in one paragraph, and how often.
- Intrinsic inputs and the observable outcome owed.
- Invariants and guarantees callers rely on (ordering, tolerances, resource bounds, thread-safety promises).
- **Synchronization contract** where state is shared (per test-practice's Concurrent correctness).
- Trust boundaries on the path.
- Statable properties (permutation, round-trip, idempotence, invariant preservation).
- The **PINNED / FLEXIBLE / SUSPECT** split of every observable behavior, per the intent-oracle rule.

The card is contract, not implementation: it must not name v1's algorithms, data structures, or module layout — that is exactly the freedom the FREE rule protects. Every use case gets a card — nothing in a greenfield rewrite is exempt from a plan; tags set depth, not existence.

**Exit:** every use case has a card; every SUSPECT behavior adjudicated or queued for ASK.

### TRIAGE

Five lenses, one pass each:

1. **Correctness:** defect-risk scan with harden's four classes as the checklist — boundary, concurrency, lifetime/resource, garbage data. Emits `SUSPECT` tags on doubted behaviors (feeding CASE's adjudication) and owns the `TRUST-BOUNDARY` tag — every use case fed by external input (parsers, network data, file formats, user input). v1's own failing tests land here as evidence.
2. **Algorithmic depth:** how rich is the edge space — parsers, math kernels, state machines, protocol handlers are `DEEP`; plumbing and pass-through orchestration is shallow. `DEEP` scales T2 (bigger class map, more properties, denser differential) and gates the edge-hunter.
3. **Concurrency:** everything that shares mutable state in v1 *plus* everything a design sketch proposes to make concurrent in v2, judged against the toolkit's facilities — the `CONCURRENT` tag routes a use case into T3 and the Concurrent-correctness treatment.
4. **Scale:** where volume, endurance, or resource ceilings are contract — the 2 GB input, the 10k connections, flat memory over days. `SCALE` routes into T3 stress shapes.
5. **Hot/cold:** tag by frequency and centrality; hot use cases plan first, lead PROVE's cross-flow drive, and anchor the perf note.

**Exit:** every use case tagged on all five lenses.

### DESIGN

The **design study**, multi-candidate: draft **2–3 genuinely distinct greenfield architecture sketches** from the use-case map and tags, under no-compatibility rules — only the spirit of v1's capabilities must survive; signatures, module layout, error models, data structures are free. One page each: module map, public surfaces, error model, concurrency posture (with its sequential fallback), build order. Sketches must differ in structure, not in name — two sketches sharing a module map are one sketch. A sketch that deliberately narrows or drops a capability declares `DROPS: <feature> — <rationale>`.

One blind **comparative design-refuter** (an agent at every level above `inline`; at `inline`, main-thread paper comparison recorded `NOT-BLIND`) receives the sketches, the use-case map, the tags, and the machine's oracle availability — never the drafting reasoning. It judges all sketches together on one scale — may rank, may not approve two contradictory ones — with the kill-checklist:

- **Testability first:** can every public surface be driven red-first in isolation — no hidden globals, no construct-the-world setup, seams where the tier ladder needs them?
- Floor coverage: does some surface serve every use case's intrinsic inputs and outcome?
- Dependency direction: acyclic, foundation buildable and testable before what stands on it — is the build order real?
- Concurrency posture: implementable with an oracle available on this machine? No oracle → refuted toward the sequential fallback.
- Error model: failures observable at the surfaces (testable), consistent across modules, never swallowed.

The surviving sketch — amended by any fixes the refuter demanded — becomes the **design brief**; losers end `LOST` with one-line reasons.

**Exit:** one sketch `WON` and standing as the design brief; every verdict recorded; ready for ASK.

### ASK — the only user stop

Present in one batch: assumptions taken, use-case inventory with tags, **SUSPECT adjudications** (defect vs quirk — with the recommended verdict each), the **design brief** (the winner plus one-line why per loser), the **agent-economy level** (when the invocation didn't set it), planned `DROPS:`, artifact location, and every genuine either-way call. One question round; answers are recorded. In autonomous mode: skip, self-decide, record.

**Exit:** every ASK-class decision recorded; design brief confirmed; every SUSPECT behavior `ADJUDICATED-DEFECT` or `ADJUDICATED-QUIRK`. From here to REPORT, zero questions.

### PLAN (pipeline entry — re-entered by PARITY, card first)

Per use case, from the card — never from v1's code: build the **class map** (equivalence partitioning + BVA per test-practice's TDD), then author the **tiered test plan** per its section above, depth per tags. Every item carries its id, tier, covered class/property/behavior, kind, oracle, and expected-value source. v1's harvested test evidence folds in as candidate classes and regression items — rewritten at contract level. A plan that deliberately narrows a capability declares `DROPS:` like a design sketch.

Check the plan against its **coverage claims** — every class exactly once, every stated property an item, every PINNED behavior a differential item, every synchronization contract T3 items, every trust boundary boundary items — before calling it complete.

`CONFIRMED-GAP` features re-enter here: a fresh or amended use-case card is written first (CASE's format, TRIAGE's tags), then the plan proceeds as usual — same rules, same adversaries, indistinguishable from first-iteration work.

**Exit:** the plan meets its coverage claims; every item names an oracle and an expected-value source.

### REFUTE (independent adversaries, dispatched per the Agent economy)

One blind **plan-refuter** per use case, given only the card, the class map, the plan, and the tags; it judges all tiers together on one coverage scale. The kill-checklist:

- Class-map holes: missed equivalence classes; boundaries without their BVA triple (at / just below / just above).
- T1 honesty: are the main flows end-to-end through public surfaces with realistic data — or toys?
- Properties: stated invariants with no item; invented properties the contract never states (both are defects).
- Differential discipline: PINNED without an exact item; FLEXIBLE compared exact (over-pinning — it freezes the design); SUSPECT in the plan without its adjudication.
- T3: `CONCURRENT` without race-oracle, interleaving, and contended-stress items; `SCALE` without its volume/endurance shapes; work parked in the wrong tier.
- Item quality: untestable items (no observable outcome), tautological items (assert whatever the code does), items testing the framework or a library instead of the contract.

Whoever refutes — agent or main thread — also **nominates the nastiest missed inputs and schedules** (hash-collision sets, pathological orderings, quadratic-blowup shapes, malformed-at-the-boundary data, hostile interleavings). The main thread adjudicates nominations like hunter claims: in-contract → new plan item; out-of-contract → rejected with a one-line reason. Verdict rule — **doubt adds the item**: an uncertain class costs one test; a missed class costs a defect. Duplicates proven at BUILD are merged and recorded, restoring the 1:1 rule.

**Exit:** every checklist dimension addressed, every nomination dispositioned, the amended plan standing.

### CAPTURE

Make the plan's expected values real — **the last gate at which v1 must still run**:

1. Execute every differential item against v1 through a contract-level adapter (cross-language is fine); harvest expected values as fixtures, or wire v1 as a live oracle for BUILD.
2. Sweep property generators against v1 where cheap — a property v1 itself violates is either a wrong property or a SUSPECT behavior.
3. **v1 failing its own PINNED item** means the item is wrong (fix the plan) or the behavior is SUSPECT (adjudicate per the ASK-recorded rules; autonomous → self-adjudicate, record). Never papered over, never silently re-pinned to whatever v1 does.
4. Every value lands `EXECUTED`; in degraded capture (dead v1, explicitly accepted at SETUP) values land `ASSERTED` and the report leads with that fact.

**Exit:** every differential item has an executed expected value (or its `ASSERTED` flag); every capture-time adjudication recorded.

### BUILD

Climb the **tier ladder** in `<target>-v2/` — fresh files per the design brief and the no-copy rule:

1. **T1 red:** main-flow tests written failing against the empty v2 surface. Implement to green — the first use case that needs a module drives that module into existence; later use cases extend it red-first (shared modules grow monotonically; no edit without a red item demanding it).
2. **T2 red → green:** class tests 1:1, property tests, differential items green against the captured expectations. An `ADJUDICATED-DEFECT` dies here, by design, and its fix is recorded.
3. **T3 red → green** where owed: synchronization-contract tests and bounded contended stress under the race oracle, deterministic interleaving forcing at suspected windows, `SCALE` volume/endurance shapes. No oracle → `UNPROVEN`, take the brief's sequential fallback, record it.
4. Plan amendments per the Red-first rule — item first, code second, count recorded.

**Exit:** every planned tier `GREEN` (or `SKIPPED-<reason>` / `UNPROVEN` recorded); amendment count in the table.

### VERIFY — adversarial proof of correctness

Per built use case, depth keyed by its tags:

1. **Edge-hunter — risk-gated:** `DEEP`, `HOT`, `TRUST-BOUNDARY`, or `SUSPECT` use cases get one blind edge-hunter (test-practice Adversarial), receiving the card, class map, plan, and code only. An untagged use case skips it — its plan already survived a blind refuter and its differential faced v1 — and the skip is recorded in the ledger. (`per-claim`: every built use case gets the hunter; `inline`: main-thread self-audit, recorded `NOT-BLIND`.) Accepted claims become new plan items fixed red-first per the fix protocol; rejections get a one-line reason.
2. **`TRUST-BOUNDARY` — boundary enforcement** (test-practice Boundary enforcement): standard mechanisms enabled per the toolkit; hand-written validation gets its red/green pair. Findings block the use case until fixed red-first.
3. **`CONCURRENT` — race oracle + interleaving-hunter:** contended stress under the race oracle, then one blind interleaving-hunter receiving the synchronization contract and code. (`minimal`: the edge and interleaving mandates merge into one dual-mandate adversary — except a use case both `HOT` and `CONCURRENT` keeps them split; `inline`: self-audit, recorded `NOT-BLIND`.) No race oracle on this machine → `UNPROVEN`, revert to the sequential fallback — never shipped on suspicion.
4. **All — hardening pass:** run the toolkit's Hardening-oracles builds (address/UB sanitizers and equivalents) over the use case's suite; any finding is a defect, fixed per the fix protocol.

**Exit:** every hunter claim triaged, every boundary enforced, oracles silent (or `UNPROVEN` recorded and reverted), ledger recorded.

### PROVE

Whole-v2, in situ:

1. Full suite green across every use case.
2. **Cross-use-case flows** from SCAN driven end-to-end — the flow map is the checklist; a use case green in isolation but broken in its flow is broken.
3. One whole-system contended run if anything is `CONCURRENT`; one endurance/volume sweep if anything is `SCALE`.
4. **Final differential sweep** against v1 at contract level — test-practice's green-before-deletion rule made real: after this gate nothing further needs v1 to run except a re-entered gap's CAPTURE.
5. **Perf note:** crude wall-clock of the hot main flows, v1 vs v2, same data, a few repetitions with the spread noted. Grossly slower → `PERF-NOTE` warning in the table naming perf-tune / algo-rewrite as the follow-up campaign. Never a gate, never optimized against here.

**Exit:** suite and flows green, differential sweep green, contended/endurance runs done where owed, perf note recorded; v1 retirable (deleted only after PARITY exits clean).

### PARITY (one gap-hunter pass — an agent at every level above `inline` — can force a new pipeline iteration)

The gap-hunter (one blind agent; at `inline`, the main thread, recorded `NOT-BLIND`) receives v1, v2, the feature map, the use-case map, the `DROPS:` list, and the SUSPECT adjudication record — not the rewrite thread's reasoning. Mandate: find features present in v1 whose **intent** is absent in v2, and **triage severity itself**:

- **Design-defining** — a capability, mode, or guarantee users build around; losing it forces callers to hand-roll logic or restructure usage. Only these come back as `PARITY-CLAIM`s.
- **Small** — convenience overloads, incidental output details, precision nuances, one-line-recoverable behaviors. Returned as `SMALL-DIFF` notes: recorded, never actioned. Unsure → must argue "usage would restructure without it"; can't → small.

The **spirit rule** for judging: changed signatures, error models, internals, module layout, drift within adjudicated tolerance = fine. Missing capability, silently narrowed input domain, dropped mode, lost guarantee = gap. **A fixed `ADJUDICATED-DEFECT` is not a gap** — it matches the adjudication record.

Main thread adjudicates each claim: **refute with a trace** (the plan item and green test that serve the intent) / **DROPPED-BY-DESIGN** (matches a declared drop) / **CONFIRMED-GAP**. An undeclared drop is always a confirmed gap — even if dropping was right, the decision must become explicit, never accidental.

**Confirmed gaps get the full pipeline, not band-aids:** each re-enters at PLAN — card first, then plan, refuter, capture against the still-alive v1, tier ladder, full verification spine. The result is indistinguishable from first-iteration work: same rules, same gates, same evidence.

**Exit:** a full gap-hunter pass (a fresh adversary) returns **zero new design-defining claims**. Cost per pass: exactly 1 agent call at every level above `inline` — the gap-hunter is never batched, merged, or risk-gated away.

### REPORT

From the status table, cumulative:

- **Served** — per use case: the tier ledger (items per tier, red→green, `SKIPPED`/`UNPROVEN` with reasons) and the amendment count — high counts called out as PLAN/REFUTE lessons.
- **Coverage** — the class→item map, properties, differential density; capture provenance (`EXECUTED` vs `ASSERTED` counts).
- **Design** — the winning sketch, one line per loser, and every recorded brief change.
- **SUSPECT adjudications** — every defect fixed-by-design and every quirk deliberately preserved.
- **Dropped features** — every `DROPPED-BY-DESIGN` with its rationale; gaps confirmed, re-entered, and covered.
- **Verification ledger** — per use case: which adversaries ran (and at what agent-economy level), claims vs actioned, every `NOT-BLIND` judgment, anything left `UNPROVEN` and what was done about it.
- **Perf note** — the crude v1-vs-v2 numbers with their spread; the pointer to perf-tune / algo-rewrite if warranted.
- Exact commands to re-run the suite and re-capture the fixtures.

Never dress up an interrupted run as complete; report the actual last status table as-is — that is the record of where the run stopped.

**Exit:** the report delivered from the final status table, nothing omitted — every use case, sketch, feature, and adjudication accounted for.
