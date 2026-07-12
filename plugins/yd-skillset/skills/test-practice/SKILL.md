---
name: test-practice
description: Language-agnostic testing rulebook — the testing half of algo-rewrite, tdd-rewrite, algo-craft, perf-tune, and harden. Five named sections. TDD — red-first protocol — watch the test fail before writing the code that greens it, class map via equivalence partitioning + boundary-value analysis, main-flow-first minimal 1:1 test set, failing-repro fix protocol. Properties & differential — property tests wherever the contract states an invariant; when a reference implementation exists it is the intent oracle — a differential suite over bounded deterministic generators compares at contract level (exact only where the contract pins the output, equivalence predicate where it permits variation, suspected-defect behaviors adjudicated before they may be pinned) and must be green before the reference is deleted. Boundary enforcement — deterministic standard replacing fuzzing — standard mechanisms (bounds-checked types, STL hardening, sanitizer builds) are enabled per the toolkit; hand-written validation at trust boundaries gets a red test proving the gap before the check lands. Concurrent correctness — shared-state components declare a synchronization contract; the race oracle's report is the red test; bounded contended stress plus deterministic interleaving forcing; no oracle available means UNPROVEN, never adopted on suspicion. Adversarial — blind hunter protocol with a shared calibration standard (claim bar, verbatim false-positive list, confidence labels) — edge-hunters attack input classes, interleaving-hunters attack schedules. Sanitizers, checked types, race oracles, and enforcement mechanisms come from the <lang>-toolkit skill (e.g. msvcpp-toolkit) or are derived at setup when none exists. PASSIVE reference — invoked by algo-rewrite, tdd-rewrite, algo-craft, perf-tune, and harden; also usable directly when defining the testing standard for a task. Trigger on "test-practice", "/test-practice", or as the testing rulebook for algo-rewrite/tdd-rewrite/algo-craft/perf-tune/harden.
---

# test-practice

The testing rulebook. Division of labor with its consumers:

```
caller skill (algo-rewrite, tdd-rewrite, algo-craft, perf-tune, harden, …)  →  WHEN   — gates, state, loop control
test-practice                                       →  HOW    — the testing method (this file)
<lang>-toolkit (e.g. msvcpp-toolkit)                →  WITH WHAT — engines, sanitizers, oracles, installs
```

This skill owns no gates, no state, no loop. Callers cite its sections by name (**TDD**, **Properties & differential**, **Boundary enforcement**, **Concurrent correctness**, **Adversarial**) and record all outcomes in their own state. Language specifics come from the caller's resolved `<lang>-toolkit` (its **Boundary enforcement**, **Concurrency oracles**, and **Hardening oracles** sections); no toolkit → the caller derives them at setup, and this rulebook applies unchanged.

## TDD

**No code exists before its failing oracle does.** Every unit of work opens red and closes green.

- **Run it red first.** Watch the test fail before writing the code (or fix) that greens it. Seeing it fail is the point — no transcription or evidence-logging requirement.
- **Class map:** enumerate representable input classes from the public contract — empty, one element, typical, max-size, malformed-only-if-reachable. Contract-impossible inputs are out of scope: no breaking things under unrealistic expectations.
- **Boundary-value analysis:** every numeric, size, and capacity boundary in the contract yields three classes — at the boundary, just below, just above. Errors cluster at edges; a class map without BVA classes is incomplete.
- **Main flow first:** the first tests written drive the component end-to-end through its public API with realistic data. Edge-class tests come after — an edge-perfect component whose main flow was never exercised is untested.
- **Minimal 1:1 set:** exactly one test per class plus the main flows — the smallest set where every class appears once. Redundant tests are refused like redundant code.
- **Fix protocol** (every fix, regardless of source — review finding, boundary-enforcement finding, adversarial gap, regression): failing repro test first → minimal fix to green → test kept permanently as a regression guard.

## Properties & differential

The algorithm-strength supplement to the 1:1 class set — properties and reference comparison catch what enumerated classes miss. Neither replaces the class map.

- **Property tests:** wherever the public contract states an invariant that holds across the input domain — permutation (output is a reordering of the input), round-trip (decode(encode(x)) = x), idempotence, monotonicity, invariant preservation — write one property test over a **bounded, seeded, deterministic generator** covering the class map's shapes plus a size sweep. No statable property → skip; never invent one to have one. This is not coverage-guided fuzzing — generators are enumerable and reproducible.
- **The reference is the intent oracle.** When a reference implementation exists (the current code in an improve/refactor job, v1 in a rewrite), it witnesses the **contract's spirit — capabilities, invariants, guarantees, tolerances — never the algorithm**. The differential suite compares through the contract:
  - **PINNED** behavior — the contract fixes the exact output → compare exact.
  - **FLEXIBLE** behavior — the contract fixes a predicate only (±tolerance, any valid ordering, error model free to change) → compare by equivalence predicate or property assertion. Pinning an incidental behavior is a defect: it freezes the algorithm.
  - **SUSPECT** behavior — the caller's correctness lens doubts the reference here → **adjudicate before pinning** (defect vs load-bearing quirk callers rely on). A defect never enters the harness as expected behavior — bug-for-bug compatibility is refused; the new code asserts the adjudicated behavior and the adjudication is recorded in the caller's state.
