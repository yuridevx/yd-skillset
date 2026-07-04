---
name: cpp-harden
description: Iterative, low-noise C++ hardening review that only fixes proven defects — boundary violations, concurrency bugs (data races/deadlocks), memory leaks/lifetime errors (UAF, double-free), and uninitialized/garbage data. Works TDD-style, gather candidates, adversarially refute them, reproduce each survivor with a failing sanitizer-backed test BEFORE any edit, fix minimally, and loop full passes until the findings ledger converges (minimum 2 iterations). Linear, no subagents. Style, performance, naming, and modernization are explicitly out of scope. Trigger on "harden", "cpp-harden", "/cpp-harden", "harden this C++", "make this C++ safe", "check boundaries and concurrency".
---

# cpp-harden

Iterative TDD hardening loop for C++. Core principle: **a defect does not exist until a failing test says it does.** Every phase is a filter; code edits happen only at the very end of each iteration, only on what survived every filter. You run this **linearly — no subagents, no fan-out.** All state lives in one ledger file.

## Defect contract — the only four classes in scope

1. **Boundary** — OOB read/write, off-by-one, unchecked size/index, integer overflow feeding an allocation or index.
2. **Concurrency** — data race, deadlock, torn read/write, missing synchronization on shared state.
3. **Lifetime/leak** — memory leak, use-after-free, double-free, dangling reference/iterator.
4. **Garbage data** — uninitialized reads, type-punning UB, partially-constructed objects observed.

Style, naming, performance, architecture, "modernize", missing `const`, refactoring opportunities: **out of scope, say nothing about them.** This silence is the contract that makes the output low-noise.

## State: the ledger

Create `harden-ledger.md` (scratchpad by default; in-repo if the user asks). One entry per candidate:

```
### C<seq> — <file>:<line> [<class>] state=<STATE> iter=<found-in-iteration>
Scenario: <concrete input/state/interleaving → specific wrong outcome>
Disposition: <refutation reason | test name | why unreproduced>
```

State machine: `CANDIDATE → REFUTED | SURVIVED → PROVEN | UNREPRODUCED | UNPROVEN → FIXED`.
The ledger is append/update only — never delete entries. It is your memory across iterations and context compaction; re-read it at the start of every phase.

## Phase 0 — Setup gate (once)

1. Determine target: the diff, user-given paths, or whole repo (ask only if genuinely ambiguous).
2. Verify the project **builds and existing tests pass**. Broken baseline → stop and report; you cannot do red/green on red.
3. Detect oracles: ASan/LSan/UBSan/TSan/MSan availability, test framework, how to add and run one test. On MSVC: ASan yes, TSan no — concurrency survivors without a TSan oracle end as `UNPROVEN`, never fixed on suspicion.
4. Create the ledger with a header recording target, toolchain, oracle availability, iteration counter.

## Iteration loop — Phases 1–4, repeat until converged

### Phase 1 — Hunt

Read the target code in **four passes, one lens per defect class** (one lens at a time beats one pass with four things in mind).

Entry bar: a candidate must have a **concrete failure scenario** — specific input/state/interleaving → specific wrong outcome. "This index isn't obviously checked" is not a candidate. "`parse()` passes `len-1` from a network header to `memcpy` without comparing against `buf.size()`; a header claiming len=0 wraps to SIZE_MAX" is.

Dedup against the **entire ledger, including REFUTED entries** — same location (± line drift) and same scenario means already dispositioned; skip it. Only genuinely new scenarios enter as `CANDIDATE`. No judging plausibility, no fixing — just record.

Scope per iteration: iteration 1 reads the whole target. Iteration N>1 reads (a) every file a Phase 4 fix touched plus its direct callers, and (b) any code a prior refutation depended on that has since changed — if "caller validates the index" was the refutation and the caller changed, reopen that entry to `CANDIDATE`.

### Phase 2 — Refute

For each `CANDIDATE`, actively try to **kill it**. Re-read with full context: callers, callees, class invariants, threading model. Standard refutations per class:

- Boundary: validated upstream? type range-limited? buffer provably large enough by construction?
- Concurrency: actually reached by >1 thread? lock held by caller? confined to pre-thread init? atomic via a typedef you missed?
- Lifetime: does ownership actually escape? is the "leak" a deliberate singleton?
- Garbage: zero-initialized via aggregate init or a calloc path you didn't trace?

**Verdict rule: to survive, the full failure path must be traced end-to-end. Uncertainty → `REFUTED (unproven path)` with a one-line reason.** This bias is what kills the noise. Otherwise `SURVIVED`.

### Phase 3 — Reproduce (TDD red)

For each `SURVIVED`, write **one minimal failing test**. The oracle is a sanitizer or an assertion — never "I'm pretty sure":

| Class | Oracle |
| --- | --- |
| Boundary | ASan report, or assertion on the corrupted value |
| Lifetime/leak | ASan/LSan report (Debug CRT leak check on MSVC) |
| Garbage data | MSan/UBSan report, or assertion on the garbage value |
| Concurrency | TSan report from a deterministic interleaving (barriers/latches forcing the order); fallback: bounded stress loop under TSan |

Run it. Three outcomes:

- Fails as predicted → `PROVEN`. Keep the test.
- Doesn't fail → `UNREPRODUCED`. Delete the test, **no edit**, record why the repro failed.
- No oracle possible on this platform → `UNPROVEN`, no edit. Honest "believed but unproven" beats a speculative fix.

Concurrency rule: a TSan report **is** the reproduction. "Looks racy" with TSan silent across the stress loop is `UNREPRODUCED`.

### Phase 4 — Fix (TDD green)

Only `PROVEN` entries. For each:

1. Minimal fix — smallest diff that makes the failing test pass. No drive-by refactoring. Anything new spotted mid-fix becomes a fresh `CANDIDATE` in the ledger, never fixed inline.
2. Rerun the repro test → passes.
3. Rerun the full suite (existing tests + every repro test accumulated across all iterations) → green. Repro tests from earlier iterations are regression guards for later ones.
4. Ledger → `FIXED`.

The suite must be fully green before the next iteration starts.

### Convergence — the only exit

**Converged = the ledger reached a fixed point on iteration ≥ 2:** a full Phase 1–3 pass with **zero new candidates AND zero state transitions** (nothing reopened, nothing changed state). A clean iteration 1 does not count — always run at least 2 full iterations; the confirming pass must itself be clean.

No iteration cap. Dedup makes the candidate space finite, so the loop terminates. If the user interrupts, the ledger lets a later session resume exactly where it stopped.

## Phase 5 — Report (once, after convergence)

From the ledger, cumulative:

- **Fixed** — one line each: root cause + repro/regression test name.
- **Refuted** — one-line refutation each. This is where the noise filter shows its value ("23 candidates, 19 false alarms, here's why").
- **Unproven / Unreproduced** — flagged for human judgment, code untouched.
- **Convergence trail** — one line per iteration: `iter 2: 4 new candidates, 3 refuted, 1 proven, 1 fixed` … ending `iter N: 0 new, 0 transitions — CONVERGED`.

Never dress up an interrupted run as converged; report the actual last state.
