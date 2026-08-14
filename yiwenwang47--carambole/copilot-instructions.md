## carambole

> > ## ▶ ACTIVE TASK — START HERE (next session, continue without re-asking)

# Carambole — French Grammar Trainer

> ## ▶ ACTIVE TASK — START HERE (next session, continue without re-asking)
>
 Everything below is DONE, MERGED to `main`, and PUSHED to origin (2026-06-19).
> Use the same discipline (branch off `main`; subagent-driven implement → spec
> review → code-quality review per task; build gate `xcodebuild build ... -scheme
> carambole -destination 'platform=iOS Simulator,name=iPhone 16'
> CODE_SIGNING_ALLOWED=NO` = EXIT 0; screenshot UI changes; commit per task; update
> this block as you go).
>
> **▶ PROGRESS BAR + TIME-TO-FINISH ESTIMATE (branch `progress-tracking-estimate`,
> head `c2eeccc` — NOT yet merged/pushed; 2026-06-21).** Answers "how far through the
> book am I / how much more time to finish" on the book home. Driven by web research
> (Lingvist's learned/learning/at-risk buckets; course-completion stats: only ~8%
> finish, realistic pace-based timelines → 3.2× more likely to finish) → design
> `docs/plans/2026-06-21-progress-tracking-design.md`. Locked decisions: estimate =
> **recent actual mastery pace** (not the daily-goal setting); finish-line = element
> **mastered** (`distinct_correct ≥ 3` OR suspended — the live `satisfiedIDs` gate);
> headline = **3-segment book bar** (maîtrisé / en cours / à venir); scope = bar +
> estimate only (History untouched). Also FIXED a confusing daily zone: the old
> `Aujourd'hui · X new · Y due` (to-do) sat right above `N appris · M révisés` (done)
> — two look-alike number pairs, opposite meaning. New: to-do counts move onto the
> **buttons** (`Apprendre · N` / `Réviser · M`, badge hidden at 0); one `fait : …`
> done-line remains. **3 tasks, subagent-driven (implement → spec ✅ → code-quality ✅
> each), + final whole-branch review = SHIP:** **T1** `90361d8` `mastery_log(element_id
> PK, day_start)` table + `user_version` **3** migration (additive; backfilled once
> from `last_review_date`, forward-only) + write-hooks in `upsertReviewStatus` AND
> `setSuspended` (the only 2 satisfaction paths) + `fetchMasteryLog()`. **T2**
> `594f80a` pure `ProgressEstimate` value type (`Core/ProgressEstimate.swift`):
> segments + `percentComplete` (caps 99 unless truly done, floors 1 if any progress)
> + trailing-14-day mastery pace ÷ elapsed-calendar-days → `daysRemaining` /
> `projectedFinish`, with an early-state gate (suppress unless ≥3 days history AND ≥3
> window masteries; nil → "Continue quelques jours…"); deterministic (injected
> `today`/`calendar`), 8-case `#if DEBUG _selfTest()` wired at launch. **T3** `c2eeccc`
> `BookView`/`BookViewModel`: capsule 3-segment bar + % + 3-state estimate line, French
> formatting (`≈ N j`/`≈ N mois`, `~15 août`, `8,5/j`) IN the view; `mastered`/segments/%
> come from the LIVE satisfied gate, `mastery_log` feeds ONLY the pace (no desync).
> Build EXIT 0 (sim id `7C7BA0A7-…`; `name:iPhone 16` still broken). Launch self-tests
> pass. Populated bar SCREENSHOT-validated against the real `BookView` via the
> `CB_SCREEN` harness (reverted; same gotcha as prior) — caught+fixed a doubled-`≈`.
> STILL UNTESTED HERE: literal on-device tap into BookView (BookView sits behind
> `LibraryView`, taps unautomatable — see NEXT (a)). FINISHING THE BRANCH (merge/PR)
> awaiting user choice.
>
> **▶ DRILL CARD: VERTICAL LOCKED + FIT-TO-HEIGHT (commit `a5dff57` — MERGED to
> `main` + PUSHED; 2026-06-21).** Bug: drill content overflowed with no way to
> reach the bottom, and vertical scroll was inconsistent (locked on live drills,
> scrolling in review — distracting). Root cause: each card's `ScrollView` fought
> the carousel's `simultaneousGesture` drag → non-deterministic vertical
> arbitration. Fix (DrillView.swift only): **drop the per-card `ScrollView`** →
> vertical is now deterministically LOCKED everywhere; the card is instead SIZED to
> fit. (1) `GeometryReader` `compact` mode (`geo.height < 540`): when the region is
> tight (keyboard up, or small screen / SE) hide the secondary chrome (cue,
> translation, Indice/Écouter) so prompt + input + footer always fit — also stops
> the English translation from half-giving the answer while typing. (2) Tighter
> `contentStack` spacing/padding (`xl`→`lg`, `xl`→`md`) reclaims ~the exitBar
> height. (3) Build/MC token input wrapped in a bounded `.basedOnSize` ScrollView —
> invisible for normal drills (locked feel holds), scrolls ONLY the slots+bank when
> a fully-built long answer (worst case: 10 slots / 16 chips) outgrows the card;
> prompt + footer stay fixed. (4) Review explanation wrapped in a bounded 220pt
> `.basedOnSize` ScrollView so long bilingual explanations fit without scrolling the
> whole locked card. Build EXIT 0. SCREENSHOT-VALIDATED on sim against the REAL
> `DrillView` via a temp `#if DEBUG` env-gated harness (`CB_HARNESS`/`CB_SCREEN`,
> launched with `SIMCTL_CHILD_*` env, reverted + deleted — same revert-then-
> uninstall/reinstall gotcha applies), using REAL worst-case content and a faithful
> exit-bar-height spacer: worst construis fits all 16 chips, typing+keyboard goes
> compact and fits, review explanation bounded-scrolls. STILL UNTESTED HERE (taps
> unautomatable): the literal place-all-10-tokens-then-scroll gesture + the two
> nested `.basedOnSize` scrolls coexisting with the carousel horizontal swipe —
> logic-sound, empty states verified, needs on-device tap-through (see NEXT (a)).
>
> **▶ CURRENT STATE (2026-06-19) — full course + library + branding. ✅ MERGED + PUSHED.**
> - **Architecture:** **Book → Module (×25) → Element (×20) → Example (×5+)** with
>   **element-level SRS** — master a rule via SHUFFLED example sentences, not a
>   memorized token order. Module unlock = all elements mastered (`distinct_correct
>   ≥ 3`, `StudySettings.masteryThreshold`, or suspended). Apprendre/Réviser live
>   INSIDE the book; `newPerDay` counts ELEMENTS (default 10, range 3–20). Content
>   is in-memory from bundle JSON (`ContentLoader` DTOs); only `review_status`
>   persists (keyed `element_id`, + `distinct_correct` / `mastered_example_ids` /
>   `suspended`; dev migration drops old shapes).
> - **Content: ALL 25 modules authored** (A1→B1: 01 articles/gender → 25 si-clauses;
>   500 elements, ~2600 grammar-audited sentences). One JSON per module in
>   `Resources/Topics/`, listed in `manifest.json` (`moduleFiles`). Generated by two
>   subagent workflows (architect → author → grammar-audit per module).
>   `tools/lint_topics.py` (rewritten for the module schema: 20 elements/module, ≥5
>   examples/element, multiset banks, unique ids) passes.
> - **UI:** **Library shelf** is the root (`LibraryView`) — pick a book; "Français —
>   Fondations" live, "Conversation"/"Intermédiaire" locked "Bientôt". Then
>   `BookView` (module ToC + Apprendre/Réviser). Drill UI unchanged (token-build /
>   MC, translation, Indice, "Trop facile" gate now per element).
> - **Branding:** carambole star mark (`CaramboleMark.swift` → `CaramboleLogo`),
>   app icon, **pure-white background** (`AppColor.paper` = FFFFFF).
> - Build-green; library + 1/10/25-module home + element drill screenshot-validated;
>   startup self-tests pass. Design + as-built deviations:
>   `docs/plans/2026-06-19-book-modules-elements-design.md`.
>
> **▶ SHIPPED TO PHONE (2026-06-19).** App now installed + running on a physical
> iPhone 16 (iOS 26.5), `wen.carambole` v1.0, signed with team `QMV4SPL476` (Apple
> ID `yiwenwang9702@gmail.com`, free/personal). `main` (incl. the persistence work
> below) PUSHED to origin. One-time device setup done (Developer Mode on, account
> signed in, device paired, profile auto-created). Per-build install = see
> "Install on a physical device" under Build/verify. Caveat: free profile expires
> in 7 days → reinstall to refresh (user data survives). Still TODO: actually
> tap-through the learn→master→unlock→review loop on-device (was never automatable
> here) — see NEXT (a).
>
> **▶ PERSISTENCE + UX (branch `persistence-migration` → MERGED to `main`, PUSHED;
> 2026-06-19).** Book-1 grammar re-audited (5 agents × 25 modules + lint):
> 0 errors. Three changes shipped (build EXIT 0, code-review passed, commit
> `6ad8353`): **(1)** `PRAGMA user_version` migration in `CoreDataManager.migrate()`
> — v1 = element-keyed `review_status` baseline, idempotent so existing installs
> upgrade in place WITHOUT dropping accumulated rows (drops ONLY the legacy
> exercise-keyed table). Future schema change = add an `if version < N` block with
> an additive `ALTER`; **never edit an earlier block or DROP the table.** Closes the
> silent-SRS-data-loss footgun on rebuilds. **(2)** Settings → "Exporter ma
> progression": checkpoint + copy `carambole.sqlite` to temp, share via system
> sheet (backup before risky rebuilds; `DataManager.exportSnapshot()` +
> `databaseFileURL`). **(3)** Mid-session **exit bar** (leading ✕) in
> `DrillSessionView` during intro + drilling → `onFinish`; progress already durable
> via per-answer upsert, so partial mastery survives exit. NOT tapped on sim
> (automation still blocked).
>
> **▶ DRILL BACK/FORWARD NAV RESTORED (branch `fix-drill-back-button` → MERGED to
> `main`, NOT pushed; 2026-06-19).** Commit `25fd220`. Regression: the
> Book→Module→Element rewrite of `DrillSessionView` dropped its drill history and
> stopped passing the back/forward params to `DrillView`, so the "Exercice
> précédent" chevron never rendered (`canGoBack` defaulted false). `DrillView`
> still had full support (`canGoBack`/`onBack`/`canGoForward`/`onForward`/
> `reviewData` + chevrons in its `header`) — only the wiring was missing. Fix
> (DrillSessionView only): the element driver streams shuffled examples across
> elements with no flat array, so reintroduce an append-only `history: [Exercise]`
> of presented examples + a `cursor` index; `submittedAnswers`/`resultByIndex`
> (keyed by history index) feed read-only revisits; `answeredCount`/`correctCount`
> derive from the dicts (idempotent on re-answer); `presentNextExample` appends +
> parks cursor at the frontier; `goBack`/`goForward` walk the cursor;
> `isReviewing = cursor < history.count-1` drives `reviewData` + chevrons. Mirrors
> the pre-rewrite (`d419532`) semantics. Build EXIT 0; code-review APPROVE (no
> CRITICAL/HIGH). VERIFIED on sim against the REAL `DrillSessionView` via a
> temporary `#if DEBUG` state-seed (reverted, NOT on main): frontier state shows
> `‹ EXERCICE` back chevron on the live drill; cursor-back state shows the prior
> drill read-only (`RÉVISION` + `VOTRE RÉPONSE` + forward chevron). Sim taps still
> not automatable (Apple-events denied `-1743`, no idb/cliclick) → the literal
> answer→advance→tap-back gesture is still untapped here (logic code-reviewed),
> see NEXT (a). GOTCHA hit: after reverting the debug scaffold + rebuilding, the
> sim still ran the stale seeded build until `simctl uninstall`+reinstall — always
> reinstall the clean build before judging "broken."
>
> **▶ PROGRESS + HISTORY TRACKER (branch `progress-history-tracker` → MERGED to
> `main` + PUSHED; 2026-06-19).** Three additions, all per-book-scoped. **(1) Daily
> progress line** on the book home under "X new · Y due": `N appris · M révisés`
> (work DONE today) — `appris` = elements first-seen today, `révisés` = elements
> re-graded today MINUS first-seen-today (genuine reviews only). Taps through
> (chevron) to the History page. `BookViewModel.refresh()` computes both off-main
> (`learnedToday`/`reviewedToday`), intersecting with `book.allElements`. **(2)
> History page** (`Features/History/HistoryView.swift`, pushed in LibraryView's
> NavigationStack, own back chevron): per-day rows newest-first (`19 juin · 3 appris
> · 5 révisés`) with two-tone proportional bars (saffron=learned, hairline=reviewed)
> scaled to the busiest day, an "approximatives" footnote, and an empty state.
> Pure-static `HistoryViewModel.activities(...)` buckets by `Calendar.startOfDay`
> (same reviewed-minus-first-seen rule as the today line), self-tested deterministic
> (fixed UTC calendar). **(3) Réviser round-size toggle** `10·20·50` (default 20),
> inline segmented "RÉVISION PAR TOUR" under Apprendre/Réviser; `StudySettings.
> reviewRoundSize` (snaps to nearest allowed option) persisted in the store;
> `reviewSessionElements()` slices the existing due+overflow queue with
> `.prefix(reviewRoundSize)` — layered ON TOP of `reviewTarget` (still the day's soft
> pool). **Data layer:** `review_log(element_id, day_start)` table + `user_version`
> **2** migration (additive block; backfilled once from `last_review_date` so
> existing installs aren't all-zeros — reviewed-history before this is forward-only,
> documented); `upsertReviewStatus` now also `INSERT OR IGNORE`s a `review_log` row
> per graded answer; reads `fetchElementIDsFirstSeen/ReviewedLogged(on:)`,
> `fetchFirstSeenByElement()`, `fetchReviewLog()`. Build EXIT 0; per-task spec +
> code-quality + whole-branch reviews all APPROVE. BOTH new screens
> SCREENSHOT-VALIDATED on sim with a temporary `#if DEBUG` seed fixture (reverted,
> NOT on main — same revert-then-uninstall/reinstall gotcha applies). Design:
> `docs/plans/2026-06-19-progress-history-tracker-design.md`. **GOTCHA (build gate):
> the CLAUDE.md `name:iPhone 16` destination now FAILS** — `iPhone 16` sims are only
> at iOS 18.5 while `OS:latest` is 26.5 (iPhone 17 line), so the name+latest pair
> matches nothing (EXIT 70). Build against the explicit booted device id instead
> (`-destination 'id=<udid>'`; the iPhone 16 / 18.5 sim here was
> `7C7BA0A7-C367-4CD4-AAE5-462B1F754986`).
>
> **▶ TOKEN-BANK SHUFFLE + HARDER CONSTRUIS DISTRACTORS (commit `365f8fc` —
> committed + PUSHED to `main`; 2026-06-19).** Construis (`buildFromTokens`) + MC
> drills presented candidates in the AUTHORED order — answer tokens first in answer
> order — so a construis was solvable by tapping left-to-right and the MC answer sat
> at row 0 in **1162/1166** drills; banks carried only ~1.2 distractors. Two fixes.
> **(1) Deterministic shuffle** — `Exercise.displayBank`: Fisher-Yates seeded from
> the exercise `id` via `splitmix64` off `Exercise.seed64` (the FNV-1a hash now
> shared with `typeBit`). STABLE per id, so chip order never reshuffles on a SwiftUI
> re-render or a read-only review-nav revisit, yet never mirrors the answer order.
> Guards: result ≠ authored order; for multi-token answers the first N chips never
> spell the answer; MC (1-token) is a uniform shuffle (answer at any row, no
> learnable position bias). `TokenBuilderView` + `ChoiceView` render `displayBank`
> (the builder maps placed indices through that same array, so duplicate tokens stay
> correct). Covered by `Exercise._selfTest()` wired into the launch DEBUG block.
> **(2) Harder distractors** — all **1440** construis banks re-authored with targeted
> grammar traps (wrong conjugation / article-gender / agreement / tense,
> indic-vs-subj, qui-vs-que…) tuned per module; bank ≈ **1.6×** answer length
> (min +2 distractors): avg extra distractors **1.24 → 2.62**. `answerTokens` +
> prompts are byte-identical. Tooling `tools/distractors/{extract,apply}.py` — extract
> emits a per-drill spec, 25 subagents (one per module) author distractors, apply
> merges them back touching ONLY `tokenBank` via a style-preserving serializer (diff
> = just the changed bank lines). lint clean; build EXIT 0; app loads + self-tests
> pass. Sim taps still unautomatable → the shuffle was proven via a Python replica of
> the Swift algo, not a literal drill screenshot. NOTE: a few modules keep
> over-regularization traps (`prendu`, `etrai`, `s'elle`) — real learner-error forms
> shown on-screen; strip later if undesired. Design:
> `docs/plans/2026-06-19-token-shuffle-distractors-design.md`.
>
> **▶ NEXT (new session):**
> - (a) **Tap-through the on-device loop.** App is now INSTALLED on the iPhone (see
>   "Shipped to phone"), but the flows were never automatable here (no
>   idb/cliclick, Apple-events denied) — they are logic-self-tested +
>   screenshot-validated, not user-tapped. Verify on the phone: Apprendre drills an
>   element to 3-distinct-correct → module completes → next module unlocks →
>   Réviser resurfaces due items; "Trop facile" suspend/clean; exit bar mid-session
>   keeps progress; Settings "Exporter" shares the .sqlite; **back/forward chevrons
>   (`‹`/`›`) navigate to prior drills read-only** (restored — see DRILL BACK/FORWARD
>   NAV; real-view screenshot-verified, literal tap still TODO on-device).
> - (b) **More books** — Conversation / Intermédiaire are locked placeholders in
>   `LibraryView`; the `Book` model already supports many. Wire real content +
>   per-book selection into `BookView` (currently loads the single `ContentLoader`
>   book).
> - (c) Expose `masteryThreshold` in Settings (today a constant).
> - (d) ✅ **DONE** — `PRAGMA user_version` migration shipped (see above). Remaining:
>   fix pre-existing `first_seen_date` binding in `upsertReviewStatus` (bound from
>   `lastReviewDate`, masked by COALESCE — low impact, worth a follow-up).
> - `main` PUSHED to origin through the shuffle + distractor work (head `365f8fc`);
>   the drill back/forward nav fix (`25fd220`) went up with it. Tree clean except:
> - Uncommitted local: `project.pbxproj` carries your `DEVELOPMENT_TEAM` signing id
>   (left alone intentionally).
>
> **▶ "Trop facile" skip. ✅ MERGED.** Per-drill too-easy gate: mark it, still answer
> once — correct → suspend forever; wrong → clean records, re-markable. Per element.
>
> **STEP 1 — merge the content pipeline. ✅ DONE (2026-06-19).** Branch
> `content-pipeline` fast-forward-merged into `main` and deleted; stray
> `verbecc.log` gitignored. Not pushed.
>
> **STEP 2 — build the app-side scheduler. ✅ DONE + MERGED (2026-06-19).**
> Fast-forward-merged into `main` (not pushed); branch deleted. (a) Settings +
> daily home ("X new · Y due", `newPerDay` 20/hard-cap 5–40 + soft review target
> 200/exceedable), (b) new intake decoupled from topic completion
> (`StudyScheduler.newQueue` across the course), (c) overflow review top-up. Ladder
> `[1,3,7,16,35]` unchanged. Build-green + screenshot-validated; spec +
> code-quality + whole-branch reviews passed. Plan + details:
> `docs/plans/2026-06-19-scheduler-implementation.md`, progress §P6.
>
> **STEP 3 — sentence-composition track. ✅ DONE + MERGED (2026-06-19).**
> Fast-forward-merged into `main` (not pushed); `sentence-composition` branch
> deleted. Replaced the 4 verb-conjugation
> topics with a 4-topic **"Composer des phrases"** track (01 S-V-O · 02
> adjectives · 03 prepositions · 04 negation+questions), shared daily-life vocab,
> controlled distractors, English ease-in per drill; grammar-audited; `type` maps
> to existing enums so ZERO Swift content changes. Added `tools/lint_topics.py`
> (Codable-contract + multiset token-bank lint). Also shipped drill-UX fixes
> (pinned footer, auto-advance on correct, read-only back/forward review) + a
> scheduler refresh fix (topic-session dismiss now recomputes daily counts).
> Full end-to-end verified on sim (home, intro card, build/MC drills,
> correct/wrong, review, completion→unlock, count refresh). Design:
> `docs/plans/2026-06-19-sentence-composition-track-design.md`.
>
> **STEP 4 (optional, ask first)** — run the pipeline at scale (5 agents × 100 →
> ~500) via `tools/content_pipeline/` (`prepare_all.py` → agents on
> `agent_template.md` → `finalize_all.py`), then CURATE `out/staging/` into
> `carambole/Resources/Topics/`. NOTE: pipeline output is verb-conjugation; the
> live course is now sentence-composition — reconcile before curating.
>
> Status tracker for the pivot is below; pipeline details in its `README.md`.

