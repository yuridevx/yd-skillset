---
name: harden
description: Iterative, low-noise hardening review that only fixes proven defects — boundary violations, concurrency bugs (data races/deadlocks), resource/lifetime errors (leaks, use-after-free, double-release, dangling references), and garbage/uninitialized data. Language-agnostic core; red-first testing discipline loads from the test-practice skill; language specifics (defect-class scope, oracle/sanitizer table, platform caveats) load from a <lang>-toolkit skill (e.g. msvcpp-toolkit) or are derived at setup when none exists. Works TDD-style, gather candidates, adversarially refute them, reproduce each survivor with a failing oracle-backed test BEFORE any edit, fix minimally, and loop until a full pass finds nothing new. Linear, no subagents. Style, performance, naming, and modernization are explicitly out of scope. Trigger on "harden", "/harden", "cpp-harden", "/cpp-harden", "harden this code", "harden this C++", "make this code safe", "check boundaries and concurrency".
---

# harden

Iterative TDD hardening loop. Core principle: **a defect does not exist until a failing test says it does.** Every phase is a filter; code edits happen only at the very end of each iteration, only on what survived every filter. You run this **linearly — no subagents, no fan-out.** All state lives in the conversation.

## Defect contract — the only four classes in scope

1. **Boundary** — OOB read/write, off-by-one, unchecked size/index, integer overflow feeding an allocation or index.
2. **Concurrency** — data race, deadlock, torn read/write, missing synchronization on shared state, async/await misuse of shared state.
3. **Lifetime/resource** — leak, use-after-free, double-free/double-release, dangling reference/iterator, unreleased native handles.
4. **Garbage data** — uninitialized reads, type-punning UB, partially-constructed/partially-initialized objects observed.

The **toolkit refines what each class means in the target language** — e.g. under a GC, use-after-free is largely out of scope but undisposed handles, event-subscription leaks, and torn initialization are in. Only scenarios the toolkit marks reachable in this language count as candidates.

Style, naming, performance, architecture, "modernize", refactoring opportunities: **out of scope, say nothing about them.** This silence is the contract that makes the output low-noise.

## State: the in-dialog candidate record

Track every candidate as one line in the conversation:

```
C<seq> <file>:<line> [<class>] <STATE> — <scenario, one clause> — <refutation reason | test name | why unreproduced>
```

States: `CANDIDATE → REFUTED | SURVIVED → PROVEN | UNREPRODUCED | UNPROVEN → FIXED`.
Restate the full candidate table at the end of every phase — never drop a line, whatever its state; the scenario clause is what dedup checks against, and the restatement is what carries the record through context compaction.

## Phase 0 — Setup gate (once)

1. Determine target: the diff, user-given paths, or whole repo (ask only if genuinely ambiguous).
2. **Resolve the language toolkit.** Identify the target language and invoke the matching `<lang>-toolkit` skill (e.g. `msvcpp-toolkit`); its **Hardening oracles** section supplies the defect-class refinements, the per-class oracle table for Phase 3, and platform caveats. No toolkit skill for this language → derive the same content yourself (which defect scenarios are reachable in this language, available sanitizers / runtime checkers / analyzers, per-class oracle, platform gaps) and state it explicitly in the dialog. Also invoke the **test-practice** skill — its TDD section governs the red-first discipline of Phases 3–4 (minimal failing test, fix protocol).
3. Verify the project **builds and existing tests pass**. Broken baseline → stop and report; you cannot do red/green on red.
4. Confirm which oracles from the toolkit table actually work on this machine, plus the test framework and how to add and run one test. A defect class whose oracle is unavailable here still gets hunted — but its survivors end as `UNPROVEN`, never fixed on suspicion.
5. State the setup summary in the dialog: target, language, toolchain, oracle availability, iteration counter. It heads every candidate-table restatement from here on.

## Iteration loop — Phases 1–4, repeat until converged

### Phase 1 — Hunt

Read the target code once with the four defect classes as your checklist; on files dense with shared state or pointer arithmetic, a second pass through the riskiest lens is worth it — but four mandated full reads are not.

