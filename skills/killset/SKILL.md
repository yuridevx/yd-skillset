---
name: killset
description: Meta-governor for expensive or long-running agent work — overlays yd-skillset workflows with evidence-first admission, risk-ranked viability spikes, bounded design and refutation, reviewable checkpoints, and mandatory pause/revert/stop rules when parity, performance, memory, scope, budget, or completion evidence fails. Use with algo-rewrite, tdd-rewrite, ux-rewrite, perf-tune, mastermind, autonomous or persistent goals, broad refactors, multi-agent campaigns, requests such as "keep working", "unbounded design", "do not stop", "kill set", or "budget guard", and whenever a task can consume substantial time or tokens without producing an accepted artifact.
---

# killset

Govern costly work so effort buys accepted evidence, not motion. A long run is
healthy only while it repeatedly converts budget into reviewable artifacts that
survive the user's real acceptance gates.

## Authority

- Use the co-active yd skill as the baseline workflow. Killset governs whether
  work may start, expand, continue, be adopted, or be called complete.
- Explicit user instructions outrank this skill. Otherwise killset wins over a
  co-active skill on budget, checkpoint, agent fanout, rollback, and stop policy.
- `autonomous`, `persistent`, `keep working`, and `unbounded` remove routine
  questions; they do not waive evidence gates or authorize unlimited elaboration.
- Never mark a stopped run complete. Report the accepted artifacts and the exact
  kill that stopped it.

## Required references

At setup, read these completely:

- [baseline-flows.md](references/baseline-flows.md) — retain the useful spine of
  the selected yd workflow and insert the kill gates at the correct points.
- [kill-rules.md](references/kill-rules.md) — canonical triggers, actions, and
  resume conditions.

Read [navplayground-postmortem.md](references/navplayground-postmortem.md) when
diagnosing a failed campaign, calibrating a rewrite, or deciding whether a
counterexample is architecture-defining.

## State

Keep one compact kill ledger in the conversation:

```text
Budget: <explicit limit | default checkpoint> | used: <measured>
Baseline: <commit/tree> | lifecycle: <production-equivalent?> | oracle: <state>
Accepted: <commits or named checkpoint patches, oldest..newest>
Current slice: <one vertical outcome> | red: <test/benchmark> | state: <status>
Parity: <PASS/RED/NA> | perf: <PASS/RED/UNPROVEN/NA> | memory: <...>
Agents: <active count and roles> | decision: <CONTINUE/PAUSE/REVERT/STOP/ESCALATE>
```

Accepted means reviewable and independently rerunnable. Design prose, test
counts without coverage, uncommitted multi-slice trees, and advisory agent
reports are not accepted artifacts.

## Flow

```text
ADMIT -> BASELINE -> VIABILITY -> DESIGN -> SLICE -> REFUTE -> PROVE
                                                        |          |
                                                        v          v
                                                      REVERT <- CHECKPOINT
                                                                   |
                                                        ITERATE or REPORT
```

Stand at one gate. Do not schedule work into future gates.

### ADMIT

1. Pin the user's real objective, required deliverable, acceptance gates, and
   explicitly excluded actions.
2. Select one vertical slice whose outcome can be accepted or rejected alone.
3. Record the user's budget. With no explicit limit, the default first evidence
   checkpoint is 45 wall-clock minutes or one vertical slice, whichever comes
   first; design gets at most 15 minutes of that checkpoint.
4. If goal/token telemetry exists, capture it now and at every checkpoint.
5. Start at most two subagents: normally one implementer and one verifier. More
   requires explicit user authorization or a measured latency reason. Every
   brief gets a deadline inside the current checkpoint; analysis-only work
   defaults to 10 wall-clock minutes.

### BASELINE

1. Verify the existing project and its relevant tests are green.
2. Capture the reference while it exists. Match the actual production lifecycle:
   retained versus fresh state, warm versus cold, contention, data, build flags,
   and publication/query boundaries.
3. For behavior work, build the contract-level differential/property oracle.
   For performance or memory claims, capture p50/p99/max, spread, allocations,
   live/peak bytes, and machine profile before candidate implementation.
4. An invalid, incomparable, or red baseline kills every dependent claim. Fix
   the harness or stop; never explain the mismatch away.

### VIABILITY

Attack the reasons the entire approach could be rejected before elaborating it:

- broad parity on a deterministic representative corpus;
- the real end-to-end integration seam;
- actual-lifecycle macro latency when performance matters;
- peak/live memory and overload behavior when boundedness matters;
- the required race/sanitizer oracle when concurrency is being introduced.