SwiftUI + raw-SQLite iOS app. **Pivoted** from a vocabulary swipe-flashcard app
to a modular French grammar trainer: active-production drills (tap-to-build +
multiple choice), an unlockable topic tree, and spaced-repetition review.

## Locked design decisions

1. **Active production** — user produces answers (conjugate / fill blank / pick
   gender), not recognition swipe.
2. **Input = tap-to-build + multiple choice** — fixed token bank → exact match,
   no free-text / accent / typo handling. The bank is presented via
   `Exercise.displayBank` (deterministic per-id shuffle, never answer-order) and
   carries grammar-trap distractors sized ≈1.6× the answer length — NOT the raw
   authored order. (Also: short answers shuffle ~50/50 into typed `freeText`.)
   **HARD RULE — typing answers carry no punctuation the learner would type**
   (`. , ? ! ; : « » … " ( )`). Enforced at the chokepoint `Exercise.isTypeable`
   (a punctuated answer is barred from the typing shuffle → stays picked) and in
   `tools/lint_topics.py` (author-set `freeText` + punctuation = lint error).
   Practically: sentences served for typing have no commas or terminal marks.
3. **Topic tree + SRS mixing** — ordered unlockable topics; daily review queue
   resurfaces due drills across finished topics.
4. **Modular content** — one topic = one JSON in `Resources/Topics/`. Add grammar
   = author JSON + manifest line, zero Swift changes.
