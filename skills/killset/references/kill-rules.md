# Canonical kill rules

Actions:

- `PAUSE` — preserve state; do not expand until missing evidence is produced.
- `REVERT` — remove the candidate/slice; baseline remains authoritative.
- `STOP` — end the run and report accepted artifacts plus the failed rule.
- `ESCALATE` — a user decision or new executed evidence is required.

## Admission and baseline

| ID | Trigger | Action | Resume condition |
|---|---|---|---|
| K01 | Existing build or relevant suite is red | STOP | Baseline repaired independently |
| K02 | Claimed reference does not match production lifecycle (warm/retained/contention/build/data) | STOP and discard its numbers | Corrected harness reruns with spread |
| K03 | Reference still exists but no contract differential/capture is established before replacement | PAUSE | Oracle is executable and green |
| K04 | Performance/memory is an explicit quality but no macro baseline exists before broad implementation | PAUSE | Durable actual-lifecycle macro recorded |
| K05 | Required sanitizer/race oracle is unavailable | Mark claim UNPROVEN; use current/sequential fallback | Oracle becomes available and passes |

## Budget and artifacts

| ID | Trigger | Action | Resume condition |
|---|---|---|---|
| K10 | Explicit time/token budget exhausted | STOP | User grants a new budget |
| K11 | More than half the explicit budget spent with no accepted slice | STOP | New scoped campaign |
| K12 | Default 45-minute checkpoint reached with no accepted commit/patch or durable benchmark | STOP | User accepts a smaller slice/new checkpoint |
| K13 | Second slice begins while first is unaccepted or unreviewable | PAUSE | First slice accepted or reverted |
| K14 | More than one slice exists only as an uncommitted/uncheckpointed tree | STOP expansion | Commit or named patch isolates one slice |
| K15 | Status/progress cannot be derived from commits, diffs, commands, or measured gates | Report UNKNOWN/0 accepted | Evidence exists; never estimate around it |

## Design and adversarial work

| ID | Trigger | Action | Resume condition |
|---|---|---|---|
| K20 | Design exceeds 15 minutes before an executable viability plan exists | PAUSE design | Smallest risk spike is named and runnable |
| K21 | More than three candidates are being elaborated | Kill extras; ledger-only | A refutation exposes a genuinely new invariant |
| K22 | Three consecutive candidates die for existing reasons and add no test/tradeoff | STOP ideation (saturated) | New evidence changes the envelope |
| K23 | Third revision at the same abstraction level is requested | ESCALATE | Executed evidence proves architecture, not local contract, is wrong |
| K24 | A local contract hole or implementation defect is used to reopen the whole architecture | Reject redesign; create red test | Test either greens locally or falsifies a pinned invariant |
| K25 | Adversarial claim lacks concrete input/schedule and expected-vs-actual path | Reject/no action | Claim is traced or oracle-backed |
| K26 | Fresh auditors are recursively dispatched until one says GO | STOP audit loop | One bounded recheck of the changed artifact only |
| K27 | Pre-viability run-wide brief exceeds 1,500 words or subsystem addendum 750 | PAUSE and compress | Executable brief within cap |

## Implementation, parity, performance, memory

| ID | Trigger | Action | Resume condition |
|---|---|---|---|
| K30 | New behavior/code was written without observed red | PAUSE; add failing oracle or revert behavior | Red observed, then green |
| K31 | Contract differential/parity is red outside an approved SUSPECT/drop decision | STOP timing; REVERT or fix | Strict parity green |
| K32 | Candidate p50 or p99 repeatedly regresses outside measured spread | REVERT | New candidate beats/equal baseline; absent spread, >3% is presumed material |
| K33 | Hot-path max regresses materially even if median wins | REVERT | Tail returns within spread/approved budget |
| K34 | Peak/live memory regresses or any unbounded queue/cache/history appears without approved tradeoff | REVERT | Bound and peak evidence meet contract |
| K35 | Micro win regresses integrated macro | REVERT fold/slice | Integrated rerun green |
| K36 | Correctness/race/lifetime claim is UNPROVEN | Current/simple implementation wins | Required oracle proves candidate |
| K37 | Public/C ABI/managed/UI surface is specified ahead of an executable capability slice | PAUSE surface work | Capability is green and its real methods are known |
| K38 | Same blocking defect recurs twice after attempted fixes | STOP slice | New design/evidence or user decision |

## Agent governance

| ID | Trigger | Action | Resume condition |
|---|---|---|---|
| K40 | More than two active subagents without explicit authorization or measured need | Interrupt extras | One implementer + one verifier remain |
| K41 | Design-only agents continue after architecture approval | Interrupt | Executable artifact needs bounded verification |
| K42 | Agent task lacks one bounded deliverable, files/scope, gate, and stop condition | Do not dispatch | Brief is verdict-shaped |
| K43 | Blind verifier receives the author's rationale or expected verdict | Invalidate verdict | Fresh minimal-context verification when risk justifies it |
| K44 | Worker advisory benchmark is used as decisive evidence | Reject number | Main/controller runs identical interleaving |
| K45 | Agent misses its local deadline without a verdict/artifact | Interrupt; count zero accepted; do not clone the audit | Brief is materially reduced to a smaller executable question |

## Completion

| ID | Trigger | Action | Resume condition |
|---|---|---|---|
| K60 | Completion claim relies on plans, intent, test count, or “no issue found” rather than requirement evidence | Reject completion | Requirement-by-requirement proof |
| K61 | Any required consumer, migration, parity, performance, memory, sanitizer, or concurrency gate is absent/red | Keep active or STOPPED, never complete | Missing gate executed green |
| K62 | User interrupts | STOPPED report; never dress as complete | Explicit resumed scope |

## Checkpoint template

```text
Decision: <CONTINUE/PAUSE/REVERT/STOP/ESCALATE> because <Kxx>
Budget: <limit/default> | used <time/tokens> | accepted slices <n>
Baseline: <commit/tree + actual lifecycle command>
Accepted artifact: <commit or named patch; none is explicit>
Red -> green: <commands and observed result>
Parity: <predicate/result> | perf: <p50/p99/max+spread> | memory: <live/peak>
Agents: <roles/count>
Next proof: <one slice, required evidence, estimated cost>
```
