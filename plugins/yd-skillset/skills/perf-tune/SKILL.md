---
name: perf-tune
description: Whole-project, profile-driven performance campaign that establishes durable benchmarks and folds proven optimizations back in place — build macro benchmarks for the flows that matter (committed to the repo as a first-class deliverable), profile them to rank where the time actually goes, attack only the dominant vectors (the smallest prefix of the ranking that dominates the profile, cap 3 per iteration, each carrying its measured share and Amdahl ceiling — a ceiling inside macro noise is not a vector, and speculation never is), pin each vector's contract with a differential + property harness under the PINNED/FLEXIBLE/SUSPECT split before anything races (SUSPECT behavior adjudicated first — candidates race against corrected behavior, never against a bug), record a granular micro baseline under measured noise, hypothesize at most 3 candidates refuted on paper, race survivors in parallel git worktrees via racer agents or sequentially inline — racers build but never judge; decisive numbers are produced by the main thread running identically-built candidates in alternation with recorded spread — adopt by proportionality (a win must clear the spread, complexity must be worth the ceiling, UNPROVEN vetoes, doubt → current code wins), verify winners through the risk-gated adversarial ladder (blind edge-hunter on hot or complexity-adding winners, race oracle + interleaving-hunter on concurrent ones, boundary enforcement, hardening-oracle pass), fold each winner onto a campaign branch as one commit with the numbers in the message and confirm in situ against the macro baseline (integration regression → revert), then re-baseline, re-profile, and iterate — bottleneck rankings shift after folds, which is the point. Iteration contract — default 2 iterations, user-set N or unbounded until stop, early exit on GOAL-MET or a dry CONVERGED iteration. Triple creed — a bottleneck does not exist until a profile names it; performance does not exist until a benchmark says it does; correctness does not exist until it survives adversarial refutation. One user stop at the ASK gate after the first profile, fully autonomous when "autonomous" appears in the request. Testing method loads from the test-practice skill; language specifics (harness, profiler stack, hot-path rules, oracles) from a <lang>-toolkit skill (e.g. msvcpp-toolkit) or are derived at setup. A single user-named target is algo-craft's job; a whole-codebase greenfield rewrite is algo-rewrite's; this skill is for "make it faster" when nobody yet knows where the time goes. Trigger on "perf-tune", "/perf-tune", "find the bottlenecks", "profile and optimize", "make the project faster", "speed up the project", "performance pass", "performance campaign", "establish project benchmarks", "set up benchmarks and optimize".
---

# perf-tune

Profile-driven performance campaign over a whole project, folded back in place. Where **algo-craft** optimizes one named target and **algo-rewrite** rebuilds a codebase greenfield, perf-tune answers a different ask: **"make it faster" when nobody yet knows where the time goes.** Triple creed:

- **A bottleneck does not exist until a profile names it** — no speculative optimization; every vector attacked carries its measured share and ceiling.
- **Performance does not exist until a benchmark says it does.**
- **Correctness does not exist until it survives adversarial refutation.**

Nothing is adopted on its author's word: racer agents build but never judge; every decisive number is produced by the main thread under measured noise; every adversary is blind per test-practice's **Adversarial** calibration standard.

**Scope routing:** the user names one function/class/file → algo-craft. The ask is a greenfield rewrite → algo-rewrite. Project-level speed with targets unknown until profiled → this skill. A vector whose only winning design demands a whole-subsystem redesign ends `REFERRED` (algo-rewrite territory for that subsystem) — the run never grows silently.

**Deliverables are two:** the folded optimizations *and* the benchmark suite itself — macro benchmarks for the flows that matter plus one granular micro benchmark per attacked vector, committed to the repo as durable regression infrastructure. Establishing the benchmarks is first-class: a campaign that folds nothing but leaves the suite behind has still delivered.

## Vocabulary

- **Vector** — one dominant bottleneck under attack: location, profile share, **ceiling** (Amdahl: the macro gain if the vector cost zero).
- **Candidate (A/B/C)** — one hypothesized optimization of a vector; at most 3 per vector.
- **Campaign branch** — `perf/<slug>`, created at SETUP off the invocation branch; every fold is one commit here. Merging it anywhere is the user's act, never the skill's, unless the invocation pre-authorized it ("merge to main when done").
- **Fold** — squash of a winning candidate onto the campaign branch, numbers in the commit message, confirmed in situ.

