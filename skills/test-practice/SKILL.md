---
name: test-practice
description: Language-agnostic testing rulebook — the testing half of algo-rewrite and harden. Three named sections. TDD — red-first protocol: no code before its failing oracle, the red run's actual output captured as evidence, class map via equivalence partitioning + boundary-value analysis, main-flow-first minimal 1:1 test set, failing-repro fix protocol. Boundary enforcement — deterministic standard replacing fuzzing: every BVA-derived boundary gets an explicit red test plus a runtime enforcement mechanism (assertion, contract, bounds-checked type, sanitizer-backed build) added at the boundary itself, proven to fire before it ships. Adversarial — blind edge-hunter agent protocol: agent sees only contract + class map + code, hunts BVA gaps and adversarial shapes, caller triages claims into new red tests or one-line rejections. Sanitizers, checked types, and enforcement mechanisms come from the <lang>-toolkit skill (e.g. msvcpp-toolkit) or are derived at setup when none exists. PASSIVE reference — invoked by algo-rewrite and harden; also usable directly when defining the testing standard for a task. Trigger on "test-practice", "/test-practice", or as the testing rulebook for algo-rewrite/harden.
---

# test-practice

The testing rulebook. Division of labor with its consumers:

```
caller skill (algo-rewrite, harden, …)  →  WHEN   — gates, ledger, loop control
test-practice                           →  HOW    — the testing method (this file)
<lang>-toolkit (e.g. msvcpp-toolkit)    →  WITH WHAT — engines, sanitizers, installs
```

This skill owns no gates, no ledger, no loop. Callers cite its sections by name (**TDD**, **Boundary enforcement**, **Adversarial**) and record all outcomes in their own state. Language specifics come from the caller's resolved `<lang>-toolkit` (its **Boundary enforcement** and **Hardening oracles** sections); no toolkit → the caller derives them at setup, and this rulebook applies unchanged.

## TDD

**No code exists before its failing oracle does.** Every unit of work opens red and closes green.

- **Red leaves evidence, not claims.** The red run's actual failure output is captured into the caller's trail/ledger. A test whose red run was never captured does not count as having predated the code.
- **Class map:** enumerate representable input classes from the public contract — empty, one element, typical, max-size, malformed-only-if-reachable. Contract-impossible inputs are out of scope: no breaking things under unrealistic expectations.
- **Boundary-value analysis:** every numeric, size, and capacity boundary in the contract yields three classes — at the boundary, just below, just above. Errors cluster at edges; a class map without BVA classes is incomplete.
- **Main flow first:** the first tests written drive the component end-to-end through its public API with realistic data. Edge-class tests come after — an edge-perfect component whose main flow was never exercised is untested.
- **Minimal 1:1 set:** exactly one test per class plus the main flows — the smallest set where every class appears once. Redundant tests are refused like redundant code.
- **Fix protocol** (every fix, regardless of source — review finding, boundary-enforcement finding, adversarial gap, regression): failing repro test first → minimal fix to green → test kept permanently as a regression guard.

## Boundary enforcement

Deterministic standard, replacing coverage-guided fuzzing: every boundary the class map's BVA identified gets enumerated and enforced, not searched for probabilistically. Mechanisms, sanitizers, and platform notes per the toolkit's **Boundary enforcement** section.

1. **Enumerate from the class map:** every BVA boundary (numeric, size, capacity) already named by TDD is the complete worklist — nothing left to a corpus or a random walk to discover.
2. **Red proves the gap:** for each boundary, a failing test first shows the violation goes uncaught (OOB read/write, off-by-one, overflow feeding an allocation or index).
3. **Enforce at the boundary itself:** add the mechanism the toolkit recommends for this language — a bounds-checked type/view, an assertion or contract on the documented precondition, a sanitizer-backed build — so the violation is caught structurally, not just by the one test that exercises it.
4. **Trust boundaries get every field checked:** parser input, network data, file formats — validate length/count/size fields against actual capacity before use; contract-valid inputs only, never garbage into internal APIs.
5. **Green proves enforcement fires:** rerun the same test — it now passes because the enforcement caught the violation (assertion tripped, sanitizer flagged, bounds check rejected), not because the underlying bug was hand-patched around. Test stays as a permanent regression guard.
6. **No enforcement without its test:** an assertion or bounds check added without a red/green pair proving it fires is unverified — treat it as absent.

## Adversarial

Blind edge-hunter protocol — one independent agent per component, after its tests are green.

- **Blindness:** the agent receives only the public contract, the class map, and the code under test — never the author's reasoning, so it cannot inherit the author's blind spots.
- **Mandate:** find input classes the map missed — BVA gaps, empty/max/duplicate shapes, and adversarial shapes: hash-collision sets, pathological orderings, quadratic-blowup inputs, malformed-at-the-boundary data.
- **Claim bar:** a claim is a concrete input plus the expected-vs-plausible-actual outcome. "This looks untested" is not a claim.
- **Caller triage**, per claim: in-contract and genuinely missed → new class, red test, fix per the TDD fix protocol; out-of-contract → rejected with a one-line reason.
- **Receipt** in the caller's ledger: `Edge-audit: N claims → A accepted / R rejected`. Cost: exactly 1 agent per component per pass.
