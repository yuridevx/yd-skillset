# Baseline yd flows and killset overlays

Use the selected yd skill as the domain workflow. Killset inserts early viability,
artifact checkpoints, and stop/revert semantics; it does not replace red-first,
contract oracles, measured noise, or proportional adoption.

| Baseline skill | Preserve | Killset insertion |
|---|---|---|
| `algo-craft` | SCOPE -> PIN -> BASELINE -> DESIGN (<=3) -> IMPLEMENT -> VERIFY -> MEASURE; current code wins on doubt | Treat this as the default unit of work. One target, one slice, one artifact. Its baseline/adoption rules are the model for every other flow. |
| `perf-tune` | Macro baseline, profile-ranked vectors, <=3 candidates, interleaved measured noise, UNPROVEN veto, one fold commit, integrated regression reverts | Reuse directly. Kill a campaign that lacks a production-equivalent macro lifecycle, attacks unprofiled work, or reaches the checkpoint without a fold or durable benchmark artifact. |
| `harden` | Only concrete failure scenarios; refute noise; oracle-backed red test before edit; minimal fix; clean rescan | Reuse its evidence bar for adversarial claims. Architecture commentary, preferences, and plausible-but-untraced concerns never trigger edits or redesign. |
| `test-practice` | Red-first, contract differential/property tests, deterministic boundaries, race-oracle requirement, concrete adversarial claim bar | Every local adversarial finding becomes a test/repro at the owning slice. It does not reopen run-wide design unless it falsifies a pinned invariant. |
| `tdd-rewrite` | SETUP/SCAN/CASE/TRIAGE, one run-wide brief, tiered plans, executed v1 capture, red-first BUILD, VERIFY, PARITY | Add a risk-ranked VIABILITY spike before full use-case plans or the second module. When performance/memory are explicit requirements, the old/new production macro is a gate, not a late warning note. Build and accept one vertical use case before elaborating the rest. |
| `algo-rewrite` | Contract cards, intent oracle, adversarial shapes, benchmark shootout, fresh v2, integrated PROVE, parity audit | Move one representative macro baseline and one component shootout ahead of whole-codebase elaboration. A losing representative approach kills its architecture family before every component gets a design. |
| `ux-rewrite` | Capability inventory, journey contracts, interaction-cost evidence, real-product walkthrough, accessibility and state hunters | Build and drive one highest-risk journey before specifying every screen. A hidden capability or worse real journey kills the layout family. |
| `mastermind` | Main thread decides; workers self-verify; blind verification at genuine dead ends | Killset overrides “dispatch freely”: default <=2 active agents, bounded briefs, no recursive design agents, and one accepted artifact before follow-up expansion. Expensive main-thread tokens and aggregate worker/token budget both matter. |

## Rewrite ordering

For a broad rewrite, use this order:

```text
green v1 -> production macro + differential corpus
         -> minimal architecture needed for one representative vertical slice
         -> representative parity/perf/memory viability
         -> accept/reject architecture family
         -> one use case at a time through the baseline skill
         -> consumers/public surface only after capability exists
```

Do not enumerate a complete C ABI, managed facade, screen catalog, or subsystem
test matrix before the representative vertical slice survives viability.

## Open-ended search semantics

“Unbounded candidates” means there is no arbitrary numeric idea ceiling before
saturation. It does not mean unlimited elaboration, audits, tokens, or wall time.
Record extra candidates tersely and stop when three consecutive candidates add no
new invariant, test, benchmark shape, or tradeoff. Only survivors earn detail.