5. **Ease-in English** — learner knows words but struggles with full sentences.
   Every exercise carries an always-visible subtle English `translation` + a
   tap-to-reveal English `hint`; each topic opens with a short English intro card
   (the `ruleNote`). Light touch — present, not cluttering. (Task T-EN.)

Full design: `docs/plans/2026-06-18-french-grammar-pivot-design.md`
Implementation plan: `docs/plans/2026-06-18-french-grammar-pivot.md`
Design system ("Le Cahier"): `docs/plans/2026-06-18-design-system.md`
Schedule / SRS design (next phase): `docs/plans/2026-06-19-schedule-design.md`

## Content pipeline (dev tooling)

`tools/content_pipeline/` — generates grammar-correct conjugation exercises at
volume. Deterministic seeds (verbecc, 3360 available) + LLM sub-agents writing
prose, gated by a seed-fidelity contract so a hallucinated form can't ship. TDD,
57 tests. Run: `cd tools && python3 -m pytest content_pipeline -q`. Workflow:
`prepare_all.py` → dispatch N agents on `agent_template.md` → `finalize_all.py` →
curate `out/staging/` into `carambole/Resources/Topics/`. See its `README.md`.

**App-side scheduler — BUILT** (branch `scheduler`, 2026-06-19): daily-driven home
("X new · Y due"), `newPerDay`(20, hard cap 5–40) + soft review target(200,
exceedable), new-intake decoupled from topic completion (`StudyScheduler.newQueue`
across the course), and overflow review (`reviewQueue` tops up past the due-stack).
See `docs/plans/2026-06-19-scheduler-implementation.md`.

