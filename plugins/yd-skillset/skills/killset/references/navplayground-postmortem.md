# NavPlayground rewrite postmortem

This case calibrates killset; it is not a universal architecture prescription.

## Outcome

A long greenfield rewrite produced a large uncommitted `hub-new/` tree but no
shippable end state. Corrected comparison later showed:

- table update + indexed publication was genuinely strong: roughly 98--99.5%
  faster and 96.9--99.2% less retained-version memory at 16K/64K rows;
- navigation was 19--22% slower on default and 11--13% slower on projectile
  when the old core was correctly retained/warmed;
- Field used 48.8% less live storage on one full-frame shape but was about 8.6x
  slower there and failed strict Arc parity;
- C ABI bodies, managed/browser migration, several systems, and final global
  gates remained incomplete.

The initial benchmark had constructed a fresh old navigation core per sample,
unlike production. It falsely made the rewrite look faster and reported 481/541
old heap calls. Correct lifecycle measurement reduced old calls to 17/21 and
reversed the latency verdict.

## Effort shape

The rewrite accumulated 31 design/test-plan documents with about 85,810 words.
The top-level candidate search itself took roughly 44 wall-clock minutes; the
long tail came from recursive subsystem proof:

- POI: six blind revisions, about 13,827 words, four-hour modification span;
- World/Danger: three audits, about 9,357 words, three-hour span;
- Field: three design revisions before broad differential/performance evidence;
- C ABI: 275 then 285 methods enumerated before most product bodies existed.

These spans overlapped, but all consumed context and agent budget.

## Successes worth retaining

1. **Work classification:** synchronous authoritative intent, latest-only pure
   recomputation, and bounded conserved temporal events is a useful distinction.
2. **Table page-COW publication:** executable evidence supported both latency and
   retained-memory improvement.
3. **Red-first boundary work:** hardened tests caught an expired-identity capacity
   undercount/OOB before release.
4. **Exact pressure contracts:** Field's hard byte boundary was executable.
5. **Corrected comparison:** admitting the reference-lifecycle defect prevented a
   false performance claim from surviving.

## Failures

1. **Risk order was inverted.** Rare concurrency/resource prose came before broad
   parity, production integration, and actual-lifecycle macro viability.
2. **Every local adversarial finding reopened design.** Signed zero, sentinels,
   LRU pin counts, and readback details should have become local red tests.
3. **No saturation rule.** Fresh blind audits continued until GO; POI reached six
   revisions without executable parity/performance.
4. **Surface before capability.** Hundreds of C ABI methods were traced before
   the corresponding components and consumers existed.
5. **Parallelism amplified work in progress.** Multiple design/implementation
   agents expanded different incomplete branches without accepted slice commits.
6. **Progress was counted from files/tests, not accepted artifacts.** A green
   subset and thousands of lines did not answer whether a usable migration landed.
7. **No early kill on regression.** Navigation and Field should have reverted or
   stopped before downstream POI/World/C ABI expansion.

## Counterfactual kill points

| Point | Evidence available | Killset decision |
|---|---|---|
| After top-level architecture (~44 min) | One survivor; no production viability | K20: stop design, build representative spike |
| Before second subsystem design | No accepted vertical slice | K13/K27: block expansion |
| First invalid nav comparison | Reference lifecycle was fresh, not retained | K02: discard numbers; no performance claim |
| Corrected nav comparison | Repeatable 11--22% latency regression | K32: revert/stop navigation slice |
| Broad Field differential | Arc parity red | K31: stop timing and fix/revert before expansion |
| Strict valid Field macro | ~8.6x latency regression | K32: revert/stop Field architecture |
| POI audit revision 3 | Local holes, no executable product evidence | K23/K24/K26: no more whole-design audits |
| Half budget with no accepted end-to-end slice | Large uncommitted tree | K11/K14: stop campaign and report zero accepted |

## Root lesson

Adversarial depth is useful only after viability and only at the abstraction
level of the evidence. The correct response to a local counterexample is usually
a red test, not a larger design document. The correct response to a measured
regression is usually revert, not more downstream implementation.