- **Mandatory before deletion:** the reference may not be deleted or replaced until the differential suite is green against it — capture the oracle while it exists.

## Boundary enforcement

Deterministic standard, replacing coverage-guided fuzzing: every boundary the class map's BVA identified gets enumerated and enforced, not searched for probabilistically. Mechanisms, sanitizers, and platform notes per the toolkit's **Boundary enforcement** section.

1. **Enumerate from the class map:** every BVA boundary (numeric, size, capacity) already named by TDD is the complete worklist — nothing left to a corpus or a random walk to discover.
2. **Standard mechanisms are enabled, not tested:** bounds-checked types/views, STL hardening flags, assertion-backed preconditions from the toolkit's list, sanitizer-backed builds — turning them on is the enforcement. Testing that a library's bounds check fires is testing the library; skip it.
3. **Hand-written validation gets the red/green pair:** checks you write yourself — length/count/size fields validated at trust boundaries (parser input, network data, file formats) — get one failing test showing the violation went uncaught, then the check, then green. The test stays as a permanent regression guard.
4. **Trust boundaries get every field checked:** validate every external length/count/size field against actual capacity before use; contract-valid inputs only, never garbage into internal APIs.

## Concurrent correctness

Applies to any component that shares mutable state, alters synchronization, or claims lock-/wait-freedom — including code a caller's lens proposes to *make* concurrent. Oracles and facilities per the toolkit's **Concurrency oracles** section.

1. **Synchronization contract first:** before implementation, state what is shared, who owns it, what ordering/visibility assumptions hold, and what may block. It is part of the component's contract/class map, and it is what the interleaving-hunter receives.
2. **The race oracle's report is the red test.** A race-detector finding *is* the reproduction: force a deterministic interleaving (barriers/latches at the suspected window) or fall back to a bounded stress loop under the detector, watch it fire, fix, watch it go silent. "Looks racy" with the detector silent is unreproduced.
3. **Bounded contended stress:** every concurrent component gets a multi-thread stress test asserting its invariants, run under the race oracle. A concurrent structure exercised only single-threaded is untested; its performance measured only uncontended is unmeasured (the caller's benchmark protocol owns the contended numbers).
4. **No oracle → UNPROVEN.** Where the toolkit says no race oracle runs on this platform, concurrent-correctness claims end `UNPROVEN` in the caller's record — never fixed, adopted, or shipped on suspicion. The caller decides the fallback (usually the simple/sequential design).

## Adversarial

Blind hunter protocol — independent agents that attack finished work, after its tests are green.

- **Blindness:** a hunter receives only the public contract (contract card where the caller has one), the class map / synchronization contract, and the code under test — never the author's reasoning, so it cannot inherit the author's blind spots.
- **Two hunter types:**
  - **Edge-hunter** — attacks the input space: BVA gaps, empty/max/duplicate shapes, and adversarial shapes (hash-collision sets, pathological orderings, quadratic-blowup inputs, malformed-at-the-boundary data).
  - **Interleaving-hunter** — attacks the schedule space of concurrent components: publication races, ABA, torn reads/writes, missing fences, lock-order inversions, assumptions the synchronization contract doesn't actually guarantee.
- **Calibration standard** (applies to every blind agent a caller dispatches — hunters, refuters, triage agents alike):
  - **Claim bar:** a claim is a concrete input or interleaving plus the expected-vs-plausible-actual outcome. "This looks untested" or "this might race" is not a claim.
  - **Verbatim false-positive list in the prompt:** out-of-contract inputs, pre-existing behavior outside the change, below-noise performance remarks, style/naming/modernization — none of these are claims.
  - **Confidence labels:** the agent labels each claim `confirmed` (failure path traced end-to-end or oracle-backed) or `plausible` (credible but untraced). Callers action `confirmed` claims via the TDD fix protocol; `plausible` claims are recorded without action unless corroborated.
- **Caller triage**, per claim: in-contract and genuinely missed → new class, red test, fix per the TDD fix protocol; out-of-contract → rejected with a one-line reason.
- **Cost:** exactly 1 agent per component per hunter type per pass; callers running a reduced agent-dispatch level may merge the two mandates into one agent, or substitute a main-thread self-audit recorded as not blind. The outcome is one line in the caller's record.