## Architecture

- **Models** `GrammarTopic`, `Exercise` (replace `Word`/`WordList`/`Book`);
  `ReviewStatus` keyed by `exerciseID`.
- **Persistence** `DataManager` (raw SQLite C API). Tables: `review_status`
  (one row per element — SRS schedule + mastery counters, keyed `element_id`) and
  `review_log` (`element_id, day_start` PK — one row per element per day it was
  reviewed; drives the per-day history tracker). Legacy `topics`/`exercises` were
  DROPPED by the v1 migration (content is now in-memory via `ContentLoader`).
  Schema is `user_version`-migrated (now at **2** — see `CoreDataManager.migrate()`).
- **Content** `Resources/Topics/manifest.json` + per-topic JSON; seeded by
  `TopicLoader`.
- **Study loop** one `DrillView` (`DrillEngine.check` = exact token match) serves
  both topic sessions and the daily review queue. Token chips / MC choices render
  in `Exercise.displayBank` order — a deterministic per-id shuffle so candidates
  never follow the answer order; banks carry grammar-trap distractors (≈1.6×
  answer length). See `tools/distractors/`.
- **SRS** `SRSScheduler`, intervals `[1, 3, 7, 16, 35]` days.
- **Feature folders** `Features/{TopicMap,Drill,Review,Completion}`, `Core/`.

