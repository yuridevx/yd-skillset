---
name: test-practice
description: Language-agnostic testing rulebook — the testing half of algo-rewrite and harden. Three named sections. TDD — red-first protocol: no code before its failing oracle, the red run's actual output captured as evidence, class map via equivalence partitioning + boundary-value analysis, main-flow-first minimal 1:1 test set, failing-repro fix protocol. Fuzzing — coverage-guided oracle-fuzzing standard: seed corpus from class-map inputs, dictionary for grammar inputs, invariant assertions in the harness (never crash-only), progress gate, corpus kept as a permanent regression artifact. Adversarial — blind edge-hunter agent protocol: agent sees only contract + class map + code, hunts BVA gaps and adversarial shapes, caller triages claims into new red tests or one-line rejections. Engines, sanitizers, and harness installs come from the <lang>-toolkit skill (e.g. msvcpp-toolkit) or are derived at setup when none exists. PASSIVE reference — invoked by algo-rewrite and harden; also usable directly when defining the testing standard for a task. Trigger on "test-practice", "/test-practice", or as the testing rulebook for algo-rewrite/harden.
---

# test-practice

The testing rulebook. Division of labor with its consumers:

```
caller skill (algo-rewrite, harden, …)  →  WHEN   — gates, ledger, loop control
test-practice                           →  HOW    — the testing method (this file)
<lang>-toolkit (e.g. msvcpp-toolkit)    →  WITH WHAT — engines, sanitizers, installs
```

This skill owns no gates, no ledger, no loop. Callers cite its sections by name (**TDD**, **Fuzzing**, **Adversarial**) and record all outcomes in their own state. Language specifics come from the caller's resolved `<lang>-toolkit` (its **Fuzzing** and **Hardening oracles** sections); no toolkit → the caller derives them at setup, and this rulebook applies unchanged.

## TDD

**No code exists before its failing oracle does.** Every unit of work opens red and closes green.

- **Red leaves evidence, not claims.** The red run's actual failure output is captured into the caller's trail/ledger. A test whose red run was never captured does not count as having predated the code.
- **Class map:** enumerate representable input classes from the public contract — empty, one element, typical, max-size, malformed-only-if-reachable. Contract-impossible inputs are out of scope: no breaking things under unrealistic expectations.
- **Boundary-value analysis:** every numeric, size, and capacity boundary in the contract yields three classes — at the boundary, just below, just above. Errors cluster at edges; a class map without BVA classes is incomplete.
- **Main flow first:** the first tests written drive the component end-to-end through its public API with realistic data. Edge-class tests come after — an edge-perfect component whose main flow was never exercised is untested.
- **Minimal 1:1 set:** exactly one test per class plus the main flows — the smallest set where every class appears once. Redundant tests are refused like redundant code.
- **Fix protocol** (every fix, regardless of source — review finding, fuzz finding, adversarial gap, regression): failing repro test first → minimal fix to green → test kept permanently as a regression guard.

## Fuzzing

Coverage-guided oracle-fuzzing, never crash-only. Engine, sanitizers, and platform notes per the toolkit's **Fuzzing** section.

1. **Seed corpus:** seed every harness with the class-map test inputs plus realistic samples. Unseeded fuzzing wastes its budget rediscovering structure.
2. **Dictionary:** token/grammar/format inputs get a dictionary of their keywords, delimiters, and magic bytes.
3. **Oracle in the harness:** assert the component's own invariants and postconditions (output sorted *and* a permutation of input; parse→serialize roundtrips; documented bounds hold) — a run that only catches crashes misses broken logic entirely. Sanitizers on for the whole run.
4. **Structure-aware at trust boundaries, contract-valid inputs only:** bytes into parsers, arrays into sorters — never garbage into internal APIs.
5. **Progress gate:** a run counts as clean only if the target neither hung nor OOM'd immediately *and* coverage grew past the seed corpus. A plateau at the seeds is a broken harness — fix the harness and rerun; it is not a clean bill.
6. **Corpus is a kept artifact** in the caller's dossier: every finding's input joins it permanently, making the corpus a regression suite. Findings block per the TDD fix protocol.
7. **Timebox:** 60s/component default, longer for parsers and decoders; the caller may override.

## Adversarial

Blind edge-hunter protocol — one independent agent per component, after its tests are green.

- **Blindness:** the agent receives only the public contract, the class map, and the code under test — never the author's reasoning, so it cannot inherit the author's blind spots.
- **Mandate:** find input classes the map missed — BVA gaps, empty/max/duplicate shapes, and adversarial shapes: hash-collision sets, pathological orderings, quadratic-blowup inputs, malformed-at-the-boundary data.
- **Claim bar:** a claim is a concrete input plus the expected-vs-plausible-actual outcome. "This looks untested" is not a claim.
- **Caller triage**, per claim: in-contract and genuinely missed → new class, red test, fix per the TDD fix protocol; out-of-contract → rejected with a one-line reason.
- **Receipt** in the caller's ledger: `Edge-audit: N claims → A accepted / R rejected`. Cost: exactly 1 agent per component per pass.