Build only enough architecture to run this spike. VIABILITY must finish before a
second product module or a complete public/C ABI surface is designed. Failure
means `REVERT` or `STOP`, not “continue and optimize later.”

### DESIGN

1. Follow the co-active skill's candidate method. Elaborate at most three
   candidates; record any additional open-ended candidates in at most two
   sentences each.
2. Open-ended search stops at saturation: three consecutive new candidates die
   for already-recorded reasons and reveal no new invariant or executable test.
3. Use one comparative refuter. A refuter must provide a concrete in-contract
   input/schedule and expected-versus-plausible-actual outcome.
4. Classify every surviving claim:
   - `ARCHITECTURE-KILL` — falsifies a pinned run-wide invariant;
   - `CONTRACT-HOLE` — becomes a red contract test;
   - `IMPLEMENTATION-DEFECT` — becomes a failing repro and minimal fix;
   - `PLAUSIBLE/OUT-OF-SCOPE` — record or reject without action.
5. Only `ARCHITECTURE-KILL` reopens architecture. Local holes never trigger a
   complete design rewrite. Two revisions at one abstraction level are the
   default maximum; a third requires executed evidence and `ESCALATE`.
6. Before VIABILITY is green, keep the run-wide brief under 1,500 words and any
   subsystem addendum under 750. Compress rather than grow a proof document.

### SLICE

1. Implement exactly one vertical slice through a real entry point and output.
2. Follow test-practice: observe red, implement minimally, reach focused green,
   then run the relevant integration/sanitizer/race gates.
3. Do not start a second slice while the first is unaccepted or unreviewable.
4. Produce one commit when commits are authorized. Otherwise produce one named
   checkpoint patch/diff and stop expansion until it is reviewed.
5. After architecture approval, do not dispatch design-only agents. Adversaries
   inspect an executable artifact or a bounded plan at its own gate.

### REFUTE

Use the co-active skill's risk ladder, capped by ADMIT's agent budget. Give a
blind verifier the artifact and contract, not the author's rationale. A local
confirmed finding enters the red-first fix protocol and is verified once. Do not
order recursive fresh audits until an agent eventually says GO.

The same blocking condition twice means the chosen approach is not converging:
`STOP` the slice and report it. Do not spend a third turn polishing the same
claim without new external evidence.

If an agent misses its local deadline without a verdict/artifact, interrupt it
and count the pass as zero accepted work. Do not dispatch a replacement auditor
unless the brief is first reduced to a materially smaller executable question.

### PROVE

1. Correctness/parity gates run before performance numbers count.
2. Rerun candidate and reference interleaved under the recorded lifecycle and
   measured noise. A repeatable regression outside spread loses. Without a noise
   model, more than 3% p50 or p99 regression is presumed material.
3. Any new unbounded path, overload loss outside contract, or peak-memory
   regression loses unless the user explicitly approved that tradeoff.
4. Missing required oracle means `UNPROVEN`; current/simple code wins.
5. An isolated win that regresses the integrated macro flow is reverted.

### CHECKPOINT

At 45 minutes, the explicit budget boundary, or completion of a slice—whichever
comes first—restate the ledger from evidence:

- commit or named patch;
- exact changed product files;
- observed red and final green commands;
- parity and performance/memory numbers with spread;
- remaining objective and next slice cost.

No accepted artifact by the checkpoint is `STOP`, not “percent complete.” If
more than half an explicit budget is spent without one accepted slice, stop
immediately. Status questions are answered from commits/diffs/tests/benchmarks;
unknown estimates stay unknown.

### ITERATE OR REPORT

Continue only when the current slice is accepted, the next slice is still in the
original objective, and its proof plausibly fits the remaining budget. Rebaseline
when the accepted slice changes the comparison surface.

Completion requires a requirement-by-requirement audit against the original
request. `STOPPED`, `REVERTED`, `UNPROVEN`, partial parity, missing consumers, or
an uncommitted current slice are never renamed complete.

## Report shape

Lead with the decision and artifact:

```text
Decision: CONTINUE | PAUSE | REVERT | STOP | ESCALATE
Accepted: <commit/patch and what it proves>
Killed: <rule id, failed evidence, what was reverted or left untouched>
Numbers: <baseline -> candidate, spread, parity, memory>
Next: <one slice and estimated proof cost, or none>
```