## Build / verify

Xcode 16 `PBXFileSystemSynchronizedRootGroup` — drop `.swift` files in
`carambole/`, no `project.pbxproj` edits. No XCTest target yet; verification gate
is a clean build + `#if DEBUG` self-tests on pure-logic types.

```bash
xcodebuild build -project carambole.xcodeproj -scheme carambole \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -quiet CODE_SIGNING_ALLOWED=NO
```
Expected: `** BUILD SUCCEEDED **`.

**GOTCHA (2026-06-19): the `name:iPhone 16` destination above now FAILS** with
EXIT 70 (`Unable to find a device matching the provided destination specifier`).
The name form resolves `OS:latest` = **26.5**, but the installed `iPhone 16` sims
sit at **iOS 18.5** (the 26.5 line is iPhone 17+), so name+latest matches nothing.
Build against the explicit booted device id instead:
```bash
xcodebuild build -project carambole.xcodeproj -scheme carambole \
  -destination 'id=<sim-udid>' -quiet CODE_SIGNING_ALLOWED=NO
# find it: xcrun simctl list devices available | grep 'iPhone 16 ('
# this machine's iPhone 16 / 18.5 sim was 7C7BA0A7-C367-4CD4-AAE5-462B1F754986
```
Note `-quiet` suppresses the `** BUILD SUCCEEDED **` banner — judge by `$?` (0).

### Install on a physical device (deployed 2026-06-19)

Target: iPhone 16, iOS 26.5. Bundle `wen.carambole`. Signing team `QMV4SPL476`
(Apple ID `yiwenwang9702@gmail.com`, free/personal). Hardware UDID
`00008140-000103393A87001C`; CoreDevice id `320124DF-9F69-5DB9-94D2-054D20732476`.

One-time setup (DONE, don't repeat): Developer Mode on (Settings → Privacy &
Security → Developer Mode → restart); Apple ID signed into Xcode → Settings →
Accounts; device paired + trusted; iOS 26.5 platform/DDI downloaded; profile
auto-created via `-allowProvisioningUpdates`.

Per-build install from CLI (device must be unlocked + `devicectl list devices`
showing `connected`):
```bash
xcodebuild -project carambole.xcodeproj -scheme carambole \
  -destination 'id=00008140-000103393A87001C' \
  -allowProvisioningUpdates -derivedDataPath /tmp/cb-device build
xcrun devicectl device install app --device 320124DF-9F69-5DB9-94D2-054D20732476 \
  /tmp/cb-device/Build/Products/Debug-iphoneos/carambole.app
```
Or simply ⌘R in Xcode. Gotchas seen during first deploy, in the order they
surfaced: `Device is busy (Waiting to reconnect)` → device still preparing, retry;
`iOS 26.5 is not installed` → download the platform (`xcodebuild -downloadPlatform
iOS` or Xcode → Settings → Components); `Developer Mode disabled` → enable on phone
+ restart; `Unable to log in … rejected` / `No profiles … found` → re-sign-in the
Apple ID in Xcode → Settings → Accounts (interactive, needs 2FA — NOT doable
headless). Free profile expires after 7 days → reinstall to refresh; **user data
(`review_status`) survives reinstall** (same bundle id + `user_version` migration);
only deleting the app icon wipes it.