## The iteration contract

Recorded in the header at SETUP, from the invocation (or defaults):

- **Budget** — N full iterations; **default 2**; "until I say stop" → `UNBOUNDED`.
- **Goal** — a measured target when the user states one ("p99 under 200 ms on ingest", "2× import throughput"); otherwise "converged".

The campaign stops at the first of: **`GOAL-MET`** (macro numbers meet the target — early, even with budget left) | **`BUDGET-SPENT`** | **`CONVERGED`** (a dry iteration: the fresh profile yields no vector whose ceiling clears the noise-and-proportionality bar, or every vector ended `DRY`/`REVERTED` — budget is never burned speculating) | **`STOPPED`** (user interrupt — report the table as-is, never dressed up as complete). Every iteration ends with an interim numbers paragraph. **Exactly one user interaction: the ASK gate, iteration 1.** Past it — and in every later iteration — zero questions; "autonomous" in the request skips ASK entirely, self-decide and record.

## Execution modes — controlled parallelism

| Mode | RACE runs as | Agents | When |
| --- | --- | --- | --- |
| `race` (default) | one git worktree + one racer agent per surviving candidate | ≤3 racers concurrently (one vector's race) + risk-gated hunters (≤2/vector) | default |
| `inline` | candidates sequentially in place on the campaign branch, reverted between trials | zero — main-thread self-audit replaces hunters, every such judgment recorded `NOT-BLIND` | "inline"/"solo"/"agentless" in the request, no worktree support, or a single surviving candidate |

Vectors are processed **sequentially in dominance order**; a vector's candidates race in parallel. That is the whole concurrency story — folds stay attributable and the profile stays honest. Invariant in either mode: oracles, the differential harness, measured noise, red-first, and the `UNPROVEN` veto are never leveled away.

## The pin rule

perf-tune is behavior-preserving by definition: **the current code is the intent oracle for every candidate** (test-practice, Properties & differential). At PIN, split the vector's observable behavior:

- **PINNED** — the contract fixes the exact output → the differential harness compares exact.
- **FLEXIBLE** — a predicate only (±tolerance, any valid ordering, error model free to change) → equivalence predicate or property.
- **SUSPECT** — the correctness scan (harden's four classes: boundary, concurrency, lifetime/resource, garbage data) doubts the current behavior → **adjudicated before anything races**: a defect gets a failing repro and a minimal fix red-first, and candidates race against the *corrected* behavior — bug-for-bug compatibility is refused, and a bottleneck is never benchmarked against wrong output.

Internals, data structures, allocation strategy, module-local signatures: **FREE** — judged only by correctness, benchmark, simplicity, and **fold size**: a candidate that must restructure callers across the project dies at HYPOTHESIZE or refers the vector to algo-rewrite. Pinned tests are contract-level, so they outlive the fold and stay valid across future campaigns.

## Universal rules

### Measured noise

Noise is measured, not assumed. Repetition count per the toolkit's Benchmarking section (≥5 hand-rolled), run-to-run **spread recorded next to the numbers** — a win must clear the recorded spread, not an asserted one. The harness defeats dead-code elimination (DoNotOptimize-equivalent or a volatile/checksum sink) or the numbers are fiction. Machine profile recorded once, at SETUP. `CONCURRENT` code benchmarks **contended** per the toolkit's pattern — uncontended numbers for a concurrent structure are not numbers. **Cross-worktree comparisons are never single-run:** the decisive micro bench is executed by the main thread over identically-built candidates in A,B,C,A,B,C… alternation.

### Red-first

No optimization exists before the number it must beat is recorded — macro at BASELINE, micro at BENCH. Every fix along the way (SUSPECT defect, hunter claim, boundary finding, regression) follows test-practice's fix protocol: failing repro → minimal fix → green, test kept as a regression guard.

### Realtime rules (hot vectors only)

Hot = on the per-item / per-frame / per-request path, tagged at PROFILE. Cold vectors are exempt and judged on simplicity + throughput only.

1. **Zero allocations steady-state** — where the language exposes counts, the hot micro bench asserts 0; nonzero fails regardless of speed.
2. **No hidden allocators** — the toolkit's Hot-path list (closures/boxing, string temporaries, container growth/rehash, exception machinery).
3. **Bounded worst case beats better average** — pre-reserve to known bounds; no amortized-only guarantees where the spike lands hot.
4. **No blocking** — no lock waits, syscalls, I/O, logging. Anything lock-free introduced here is `CONCURRENT` and owes the full ladder — a fast structure that races is not fast, it is wrong.
5. **Errors without unwinding or allocating** — value/status returns per the toolkit's error model.
6. **Layout matches access pattern** — contiguous over node-based.

Latency is judged **median + p99 + max** — winning median while regressing max is a loss.

## State

Header: assumptions, target, language + toolkit (or derived sections), toolchain, machine profile, oracle availability, **mode**, **budget**, **goal**, iteration counter. One status-table line per vector and candidate: id, tags (`HOT/COLD`, `CONCURRENT`, `TRUST-BOUNDARY`, `SUSPECT`), profile share + ceiling, status, numbers with spread (micro; macro once folded), and — after VERIFY — the verification ledger (adversaries run, claims vs actioned, `NOT-BLIND` judgments, anything `UNPROVEN`). Status vocabulary: candidates end `REFUTED | LOST | WON | UNPROVEN`; vectors end `FOLDED | DRY | REVERTED | REFERRED`; the campaign ends `GOAL-MET | BUDGET-SPENT | CONVERGED | STOPPED`. Restate the table compactly at every gate exit — never drop a line; the restatement carries the record through context compaction.

On disk:

```
repo:        perf/<slug> campaign branch (folds land here) + the bench suite
             (macro + per-vector micro, in the project's bench layout)
scratchpad:  race worktrees <scratchpad>/race/<vector>-<a|b|c> — removed at FOLD,
             pruned at SETUP when re-entering after an interrupt
```

## Flow map

```
SETUP → BASELINE → PROFILE → ASK ══ only user stop (iteration 1; skipped if "autonomous")
                      ▲       │
                      │       ▼  per vector, dominance order (≤3 per iteration)
                      │   PIN → BENCH → HYPOTHESIZE → RACE → ADOPT → VERIFY → FOLD
                      │       │ all vectors dispositioned
                      └── ITERATE   re-run macro baseline, re-profile
                              │ GOAL-MET | BUDGET-SPENT | CONVERGED | STOPPED
                              ▼
                           REPORT
```

## The gates

### SETUP

1. Target = whole repo, or the subtree the user scoped. Budget, goal, and mode from the invocation; ambiguity → reasonable reading, recorded under `Assumptions:`.
2. Verify the project **builds and existing tests pass** — broken baseline → stop and report; nothing can be baselined on red.
3. **Invoke test-practice; resolve the `<lang>-toolkit`** (e.g. msvcpp-toolkit) or derive its sections — Benchmarking (harness order **and the macro profiling stack**), Hot-path rules, Concurrency oracles, Boundary enforcement, Hardening oracles. Confirm which oracles actually run on this machine — a missing race oracle never blocks the run; it dooms specific claims to `UNPROVEN`.
4. Install the benchmark harness library-first per the toolkit; wire alloc/scope instrumentation gated on a build flag with the red-first self-check (guard catches a deliberate allocation; flag off → compiles to nothing).
5. Create the campaign branch `perf/<slug>`; `git worktree prune` stale race worktrees from any interrupted prior run.
6. Record the machine profile (CPU, cores, hybrid, caches, power plan per the toolkit's hygiene rules) — numbers are meaningless without it.

**Exit:** build green, tests pass, harness self-check red→green, header stated.

### BASELINE — macro

Identify the flows that matter: the existing bench/perf suite if there is one, else derived from the user's goal, entry points, and docs. Build or extend **macro benchmarks** driving those flows end-to-end with realistic data — committed to the suite, runnable from a clean checkout with documented commands. Run under measured noise: median + p99 + max, allocation counts where the language exposes them cheaply, spread recorded. These numbers are the campaign's bar: the goal is judged against them and every ITERATE re-runs them.

**Exit:** every flow's baseline recorded with spread; the suite reruns from documented commands.

### PROFILE

Profile the macro benchmarks with the toolkit's profiler stack — sampling first, the gated instrumentation scopes only where sampling is ambiguous. The metric follows the goal: wall time by default, allocation/GC profile when memory-bound, IO waits when IO-bound. Rank by inclusive share. **Vectors = the smallest prefix of the ranking that dominates the profile, cap 3 per iteration**, each recorded with location, share, and ceiling. A ceiling inside the macro spread is not a vector. A user-nominated suspect is profiled first — not dominant → reported with its measured share and attacked only if the user insists (recorded). Tag each vector: `HOT/COLD`, `CONCURRENT`, `TRUST-BOUNDARY`, `SUSPECT`.

**Exit:** ranked profile in the table; ≤3 vectors with shares + ceilings; the residual ranking recorded for the report.

### ASK — the only user stop (iteration 1; skipped when "autonomous")

One batch, with numbers in hand: assumptions, baselines, the ranked profile, chosen vectors with ceilings, candidate sketches, mode, budget + goal, bench-suite location, **dependency policy** (default: already-installed libraries only; new dependencies only if authorized here or in the invocation), campaign-branch policy (offered at REPORT vs pre-authorized merge). Answers recorded; autonomous → self-decide and record every ASK-class decision.

**Exit:** every ASK-class decision recorded. From here to REPORT, zero questions — later decisions are resolved by the recorded decisions, the adoption judgment, or a benchmark.

### Per vector, in dominance order

#### PIN

Split PINNED / FLEXIBLE / SUSPECT per the pin rule; adjudicate every SUSPECT now (defect → failing repro + minimal fix red-first; candidates race against the corrected behavior). Build the **differential + property harness against the current code, green** (test-practice, Properties & differential) — contract-level, so it outlives the fold. Write the class map (equivalence classes + BVA) — it shapes the differential generator and is what any hunter later receives.

**Exit:** harness green; every SUSPECT adjudicated.

#### BENCH — granular

Micro benchmark isolating the vector, committed to the suite: realistic small / typical / large **plus one worst-case shape** (plus every adversarial shape HYPOTHESIZE later nominates — added before the race); allocation counts; contended when `CONCURRENT`. Recorded with spread: this is the bar every candidate must clear.

**Exit:** micro baseline with spread recorded; hot vectors have allocation counts.

#### HYPOTHESIZE

At most 3 candidates, one line each: idea, expected win, complexity cost. **Library-first**: the toolkit's shortlist and already-installed dependencies are candidates before hand-rolled cleverness; new dependencies only per the recorded policy. **Refute each on paper** with the kill-checklist: complexity math at realistic N; constant factors (cache, branches, allocations); sync contract implementable where `CONCURRENT`; realtime compliance where `HOT`; fold size (must not restructure callers project-wide); a plausible win that cannot clear the recorded spread is dead on arrival. Whoever refutes nominates one **adversarial bench shape** per survivor — the input on which that candidate should be weakest; it joins the micro bench before the race. Zero survivors → the vector ends `DRY` — a defended "leave it alone" is a success, not a failure.

**Exit:** every candidate refuted or surviving with an adversarial shape nominated.

#### RACE

`race` mode: per surviving candidate, `git worktree add <scratchpad>/race/<vector>-<c> -b perf/<slug>/<vector>-<c>` off the campaign head, plus one **racer agent** (same model as the main thread). The racer receives the pin card (contract, split, sync contract), class map, its candidate line, target files, the micro-bench and differential commands, and the realtime rules where `HOT`; it implements red-first in its worktree to differential-green and reports a diff summary with advisory numbers. **Racers build; they never judge.**

- **Correctness gate before any number counts** (per candidate, in its worktree): differential + property harness green; full suite green; `CONCURRENT` → race oracle silent under bounded contended stress, else the candidate is `UNPROVEN` and cannot win.
- **Decisive numbers by the main thread:** identical build flags in every worktree; the micro bench (adversarial shapes included) run in A,B,C,A,B,C… alternation at the toolkit's repetition count, spread recorded.

`inline` mode: same order, no agents — candidates implemented sequentially on the campaign branch, reverted between trials, benched interleaved against the current code directly.

**Exit:** every candidate's numbers — or `UNPROVEN` / lost-at-correctness — in the table.

#### ADOPT

Proportionality, not gates — the four judges are **correctness, proof, benchmark, simplicity**; nothing else has a vote. The complexity class (simpler/same, more complex, hack) is assigned by the hunter that runs at VERIFY — self-assigned and flagged only when no hunter runs. Anchors:

- Inside the recorded spread is not a win — that's measurement, not judgment.
- Simpler or same: any real win adopts; a tie adopts only if the code got simpler. A library already in the project counts as simpler.
- More complex: the win must be worth maintaining **against the vector's ceiling** — a 40%-of-runtime vector can buy complexity a 3% vector cannot.
- Hack (fragile cleverness, UB-adjacent tricks, layout gymnastics): only an exceptional, measured win on a genuinely `HOT` vector, contained behind a clean interface and commented as such.
- Hot vectors: a max/p99 regression or new steady-state allocations lose regardless of median.
- `UNPROVEN` vetoes adoption regardless of the numbers — fast-but-unproven loses to slow-but-proven.
- Doubt → the current code wins (`DRY`). A rationale you'd be embarrassed to read back is a rejection.

**Exit:** exactly one `WON` (or the vector is `DRY`), every candidate carrying a one-line rationale — the rationale is the gate.

#### VERIFY — winner only, before the fold

Risk-gated adversarial ladder in the winner's worktree; cap 2 hunters per vector (`inline`: zero — self-audit recorded `NOT-BLIND`; oracles unchanged):

- **Always:** full suite + differential/property harness green.
- **`HOT` or complexity-added:** one blind **edge-hunter** (contract card, class map, code — test-practice Adversarial); accepted claims become red tests fixed per the fix protocol; the hunter assigns the complexity class.
- **`CONCURRENT`:** race oracle + bounded contended stress, plus one blind **interleaving-hunter** on the sync contract + code. No race oracle on this machine → `UNPROVEN` → the fold is refused and the vector ends `DRY` — never folded on suspicion.
- **`TRUST-BOUNDARY`:** boundary enforcement per test-practice/toolkit — standard mechanisms enabled; hand-written validation gets its red/green pair.
- **All:** the toolkit's hardening-oracle build over the vector's tests; findings are defects, fixed red-first.
- **Cold + sequential + simpler:** zero agents — the paper refutation was the proportionate adversary.

**Exit:** ladder recorded in the vector's ledger; oracles silent or the fold refused.

#### FOLD

Squash the winner onto the campaign branch — **one commit**: vector, candidate, micro before/after with spread, complexity class. Remove every race worktree and candidate branch (losers' numbers are already in the table). **Confirm in situ:** re-run the affected macro benchmark(s) on the campaign branch — the micro win must survive integration; a macro regression outside the recorded spread → revert the fold, vector ends `REVERTED`, the current code stands. Full suite green after the fold.

**Exit:** fold committed and confirmed (or reverted and recorded); the repo clean of race artifacts.

### ITERATE

When the iteration's vectors are all dispositioned: re-run the full macro baseline — post-fold numbers become the new bar and the goal check — then **re-profile**: folds shift the ranking, which is the point. Select the next iteration's vectors by the same dominance rule; a vector already `DRY`/`REVERTED` re-enters only if its share grew (the code under it changed). Emit the interim numbers paragraph. Stop per the iteration contract; otherwise next iteration.

### REPORT

Cumulative, from the final table:

- **Macro before/after** per flow (median/p99/max with spread) across all iterations, against the goal.
- **Per vector:** share, ceiling, candidates with race numbers and why the losers died, fold commit id, verification ledger (adversaries run, claims vs actioned, `NOT-BLIND` judgments, anything `UNPROVEN`).
- **`DRY` / `REVERTED` / `REFERRED`** vectors with one-line reasons — defended "leave it alone" verdicts are successes.
- **The bench suite:** location and exact rerun commands; the machine profile.
- **The residual profile:** the top of the ranking left unattacked — what the next campaign would target.
- **Stop reason:** `GOAL-MET | BUDGET-SPENT | CONVERGED | STOPPED`.

The campaign branch is offered for merge; it is merged only where the invocation or ASK pre-authorized it. Never dress up an interrupted run as complete — the last status table is the record of where it stopped.