Entry bar: a candidate must have a **concrete failure scenario** — specific input/state/interleaving → specific wrong outcome. "This index isn't obviously checked" is not a candidate. "`parse()` passes `len-1` from a network header to a raw copy without comparing against the buffer size; a header claiming len=0 wraps to the type's max" is.

Dedup against **every candidate dispositioned so far, including REFUTED entries** — same location (± line drift) and same scenario means already dispositioned; skip it. Only genuinely new scenarios enter as `CANDIDATE`. No judging plausibility, no fixing — just record.

Scope per iteration: iteration 1 reads the whole target. Iteration N>1 reads (a) every file a Phase 4 fix touched plus its direct callers, and (b) any code a prior refutation depended on that has since changed — if "caller validates the index" was the refutation and the caller changed, reopen that entry to `CANDIDATE`.

### Phase 2 — Refute

For each `CANDIDATE`, actively try to **kill it**. Re-read with full context: callers, callees, class invariants, threading model. Standard refutations per class:

- Boundary: validated upstream? type range-limited? buffer provably large enough by construction? language-level bounds check turns the scenario into a safe, expected failure rather than corruption?
- Concurrency: actually reached by >1 thread? lock held by caller? confined to pre-thread init? atomic via a typedef/annotation you missed?
- Lifetime/resource: does ownership actually escape? is the "leak" a deliberate singleton? does the runtime (GC, RAII, using/defer) already release it?
- Garbage: zero/default-initialized by the language or a construction path you didn't trace?

The toolkit's refinements apply here too — a scenario the language rules out by construction is `REFUTED (impossible in <lang>)`.

**Verdict rule: to survive, the full failure path must be traced end-to-end. Uncertainty → `REFUTED (unproven path)` with a one-line reason.** This bias is what kills the noise. Otherwise `SURVIVED`.

### Phase 3 — Reproduce (TDD red)

For each `SURVIVED`, write **one minimal failing test**. The oracle comes from the toolkit's per-class oracle table — a sanitizer, runtime checker, leak detector, or assertion on the corrupted value; never "I'm pretty sure".

Run it. Three outcomes:

- Fails as predicted → `PROVEN`. Keep the test.
- Doesn't fail → `UNREPRODUCED`. Delete the test, **no edit**, record why the repro failed.
- No oracle possible on this platform → `UNPROVEN`, no edit. Honest "believed but unproven" beats a speculative fix.

Concurrency rule: a race-detector report **is** the reproduction — force a deterministic interleaving (barriers/latches) or fall back to a bounded stress loop under the detector. "Looks racy" with the detector silent is `UNREPRODUCED`; no race detector on this platform means concurrency survivors end `UNPROVEN`.

### Phase 4 — Fix (TDD green)

Only `PROVEN` entries. For each:

1. Minimal fix — smallest diff that makes the failing test pass. No drive-by refactoring. Anything new spotted mid-fix becomes a fresh `CANDIDATE` in the record, never fixed inline.
2. Rerun the repro test → passes.
3. Rerun the full suite (existing tests + every repro test accumulated across all iterations) → green. Repro tests from earlier iterations are regression guards for later ones.
4. Record → `FIXED`.

The suite must be fully green before the next iteration starts.

### Convergence — the only exit

**Converged = a full Phase 1–3 pass with zero new candidates AND zero state transitions** (nothing reopened, nothing changed state). A pass that fixed anything is never the converging pass — Phase 4's touched-files rescan (next iteration's Phase 1 scope) must come back clean first. A clean iteration 1 converges immediately.

No iteration cap. Dedup makes the candidate space finite, so the loop terminates. If the user interrupts, report the current candidate table as-is — that is the record of where the run stopped.

## Phase 5 — Report (once, after convergence)

From the candidate record, cumulative:

- **Fixed** — one line each: root cause + repro/regression test name.
- **Refuted** — one-line refutation each. This is where the noise filter shows its value ("23 candidates, 19 false alarms, here's why").
- **Unproven / Unreproduced** — flagged for human judgment, code untouched.

Never dress up an interrupted run as converged; report the actual last state.