## Progress

Branch: `french-grammar-pivot`. Status legend: ☐ todo · ◐ in progress · ☑ done.

### P1 — Data spine
- ☑ T1 `GrammarTopic` + `Exercise` models; rekey `ReviewStatus`
- ☑ T2 SQLite tables + `DataManager` methods (topics/exercises/review_status)
- ☑ T3 `TopicLoader` + first module (`03_passe_compose_etre.json`)

### Design foundation (before any UI task)
- ☑ T-DS `DesignSystem.swift` — palette, serif typography, `PrimaryButtonStyle`,
  `ProgressRing`, `DesignGalleryView` ("Le Cahier"). Visually validated. API:
  `AppColor.{paper,ink,graphite,saffron,saffronDeep,success,error,hairline}`,
  `Font.{appPrompt,appTitle,appToken,appLabel,appExplain}`, `Spacing`,
  `PrimaryButtonStyle`, `SectionLabel`/`.uppercaseLabel()`, `ProgressRing(progress:)`.

### P2 — Drill core
- ☑ T4 `DrillEngine` (exact-token check)
- ☑ T5 `DrillView` + token/choice input + `DrillSessionView` (consumes T-DS).
  Visually validated.

### Ease-in English (before T7 content)
- ☑ T-EN `Exercise.translation`/`hint` + schema/content + DrillView surfacing +
  topic intro card ("LEÇON"). English for a beginner (decision #5). Visually
  validated — intro card + subtle translation + "Indice" reveal.

### P3 — Topic map + unlock
- ☑ T6 `TopicMapView` ("Table des matières") nav; vocabulary flow removed.
  Visually validated. Single seeder = `TopicMapViewModel`. `Font.appDisplay` added.
- ☑ T7 4-topic course: être/avoir présent → -ER présent → passé composé être →
  imparfait. English + `wordOrder` sentence-build drills. Grammar-audited (0
  errors / 21 exercises). Chain verified: only topic 01 unlocked.

### P4 — SRS / Daily Review
- ☑ T8 `SRSScheduler` (ladder [1,3,7,16,35]) + persisted `review_status`, graded
  per answer. NOTE deviation: unlock stays "finished once"; SRS drives retention
  only (not unlock gating) — keeps progression fast (decision-aligned).
- ☑ T9 Daily Review queue (`ReviewQueueView`) — reuses `DrillSessionView` (SRS
  re-grades on review), real `dueCount` on the map, editorial empty state.
  Visually validated.

**PIVOT COMPLETE** (T1–T9 + T-DS + T-EN). App: topic map → English intro →
tap-build/MC drills with translation + Indice → completion unlocks next; Daily
Review surfaces SRS-due items. Adding a topic = JSON + manifest line.

### P5 — Content scale (ongoing, no code)
- Author topic JSON modules one at a time.

### P6 — App-side scheduler (branch `scheduler`, 2026-06-19)
- ☑ S1 `StudySettings`/`StudySettingsStore` (newPerDay 5–40 hard cap, reviewTarget
  soft) + `StudyScheduler` (`courseOrdered`/`newQueue`/`reviewQueue`, self-tested)
  + `DataManager` `first_seen_date` UPSERT, `fetchSeenExerciseIDs`,
  `countNewIntroduced(on:)`, `fetchOverflowExerciseIDs(asOf:limit:)`.
- ☑ S2 Daily-driven home ("Aujourd'hui · X new · Y due"), `SettingsView` (gear →
  sheet), Apprendre = cross-course new intake (decoupled from topic completion),
  Réviser = due + overflow top-up. Topic ToC + topic-tap drilling preserved.
  Build-green + screenshot-validated; spec + code-quality + whole-branch reviews
  passed. Fast-forward-merged into `main` (not pushed); `scheduler` branch deleted.

### P7 — Sentence-composition track (branch `sentence-composition`, 2026-06-19)
- ☑ C1 4-topic **"Composer des phrases"** course replacing the verb topics:
  01 `phrases_svo` (S-V-O + article/gender) · 02 `phrases_adjectifs` (placement +
  agreement) · 03 `phrases_prepositions` (à/de/dans/avec + à+le→au) · 04
  `phrases_negation_questions` (ne…pas / n' / est-ce que / inversion). Shared
  daily-life vocab, controlled distractors, English `translation`+`hint` per
  drill, grammar-audited. `type` reuses existing enums → no Swift content changes.
- ☑ C2 `tools/lint_topics.py` — validates topic JSON vs the Codable contract
  (types, multiset token-bank subset, MC shape, required English ease-in).
- ☑ C3 Drill UX: pinned Vérifier/Suivant footer (no longer hidden by a tall
  banner); correct → green flash + auto-advance; wrong → stays with answer +
  manual Suivant; read-only back/forward **review** of answered drills (session
  holds submitted tokens + result). `DrillView`/`DrillSessionView`.
- ☑ C4 Scheduler fix: topic-session `fullScreenCover` now refreshes daily counts
  on dismiss (was the only cover missing it → stale "X new"). `TopicMapView`.
- Verified end-to-end on sim; lint + build green. Fast-forward-merged into
  `main` (not pushed); `sentence-composition` branch deleted.

### P8 — Progress + history tracker (branch `progress-history-tracker`, 2026-06-19)
- ☑ H1 Data: `review_log(element_id, day_start)` table + `user_version` **2**
  migration (additive block, backfilled once from `last_review_date`);
  `upsertReviewStatus` logs a row per graded answer; reads
  `fetchElementIDsFirstSeen/ReviewedLogged(on:)`, `fetchFirstSeenByElement()`,
  `fetchReviewLog()`. `CoreDataManager`.
- ☑ H2 `StudySettings.reviewRoundSize` {10,20,50} default 20 (snaps to nearest),
  persisted in `StudySettingsStore`; self-tested.
- ☑ H3 `HistoryView` + `HistoryViewModel` (pure-static `activities(...)`,
  self-tested) — per-day learned/reviewed page, two-tone bars, footnote, empty
  state. `Features/History/`.
- ☑ H4 Book home: daily progress line `N appris · M révisés` (→ pushes
  `HistoryView`), inline `10·20·50` "Révision par tour" selector,
  `reviewSessionElements().prefix(reviewRoundSize)` round slice. `BookView`/
  `BookViewModel`. Per-task spec + code-quality + whole-branch reviews APPROVE;
  build EXIT 0; both screens screenshot-validated. Fast-forward-merged into `main`
  + **PUSHED** (head `bf0ed87`); branch deleted. Design:
  `docs/plans/2026-06-19-progress-history-tracker-design.md`.

### P9 — Token-bank shuffle + harder construis distractors (commit `365f8fc`, 2026-06-19)
- ☑ S1 `Exercise.displayBank` — deterministic per-id Fisher-Yates (`splitmix64` off
  `Exercise.seed64`, the FNV-1a hash now shared with `typeBit`); stable across
  re-renders + review-nav; guards so it never reproduces authored order / answer
  order (multi-token) and MC is uniform. `TokenBuilderView` + `ChoiceView` render it.
  `Exercise._selfTest()` wired into the launch DEBUG block.
- ☑ S2 All 1440 `buildFromTokens` banks re-authored with targeted grammar-trap
  distractors, bank ≈1.6× answer length (avg extra distractors 1.24→2.62);
  `answerTokens`/prompts unchanged. Authored by 25 subagents (1/module) gated by
  `tools/distractors/{extract,apply}.py` (style-preserving merge, touches only
  `tokenBank`) + `lint_topics.py`. Committed + **PUSHED** to `main` (head `365f8fc`).
  Design: `docs/plans/2026-06-19-token-shuffle-distractors-design.md`.

### P10 — Progress bar + time-to-finish estimate (branch `progress-tracking-estimate`, head `c2eeccc`, 2026-06-21)
- ☑ T1 `mastery_log(element_id PK, day_start)` + `user_version` **3** migration
  (additive; `backfillMasteryLog()` once from `last_review_date`, forward-only);
  write-hooks in `upsertReviewStatus` + `setSuspended`; `fetchMasteryLog()`.
  `CoreDataManager`. Commit `90361d8`.
- ☑ T2 `ProgressEstimate` pure value type (`Core/ProgressEstimate.swift`): bar
  segments + `percentComplete` + trailing-14d mastery-pace `daysRemaining` /
  `projectedFinish`, early-state gate (≥3 days history AND ≥3 window masteries),
  deterministic (injected `today`/`calendar`), 8-case `_selfTest()` at launch.
  Commit `594f80a`.
- ☑ T3 `BookView`/`BookViewModel`: 3-segment completion bar (maîtrisé/en cours/à
  venir) + % + 3-state estimate line, French formatting in the view; daily zone
  fix (to-do counts onto `Apprendre · N`/`Réviser · M` buttons, one `fait :`
  done-line). `mastered`/segments/% from the LIVE satisfied gate; `mastery_log`
  feeds ONLY pace. Commit `c2eeccc`.
- Per-task spec + code-quality + final whole-branch reviews all APPROVE/SHIP;
  build EXIT 0; launch self-tests pass; populated bar screenshot-validated vs the
  real `BookView` (CB_SCREEN harness, reverted). Design:
  `docs/plans/2026-06-21-progress-tracking-design.md`. NOT yet merged/pushed.

## Authoring a new grammar module

1. Add `carambole/Resources/Topics/NN_slug.json` — one `GrammarTopic` + 5–6
   `Exercise`s, mixed `buildFromTokens` / `multipleChoice`. JSON keys must match
   the Codable property names exactly (`sortOrder`, `prerequisiteID`, `topicID`,
   `answerTokens`, `tokenBank`, `inputMode`). Set `sortOrder` + `prerequisiteID`
   to place it in the unlock chain.
2. Add the filename to `carambole/Resources/Topics/manifest.json` →
   `{ "topicFiles": [ ... ] }`.
3. No Swift changes. `TopicLoader.seedIfNeeded()` picks it up.

**Token banks:** authored bank ORDER does not matter — `Exercise.displayBank`
shuffles it deterministically at render time, so don't hand-shuffle. Each
`buildFromTokens` bank should be `answerTokens` + grammar-trap distractors sized
≈1.6× the answer length (min +2). To bulk-add distractors to existing modules use
`tools/distractors/extract.py` (per-drill spec) → author distractors → `apply.py`
(merges back, touching only `tokenBank`; validates count + multiset subset).

**Typing answers — no punctuation (HARD RULE):** short answers (≤2 tokens) may be
shuffled into typed `freeText`, and the learner is never asked to type punctuation
(`. , ? ! ; : « » … " ( )`). Don't put terminal marks or commas in an answer you
want typed; an answer carrying them is auto-demoted to its picked mode by
`Exercise.isTypeable`, and an explicit `freeText` answer with punctuation fails
lint. Build/MC answers may keep punctuation as their own chips (it's tapped, not
typed).

**Rendered-surface grammar (HARD RULE):** audit every drill by reading the
RENDERED sentence — the blank filled with the answer — as final French, never as
template slots. The blank hides agreement bugs: `Je ___` + `habitais` shipped as
the ungrammatical « Je habitais » (h muet → « J'habitais », found 2026-08-03)
because auditors read prompt and answer separately. Prompts must pre-elide
(`J'___`, `N'___ pas`) when every bank candidate starts with a vowel or mute h.
`tools/lint_topics.py` enforces the deterministic classes on the rendered
surface — elision (je/ne/le/la/de/que/… + vowel or mute h, with an h-aspiré
whitelist `H_ASPIRE_PREFIXES` to extend when new aspirated-h words enter the
content), mandatory contractions (`à/de + le/les` → au/aux/du/des), and
`si il(s)` → `s'il(s)`. `tools/deep_lint.py` (Grammalecte, vendored offline in
`tools/vendor/`, ~1.5 s corpus) catches the open classes — agreement,
conjugation, missing elision generally — on the same rendered surface; grammar
flags are errors gated by the reviewed known-noise list
`tools/deep_lint_suppressions.json` (read the sentence aloud before ever adding
an entry), unknown-word spell flags are warnings (ease-in English + stem
fragments). Both linters run automatically via the project PostToolUse hook
(`.claude/settings.json` at the workspace root) after any edit to
`Resources/Topics/*.json`. Run them manually before handing off a module anyway.
Related gotcha this net caught: `Exercise.spokenFrench` / `tools/tts/extract.py`
glue elisions with a whitelisted-prefix regex (`ELISION_GLUE`, byte-identical in
both) — a blanket `"' " → "'"` once swallowed the closing quote of
straight-quoted mentions and corrupted ~11 TTS sentences.

**Prompt self-sufficiency (HARD RULE):** every drill must be answerable from the
prompt (+ visible cue) alone — never only from the collapsed hint or the
post-answer explanation. The classic violation: English "it" carries no gender,
so `Construis : « We sell it. »` with answer `Nous la vendons` and a bank
offering both `le` and `la` lets the learner build a grammatically valid
sentence that gets marked wrong. When the answer hinges on the gender/number of
an unstated referent, put the referent in the prompt as a French-style setup
question mirroring the fillBlank pattern — e.g.
`Construis la réponse : « Vous vendez la voiture ? » — « We sell it. »`.
Enforced (heuristically, clause-final object "it" only) by
`gender_ambiguity_errors` in `tools/lint_topics.py`; the same principle applies
beyond that check — number, formality (tu/vous), and person must also be
recoverable from the prompt, not hidden in the hint. (Found 2026-07-12: 5 items
in `17_pronoms_objets.json` shipped ambiguous; fixed with setup questions.)

**Gotchas (verified in T3):**
- Xcode 16 `PBXFileSystemSynchronizedRootGroup` **flattens** `Resources/Topics/*`
  to the bundle ROOT at build time (no `Topics/` subdir in the `.app`).
  `TopicLoader.bundleURL` tries the subdirectory then falls back to root — works
  either way; filenames must stay unique.
- `seedIfNeeded()` only seeds when `topicCount() == 0`. To re-seed after editing
  content during dev: `xcrun simctl uninstall booted wen.carambole` then reinstall
  (clears the SQLite DB). Bundle id = `wen.carambole`.

## Known tech debt
- **SQLite schema is now `user_version`-migrated** (RESOLVED — was the top debt
  here). `CoreDataManager.migrate()` runs ordered `if version < N` blocks; current
  version = **3** (v1 = element-keyed `review_status` baseline; v2 = `review_log`
  table + backfill; v3 = `mastery_log` table + backfill). RULES for any future
  change: add a NEW block, additive only
  (`ALTER TABLE … ADD COLUMN` / `CREATE TABLE IF NOT EXISTS`), bump `setUserVersion`;
  **never edit an earlier block, never DROP a user-data table.** The old
  exercise-keyed `review_status` and the `topics`/`exercises` content tables are
  dropped by v1; content is in-memory now.
- **`first_seen_date` binding quirk in `upsertReviewStatus`** (low impact, open —
  NEXT (d)). The bind is sourced from `lastReviewDate`, not a true first-seen
  timestamp; masked by `COALESCE` so the FIRST write still stamps ≈ introduction
  time. Fine for the per-day "learned" counts (first write ≈ today) but worth a
  clean follow-up.
- **Reviewed-history before the v2 migration is forward-only.** The `review_log`
  backfill seeds only the *latest* review day per element (all `last_review_date`
  retained). Learned-per-day is fully historical (`first_seen_date` is permanent);
  reviewed-per-day for past days fills in going forward. Documented in-app
  (`HistoryView` footnote) + in `backfillReviewLog()`.
- **Mastery-per-day before the v3 migration is forward-only.** The `mastery_log`
  backfill stamps each currently-satisfied element at its `last_review_date` day
  (one row/element), so the trailing-window *pace* estimate is approximate for
  pre-migration days; current `% complete`/segments are always exact (from the live
  `review_status` gate, not the log). `mastery_log` is a HISTORICAL event log —
  `clearReviewStatus` ("Trop facile → wrong → clean records") intentionally does NOT
  retract a logged mastery (mirrors `review_log`). Documented in the v3 block +
  `backfillMasteryLog()`.

## Execution

Driven by `superpowers:subagent-driven-development`: fresh implementer subagent
per task → spec-compliance review → code-quality review → mark done. Update the
checkboxes above as tasks land.

---
> Source: [yiwenwang47/carambole](https://github.com/yiwenwang47/carambole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
