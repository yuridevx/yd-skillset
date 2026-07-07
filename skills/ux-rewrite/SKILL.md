---
name: ux-rewrite
description: Whole-product greenfield UX rewrite pipeline — inventory every screen, flow, and user journey; extract per-journey cards that split observable behavior into PINNED / FLEXIBLE / SUSPECT under the journey-oracle rule (v1 witnesses the user's intent, never the screen layout; pixel-for-pixel compatibility is refused, and deliberate dark patterns are adjudicated, never preserved); capture one run-wide theme brief — the new visual theme (direction, tokens, component idiom) every v2 screen is built on; triage journeys through friction / navigation-depth / cognitive-load / consistency / accessibility / hot-cold lenses; then run each journey through a pipeline — ideate greenfield flows under a no-layout-compatibility rule with click minimization bounded by each task's intrinsic-input floor, adversarially refute each journey's ideas in an independent comparative refuter that nominates a worst-case persona or path per survivor, settle survivors by interaction-cost shootout (counted clicks, field entries, screen transitions, decisions, scrolls, waits — recorded per path, never asserted), build fresh v2 screens on the theme brief with red-first flow tests where a harness exists, verify each through the adversarial spine — blind walkthrough hunters over edge personas, a state-hunter attacking back/refresh/deep-link/abandon, safety enforcement at every destructive or irreversible boundary (undo over confirm where reversible, never fewer steps than safety demands), and an accessibility pass — prove in situ by driving the real v2 end to end and re-counting cost against v1 baselines, and audit capability parity with a gap-hunter agent that catches clicks saved by hiding features and can force full re-iteration. Twin creeds — friction does not exist until an interaction-cost count says it does; a flow is not usable until it survives an adversarial walkthrough. Agent dispatch is leveled by a run-wide tag named for its policy — per-claim / batched (default) / minimal / inline — chosen in the invocation or at ASK; cost counting, safety enforcement, parity, and the theme brief are never leveled away, and inline runs record every not-blind judgment. Runs as a chain of named gates, each with a hard exit condition — one user question batch at the ASK gate (where the theme brief is confirmed), fully autonomous end-to-end when "autonomous" appears in the request. Trigger on "ux-rewrite", "/ux-rewrite", "UX rewrite", "rewrite the UX", "redesign the flows", "redesign the user experience", "minimize clicks", "simplify the flows", "user journey rewrite", "rethink the screens".
---

# ux-rewrite

Greenfield rewrite of a product's screens and flows into a v2 UX, gated by interaction-cost accounting and adversarial walkthroughs. Twin creeds:

- **Friction does not exist until an interaction-cost count says it does.** "Feels simpler" is not evidence; a counted path is.
- **A flow is not usable until it survives an adversarial walkthrough** — a blind adversary that tried to get lost, stuck, or hurt in it and failed.

Nothing is adopted on its author's word. The skill is a chain of **gates**: a gate is not a phase to schedule work into — it is a checkpoint with a hard exit condition; you are always standing at exactly one gate, doing its work, and you may not step past it until its exit condition is objectively green. There is no "later": work either happens at its gate or is killed at its gate.

The main thread runs linearly and owns all state. Independent agents (same model as the main thread) are the adversaries of the spine below, dispatched per the **Agent economy** level. Every agent is **blind** — it receives only the artifact under judgment (journey card, flow spec, screen, theme brief), never the generating thread's reasoning.

## The adversarial spine

Every positive claim has a named adversary that must fail to kill it before the claim counts:

| Claim | Adversary | Where |
| --- | --- | --- |
| "this flow idea is better" | refuter agent with kill-checklist | REFUTE |
| "this flow costs less" | counted interaction cost vs v1 baseline | SHOOTOUT / PROVE |
| "this journey is completable" | walkthrough hunter over edge personas | VERIFY |
| "this flow survives real navigation" | state-hunter (back, refresh, deep link, abandon) | VERIFY |
| "this destructive step is safe" | safety enforcement, per-boundary checklist | VERIFY |
| "everyone can use it" | accessibility pass | VERIFY |
| "v2 serves every v1 capability" | gap-hunter | PARITY |
| "v1's behavior here is right" | friction lens; SUSPECT adjudicated before pinning | TRIAGE / JOURNEY |

## Agent economy

Agent dispatch is leveled by one run-wide tag, named for its dispatch policy: **`per-claim` / `batched` (default) / `minimal` / `inline`**. The level rides the invocation ("per-claim", "minimal agents", "inline"; aliases: "lean"/"budget" → `minimal`, "solo"/"agentless" → `inline`) or is settled at ASK, and is recorded in the status-table header.

| Level | REFUTE | VERIFY hunters | PARITY |
| --- | --- | --- | --- |
| `per-claim` | 1 agent per idea | walkthrough hunter on every rebuilt journey; state-hunter on every one; always split | 1 agent/pass |
| `batched` (default) | 1 agent per journey — judges its ideas **comparatively**: can rank, cannot approve contradictory ones | walkthrough hunter on `HOT` / `TRUST-BOUNDARY` / `SUSPECT` / `WON` journeys; state-hunter on every rebuilt journey, split | 1 agent/pass |
| `minimal` | agents only for `HOT` / `TRUST-BOUNDARY` / `SUSPECT` journeys; the rest refuted on paper by the main thread | one merged dual-mandate adversary (personas + state) per risky journey; split only where `HOT` and `TRUST-BOUNDARY` coincide | 1 agent/pass |
| `inline` | all paper | main-thread self-walkthrough against the journey card | main-thread sweep of the capability map + `DROPS:` |

Invariant at every level: interaction-cost counting, safety enforcement at destructive boundaries, the accessibility pass, the theme brief, and the parity audit are agent-free or single-pass and are **never leveled away**. **Escalation:** tags outrank the level — the main thread may raise a single journey one level, recorded in the table; it may never lower one. `inline` means **blindness is gone** — every not-blind judgment is recorded `NOT-BLIND` in the ledger, and the only true adversaries left are the counts and the in-situ walkthrough. Use `inline` for tiny targets or a cheap first iteration to be re-verified at `batched` later.

## Flow map

```
SETUP → SCAN → JOURNEY → TRIAGE → THEME → ASK ══ only user stop (skipped if "autonomous")
                                           │
      ┌────────────────────────────────────▼───────────────────────┐
      │ PER-JOURNEY PIPELINE (journeys flow independently)         │
      │ IDEATE → REFUTE → BASELINE → SHOOTOUT → BUILD → VERIFY     │
      │    ▲                                                       │
      └────┼──────────────────────────────────────┬────────────────┘
           │                          all journeys done
           │                                      ▼
           └── CONFIRMED-GAP capabilities ◄──── PARITY ◄── PROVE
               (full pipeline, never a band-aid) │
                                zero design-defining claims
                                                  ▼
                                               REPORT
```

## Autonomy contract

- **Exactly one user interaction: the ASK gate.** Every question the run will ever need — including the theme brief — is batched there.
- Past ASK, **never** ask "shall I proceed" or present intermediate results as questions. The only stops are a broken build at SETUP and completion at REPORT.
- If the invocation contains "autonomous", "auto", "no questions", or equivalent intent: **skip ASK entirely**, self-decide (including the theme), run end to end. Either way every ASK-class decision is recorded in the dialog.

## The journey-oracle rule

v1's UI is the **intent oracle, never the layout oracle**. It serves three roles until retired: cost baseline, behavior reference, parity reference. The JOURNEY gate splits every observable behavior:

- **PINNED** — users or external systems depend on the exact behavior (data entered, records produced, notifications sent, URLs shared, keyboard shortcuts muscle-memorized by a known user base) → v2 must match exactly.
- **FLEXIBLE** — only the outcome is owed; the path, layout, wording, and widget choice are free → v2 is judged by outcome + counted cost only.
- **SUSPECT** — the friction lens doubts v1 here → **adjudicated before it may be pinned**: `ACCIDENTAL-FRICTION` (a defect — v2 removes it), `DARK-PATTERN` (deliberate friction against the user — surfaced at ASK; never silently preserved, never silently removed), or `LOAD-BEARING-QUIRK` (compliance steps, safety confirmations, behaviors a trained user base relies on — preserved with the reason recorded).

Layout, navigation structure, widget choice, copy, visual style: **FREE** — judged only by counted cost, walkthrough survival, safety, and the theme brief. Pinning an incidental layout is a defect: it freezes the flow. Pinning a dark pattern is worse: it enshrines user harm as contract.

## The theme brief — one run-wide artifact

Every v2 screen is built on **one theme brief**, captured before any screen exists and confirmed at ASK (self-decided in autonomous mode). It is deliberately small — one page:

- **Direction:** 3–5 words naming the aesthetic intent (e.g. "calm, dense, keyboard-first") plus what it deliberately rejects.
- **Tokens:** type scale, color palette (with light/dark values and contrast-checked pairs), spacing scale, radius, elevation, motion policy.
- **Component idiom:** the one blessed pattern per recurring need — forms, tables, dialogs vs inline editing, primary/secondary actions, empty states, error display. One pattern each; a second pattern for the same need must displace the first, never coexist.
- **Density and platform posture:** target devices, breakpoints, pointer vs touch, keyboard support level.

Where a frontend-design skill or an existing design system is available, use it to derive the brief — never to skip it. The brief is contract for BUILD: a v2 screen that wins its cost shootout but breaks the idiom is a failed screen; consistency is what makes the tenth screen free to learn. Drift found later is a defect against the brief, not a reason to grow it — the brief only changes by explicit decision recorded in the status table.

## Interaction-cost model

The benchmark analog. Cost is **counted, never asserted**, per journey path, as a tuple:

`(C clicks/taps, F field entries, T screen transitions, D decisions, S scrolls, W waits)`

- **Clicks/taps** — every pointer activation on the path, including opening menus that exist only to be opened.
- **Field entries** — each field the user must focus and fill (not keystrokes; a prefilled or defaulted field costs 0).
- **Screen transitions** — every navigation, page load, or modal that replaces the user's context.
- **Decisions** — at each step, the count of competing interactive choices the user must scan to find the right one; a screen of 40 equally-weighted buttons is expensive even at 1 click.
- **Scrolls / waits** — viewport moves the path requires; spinners and artificial delays on the path.

Counted on the **main path** and every **declared edge path** (error, empty state, first run) of the journey. Counts land in the status table exactly like benchmark numbers.

**The intrinsic floor:** every journey has an information-theoretic minimum — the inputs the task genuinely requires (a payment needs an amount and a recipient; deletion of something irreversible needs a real confirmation). A journey already at its floor is `KEPT-AS-IS`; no redesign may claim to beat the floor, only to approach it. Clicks removed by dropping a required input, hiding a capability, or gutting a safety step are not savings — PARITY and safety enforcement exist to catch exactly that.

## Universal rules (govern every gate)

### Count-first

No redesign exists before the number it must beat does. Every journey's v1 cost tuple is counted at BASELINE before any candidate flow is specced. Where the product has an automated flow-test harness, BUILD is red-first per the **test-practice** skill: a failing flow test per journey main path before the screens that green it; per fix, a failing repro first. Where no harness exists, the walkthrough script (exact step list an agent or human can execute) is the test artifact, written before the screen and kept with it.

### No-copy

**Never copy an existing screen and restyle it — rebuild from the journey card and the theme brief.** Copies smuggle in old navigation structure, dead controls, and layout residue — exactly what the journey-oracle rule frees you from. The v2 screen tree is designed, not accreted: navigation mirrors the journey inventory, one clear purpose per screen, and the theme brief's idiom everywhere. v1 stays open as behavior reference only.

### Hot-journey rules (frequent paths only)

Hot journey = performed many times per day per user, tagged at TRIAGE. Cold journeys (setup wizards, rare admin tasks) are exempt and judged on clarity over speed — a once-a-year task should optimize for zero recall, not zero clicks.

1. **Minimum steps toward the intrinsic floor;** defaults over choices; remembered context over re-entry.
2. **Complete keyboard path** — the whole journey without touching the pointer.
3. **No confirmation for reversible operations — undo over confirm.** Confirmations are spent only where the safety rule demands them; spending them on reversible ops trains users to click through the ones that matter.
4. **No dead ends:** every error state on a hot path names the fix and offers the action.
5. **Bounded worst case:** the edge paths (error, empty, slow network) are counted too; winning the happy path while the error path strands the user is a loss.

### Safety boundaries (the trust-boundary analog)

Destructive, irreversible, costly, or outward-facing actions (delete, payment, publish, send, grant access) are `TRUST-BOUNDARY` steps. Click minimization **stops at the boundary**: these steps keep explicit, informed confirmation — or genuine undo where the operation is truly reversible — and legally or compliance-required steps are `LOAD-BEARING-QUIRK` by definition. A flow that reaches the floor by shaving a safety step is defective regardless of its count.

### Counted, not vibed

Every cost comparison states both tuples side by side in the status table. A candidate wins only if it strictly improves the components that matter for the journey's tags without regressing the others past a stated, recorded trade ("+1 decision on a cold path buys −2 transitions" is a legal recorded trade; an unstated regression is not). Edge paths count in the comparison, not just the happy path.

## State

Artifacts on disk; everything else lives in the conversation:

```
ux-rewrite/              (scratchpad by default; in-repo if chosen at ASK)
├── theme/               the theme brief
├── journeys/            journey cards + cost counts (v1 baseline and v2)
└── <target>-v2/         the rebuilt screens/flows — nearest sibling to the original target
```

Working state is an in-dialog status table — one line per journey, idea, and capability: id, tags (`HOT/COLD`, `TRUST-BOUNDARY`, `SUSPECT`), current status, the cost tuples or one-line reason that justify it, and — once VERIFY has run — the journey's **verification ledger**: which adversaries ran and claims-vs-actioned. Status vocabulary: journeys end `REBUILT | KEPT-AS-IS`; ideas end `REFUTED | LOST | WON`; capabilities end `COVERED | SMALL-DIFF | DROPPED-BY-DESIGN` (via `CONFIRMED-GAP` when PARITY forces re-entry); SUSPECT behaviors end `ACCIDENTAL-FRICTION | DARK-PATTERN | LOAD-BEARING-QUIRK`. Header: `Assumptions:`, target, platform posture, theme-brief status, agent-economy level, iteration counter.

Restate the current table compactly at every gate exit — never drop a line; the restatement is what carries the record through context compaction. Anything an agent needs (journey card, flow spec, theme brief, capability map, `DROPS:` list) is passed explicitly in its prompt.

## The gates

### SETUP

1. Determine target (whole product, named app area, or given screens). Ambiguity → reasonable reading, recorded under `Assumptions:`.
2. Verify the product **builds and runs** — a UX cannot be baselined from screenshots of a broken app. Broken → stop and report.
3. Establish the **walkthrough vehicle**: how flows will actually be driven — browser automation (e.g. chrome-devtools tooling), an e2e harness, a simulator, or, last resort, code-reading plus manual step scripts. Record which; it determines whether BASELINE/PROVE counts are executed or derived.
4. Invoke **test-practice** if an automated flow-test harness exists or will be created.

**Exit:** app runs, walkthrough vehicle named, setup summary stated.

### SCAN

Walk the whole target. Record every **screen** (route, purpose, entry points), every **flow** (the click-path graph between screens), and the **journey map** — what users accomplish, at intent level: the task, who performs it and how often, entry points, the outcome, and every capability on the way (modes, filters, bulk actions, shortcuts, config-driven behaviors). **Capabilities are features:** keyboard shortcuts, URL addressability, saved state, offline behavior, notification side effects — the easiest things to lose silently in a redesign.

Every journey gets one status-table line. Inventory only — no judging, no ideas.

**Exit:** every screen, flow, and journey in the status table; each journey's screens and capabilities listed.

### JOURNEY

Per rewrite-candidate journey (a cold `KEPT-AS-IS` journey needs no card), extract one **journey card** — the single artifact every downstream adversary receives:

- User intent: what the user is trying to accomplish, in one paragraph, and how often.
- Inputs the task intrinsically requires (this defines the floor) and the outcome owed.
- Entry points, preconditions, and the states the flow must survive (empty, error, partial, interrupted).
- Trust boundaries on the path.
- The **PINNED / FLEXIBLE / SUSPECT** split of every observable behavior, per the journey-oracle rule.

The card is intent, not implementation: it must not name v1's screens, layout, or navigation — that is exactly the freedom the FREE rule protects.

**Exit:** every rewrite candidate has a card; every SUSPECT behavior is adjudicated or queued for ASK.

### TRIAGE

Six lenses, one pass each:

1. **Friction:** counted cost vs the intrinsic floor — the gap is the opportunity. Emits `SUSPECT` tags on doubted behaviors (feeding JOURNEY's adjudication). A journey suspected of harming users is a first-class rewrite candidate — rewrite-because-hostile ranks with rewrite-because-slow.
2. **Navigation depth:** how many transitions from each entry point to each task; orphan screens; detours through hub screens that add no decision.
3. **Cognitive load:** decisions per step, recall the flow demands (codes, names, prior answers), mode confusion, inconsistent placement of the same action.
4. **Consistency:** where v1 solves the same need two ways; each duplicate idiom is a rewrite candidate that the theme brief's component idiom will collapse.
5. **Accessibility:** keyboard reachability, focus order, labels, contrast — findings become requirements on v2, not optional polish.
6. **Hot/cold:** tag every journey by frequency; hot → hot-journey rules apply. `TRUST-BOUNDARY` tags land on every destructive/irreversible step.

**Exit:** every journey tagged on all six lenses.

### THEME

Draft the theme brief per its section above: audit v1's current visual language first (what to keep, what the new direction rejects), then propose direction, tokens, component idiom, and platform posture. If the user's invocation already names a direction ("make it feel like X"), the brief elaborates it rather than inventing one.

**Exit:** a complete one-page theme brief, ready for confirmation at ASK.

### ASK — the only user stop

Present in one batch: assumptions taken, journey inventory with tags, **SUSPECT adjudications** (with the recommended verdict each — every `DARK-PATTERN` is decided here: fix or preserve, on the record), the **theme brief** for confirmation or correction, the agent-economy level (when the invocation didn't set it), planned capability drops, and every genuine either-way call (artifact location, disputed tags, scope). One question round; answers are recorded. In autonomous mode: skip, self-decide, record.

**Exit:** every ASK-class decision recorded; theme brief confirmed; every SUSPECT behavior adjudicated. From here to REPORT, zero questions.

### IDEATE (pipeline entry — re-entered by PARITY)

Per journey, generate alternative flows under **no-layout-compatibility** rules: only the intent must survive — free to merge screens, kill screens, change navigation, replace a wizard with one form or one form with progressive disclosure, move work to defaults, or eliminate the journey by making its trigger unnecessary. Killing a screen by changing an upstream flow is a first-class idea — the best flow is the one the user never has to enter.

An idea that removes or narrows a capability must declare `DROPS: <capability> — <rationale>` (surfaced at ASK, or self-decided and recorded). This is what lets PARITY distinguish deliberate removal from accidental loss.

Cost honesty at the door: every idea states its estimated tuple and where the cost went — clicks removed by moving them to another journey, to setup, or to cognitive load must say so. Hot-journey ideas must obey the hot-journey rules; safety-shaving ideas die here.

**Exit:** every live journey has its ideas recorded, each with an estimated tuple and any `DROPS:` declared.

### REFUTE (independent adversaries, dispatched per the Agent economy)

One blind refuter per **journey**, given only that journey's ideas, the journey card, the current flow spec, and the kill-checklist; it judges the ideas **comparatively** — may rank, may not approve two contradictory ones. The kill-checklist:

- Does the click saving survive the edge paths, or only the happy path?
- Where did the cost actually go — another journey, first-run setup, recall burden, decision density?
- Does the merged/simplified screen still scale at realistic data volume (the 3-item design at 3,000 items)?
- Does the idea silently narrow a capability or an input domain without a `DROPS:`?
- Do hot-journey rules hold — keyboard path, undo-over-confirm, no dead ends?
- Does any trust-boundary step lose its safety property?

Whoever refutes — agent or main thread — also **nominates one worst-case persona or path per surviving idea** — the user or situation on which that idea should be weakest (first-time user, screen-reader user, error mid-flow, huge dataset, interrupted session); it joins the journey's counted paths for BASELINE and SHOOTOUT.

Verdict rule — **doubt flows forward** (the count downstream is a cheap objective judge); certainty-it-cannot-win or a safety violation → `REFUTED` with a one-line reason.

**Exit:** every idea dispositioned; survivors each carry a nominated worst-case path; verdicts recorded.

### BASELINE

For every journey with a surviving idea: count **current v1** — main path, declared edge paths, and the nominated worst-case paths — via the walkthrough vehicle (drive it where possible; derive from code and record `DERIVED` where not). Also capture v1's end-to-end journey timings and any existing funnel/analytics numbers as PROVE's bar.

**Exit:** the tuples every candidate must beat are recorded. No candidate flow is specced before its baseline exists.

### SHOOTOUT

Per journey: spec each surviving idea as a **flow spec** — the exact screen sequence, controls, defaults, and edge-path behavior; cheap sketches, not built screens. Count each spec on the same paths as BASELINE.

**Honesty gate before any count counts:** the main thread (at `per-claim`, an agent) audits each spec against the journey card — every capability present, every trust boundary intact, edge paths specified, hot-journey rules held. A spec that wins by omission is fixed and **re-counted** (the fix may eat the win), or `LOST`.

**Adoption judgment — proportionality, not gates.** The judges are counted cost, walkthrough survivability, safety, and consistency with the theme brief — nothing else has a vote. The burden of proof grows with the novelty a candidate adds: an unfamiliar interaction pattern must beat the conventional one by enough to be worth teaching. Anchors:

- A tie or an unstated trade is not a win; ties go to the more conventional flow.
- Fewer clicks that became more decisions or more recall is a trade, judged not assumed.
- Cold journeys almost never earn novel interaction patterns; hot journeys can.
- Doubt → conventional wins. A rationale you'd be embarrassed to read back is a rejection.

**Exit:** every candidate `WON` or `LOST`, with both tuples and the rationale in the status table.

### BUILD

Build the journey's screens in `<target>-v2/` from its `WON` spec — or, with no winner, as the **simplest faithful rebuild** — fresh screens per the no-copy rule, on the theme brief, into a deliberately designed navigation tree. Per journey, in order:

1. **Flow test red** (or walkthrough script written) for the main path against the empty v2.
2. **Build to green:** main path works end to end on real components from the theme idiom.
3. **Edge paths built and tested:** empty, error, interrupted — per the journey card's state list.
4. **PINNED behaviors verified** against v1; SUSPECT behaviors assert the **adjudicated** behavior, not v1's — an `ACCIDENTAL-FRICTION` or fixed `DARK-PATTERN` dies here, by design, and the fix is recorded.

**Exit:** journey works in v2 on the theme brief, main + edge paths green, PINNED verified.

### VERIFY — adversarial proof of usability

Per rebuilt journey, depth keyed by its tags:

1. **Walkthrough hunter — risk-gated:** journeys that are `WON`, `HOT`, `TRUST-BOUNDARY`, or `SUSPECT` get one blind walkthrough hunter receiving the journey card, the flow, and the nominated worst-case personas only. Mandate: complete the task as each persona; report every point of hesitation, wrong turn, dead end, or unrecoverable state. Accepted claims become fixes with their repro recorded; rejections get a one-line reason.
2. **State-hunter — every rebuilt journey:** attack the flow with real navigation weather — back button at every step, refresh mid-flow, deep link into the middle, duplicate tab, session expiry, abandon-and-return. Anything that loses user data or strands the user is a defect.
3. **`TRUST-BOUNDARY` — safety enforcement:** per boundary step, verify the safety property survived the redesign — informed confirmation or genuine undo, correct scoping ("delete which one?"), no default-to-destructive. Findings block the journey until fixed.
4. **All — accessibility pass:** keyboard-only run of the main path, focus order, labels, contrast per the theme brief's checked pairs. Findings are defects, not polish.

The journey's verification ledger (adversaries run, claims vs actioned) lands in the status table.

**Exit:** every hunter claim triaged, every boundary safe, accessibility pass clean, ledger recorded.

### PROVE

Drive the real, built v2 end to end via the walkthrough vehicle and **re-count every journey's actual tuples** — specs estimated; the build is what counts. Compare against v1 baselines: a journey that won on spec but regressed as built (the real dialog added a transition, the real list needs a scroll) is fixed or reverted to the conventional flow. Verify the theme brief held across all screens in one consistency sweep.

**Exit:** every adopted win confirmed in the running product; regressions fixed or reverted; counts final.

### PARITY (one gap-hunter pass — an agent at every level above `inline` — can force a new pipeline iteration)

The gap-hunter (one blind agent; at `inline`, the main thread, recorded `NOT-BLIND`) receives v1, v2, the journey map, the capability map, the `DROPS:` list, and the SUSPECT adjudication record — not the rewrite thread's reasoning. Mandate: find capabilities reachable in v1 whose **intent** is unreachable in v2, and triage severity itself:

- **Design-defining** — a task, mode, or capability users rely on; losing it forces workarounds or exodus. Only these come back as `PARITY-CLAIM`s.
- **Small** — incidental layout details, cosmetic differences, one-extra-click reachability. Returned as `SMALL-DIFF` notes: recorded, never actioned.

The **spirit rule** for judging: moved, merged, renamed, restyled, reached by a different path = fine. Unreachable capability, silently narrowed input domain, lost keyboard path, dropped shareable URL, gutted safety step = gap. A fixed `ACCIDENTAL-FRICTION` or adjudicated `DARK-PATTERN` is not a gap — it matches the adjudication record.

Main thread adjudicates each claim: **refute with a walkthrough** showing how v2 serves the intent / **DROPPED-BY-DESIGN** (matches a declared drop) / **CONFIRMED-GAP**. An undeclared drop is always a confirmed gap — even if dropping was right, the decision must become explicit, never accidental.

**Confirmed gaps get the full pipeline, not band-aids:** each re-enters at IDEATE — ideate how the capability fits v2's design natively, refute, count, shoot out, build on the theme brief, verify through the full spine, prove in situ.

**Exit:** a full gap-hunter pass (a fresh adversary) returns **zero new design-defining claims**.

### REPORT

From the status table, cumulative:

- **Adopted** — per journey: v1 vs v2 tuples on every counted path, and where the savings came from.
- **Theme** — the confirmed brief and the consistency sweep result.
- **Refuted / lost / reverted** — one line each ("23 flow ideas, 7 adopted, here's why the rest died").
- **SUSPECT adjudications** — every accidental friction removed, every dark pattern decision, every load-bearing quirk preserved and why.
- **Dropped capabilities** — every `DROPPED-BY-DESIGN` with its rationale; gaps confirmed, re-entered, and covered.
- **Verification ledger** — per journey: which adversaries ran (and at what level), claims vs actioned, every `NOT-BLIND` judgment.
- **Walkthrough vehicle** and exact steps to re-drive and re-count any journey.

Never dress up an interrupted run as complete; report the actual last status table as-is.

**Exit:** the report delivered from the final status table, nothing omitted — every journey, idea, capability, and adjudication accounted for.
