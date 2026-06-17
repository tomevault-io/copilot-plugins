## personal-hygiene

> This file is read automatically at the start of every Claude Code session in this repository.

# CLAUDE.md — instructions for Claude Code

This file is read automatically at the start of every Claude Code session in this repository.

---

## 1. MANDATORY SESSION STARTUP — read these first

In order, before responding to any user request:

1. `~/.claude/projects/-Volumes-USB1TBWD--develop--repos-personal-hygiene/memory/MEMORY.md` → index of memory files
2. `~/.claude/projects/-Volumes-USB1TBWD--develop--repos-personal-hygiene/memory/session_handoff.md` → what happened last (when present)
3. `~/.claude/projects/-Volumes-USB1TBWD--develop--repos-personal-hygiene/memory/project_status.md` → roadmap progress (when present)
4. `~/.claude/projects/-Volumes-USB1TBWD--develop--repos-personal-hygiene/memory/feedback_working_style.md` → how this user works
5. `PRD.md` → product requirements
6. `ROADMAP.md` → current phase + acceptance criteria
7. `LESSONS.md` → numbered lessons (never repeat past mistakes)

---

## 2. PROJECT IDENTITY

**What:** Native iOS + watchOS personal scheduling app with military-precision time blocks. Single-user. Reliable medication reminders. Holistic vacation module. CloudKit sync.

**What it is NOT:** not multi-user, not a calendar replacement, not a habit gamifier, not analytics-heavy, not for sale (yet), no backend.

**Privacy stance:** All data stays in the user's Apple ecosystem. No telemetry. No third-party SDKs.

**Languages:** UI in ES + EN + FR. Documentation in English. Conversation in user's preferred language (Spanish currently).

---

## 3. TECH STACK (verified 2026-04-25)

| Layer | Choice | Version |
|---|---|---|
| Language | Swift | 6.0+ |
| UI | SwiftUI | iOS 18+, watchOS 11+ |
| Persistence | SwiftData (preferred) or CoreData + CloudKit | iOS 18+ |
| Sync | CloudKit Private Database | — |
| Health | HealthKit (Medications + Sleep) | iOS 18+ |
| Calendar | EventKit | — |
| Contacts | Contacts framework | — |
| OCR | VisionKit + Vision | — |
| AI | Apple Intelligence Foundation Models | iOS 18.1+ |
| Weather | WeatherKit | — |
| Marine | Open-Meteo Marine API | — |
| Currency | Frankfurter API | — |
| CI | GitHub Actions (`macos-latest`) | — |
| Lint | SwiftLint + swift-format | latest |

---

## 4. ARCHITECTURE PATTERNS

- **One type per file.** File name = type name.
- **MVVM with SwiftUI.** Views observe `@Observable` view models. No business logic in views.
- **Shared module** for code used by both iOS + watchOS targets (in `App/Shared/`).
- **Feature-folder layout** (`App/PersonalHygiene/Features/<Module>/`), not type-folder layout.
- **Async/await first.** No completion handlers in new code.
- **No third-party dependencies** unless absolutely necessary — prefer Apple frameworks.
- **i18n: every UI string** goes through `Localizable.xcstrings`. Hardcoded user-facing strings = build error (SwiftLint custom rule TBD).

---

## 5. TECHNICAL LESSONS — quick reference

See [LESSONS.md](LESSONS.md) for full text. Quick table:

| L0NN | Rule | Guard |
|---|---|---|
| L001 | Keep `ModelContainer` alive for the lifetime of any `ModelContext` built from it; never drop it from a helper | `SwiftDataRoutineRepositoryTests` exercises insert + cascade-delete + relationship-append (suite crashes if regressed) |
| L002 | A notification-identifier parser must recognize every module's prefix; model the prefix as an enum so adding a kind without updating the parser is a compile error | `BlockNotificationIdentifierTests.test_parse_recognizesAllKnownPrefixes` iterates `NotificationKind.allCases` and round-trips each |
| L003 | Files in `App/Shared/` using iOS-only APIs (UIGraphicsPDFRenderer, UIActivityViewController, etc.) must be `#if canImport(UIKit) && !os(watchOS)`-guarded — the watch target compiles `Shared/` too | `./scripts/deploy-watch.sh --no-install` builds the watch scheme; run after touching `Shared/` |
| L004 | Tab-root views inside iOS 18 TabView "More" overflow must NOT add their own `NavigationStack` (More provides one); doing so produces two stacked back chevrons on every push | Manual on-device check: open any overflowed tab → push into a child → exactly one back arrow |
| L005 | Test process crashes (signal-trap / "Restarting after unexpected exit") must NOT be filtered as the LLDB glitch in `check-tests.sh` — a real crash with no failed-test-method line silently passed CI | `scripts/check-tests.sh` counts process-crash lines separately; treats them as real failures regardless of exit code |
| L006 | `Text(LocalizedStringKey("foo.\(rawValue)"))` looks up `"foo.%@"`, not the runtime key — UI shows raw keys (`category.work` etc.). Use `Text(localizedKey: "foo.\(rawValue)")` for discrete-suffix lookups; xcstrings keys with placeholders must include the matching `%@` / `%lld` suffix | `BundleLocalizationLookupTests` resolves every enum-driven discrete-suffix key + every format-string key through `Bundle.main`; round-19 SwiftLint custom rule `dynamic_localized_key` rejects new `LocalizedStringKey("...\(...)")` at compile time; backup `scripts/check-localized-key-usage.py` and `scripts/check-xcstrings-format-consistency.py` static scans run alongside |
| L007 | **Inverse of L004**: a tab-root MISCLASSIFIED as More-overflow loses its system-provided `NavigationStack`. Without one, `.navigationTitle` and `.toolbar` don't render — view chrome silently disappears. Update `scripts/check-tabroots.py` `TAB_ROOTS` whenever the iOS 18 TabView grows or reorders | Manual `QA_MANUAL.md` checklist after every TabView reorder: open each direct tab (first 4) → confirm title + ≥1 toolbar item visible. Future: XCUITest smoke per direct tab |
| L008 | Prefer SwiftData `@Query` over repository-cached VM properties for **cross-tab reactive state**. `@Observable var activeTemplate` populated by `repository.fetch(...)` goes stale across iOS 18 TabView switches; `@Query` observes the modelContext directly and auto-refreshes. Repository pattern stays fine for one-shot writes | None automated yet. Future: XCUITest smoke "create entity in tab A → switch to tab B → assert reactive re-render <1s" |
| L009 | Local Xcode and CI's `macos-latest` runner produce **different Swift 6 verdicts** for pre-Swift-6 Apple SDK call sites and SwiftUI ViewBuilder closures: local emits warnings, CI escalates to `##[error]`. Local-test-pass is necessary but **not sufficient** to declare CI green. Defensive `@preconcurrency import` on Apple SDKs that override delegate methods with closure parameters; for full migration see Batch Q | `gh run list --limit 3` mandatory after lint-cleanup rounds before declaring CI green; `feedback_repo_quirks.md` documents the pattern; future `scripts/check-strict-concurrency.sh` runs `xcodebuild build SWIFT_STRICT_CONCURRENCY=complete` |
| L010 | Repo lives on USB drive with macOS resource-fork siblings (`._*.swift`); naive `find ... -name "*.swift" \| wc -l` overcounts ~2× (e.g. 326 vs real 163). Audits that compare against memory's counts produce false drift. Always pass `-not -name "._*"` when counting Swift / md / json files for an audit | `scripts/check-counts.sh` exposes `count_swift PATH` and `count_glob PATH PATTERN` helpers that always exclude `._*`; CI pre-flight calls it to fail-fast if the inflation pattern recurs in any new audit script |
| L011 | Swift 6 strict-mode migration after the round-29 prep work collapses to 4 distinct fix-classes: (1) drop redundant `@preconcurrency` from conformance lines whose framework is already preconcurrency-imported, (2) capture a `Sendable` value type (Bool/String/Int) into a `let` BEFORE a SwiftUI ViewBuilder closure that reads MainActor state — non-Sendable types like `LocalizedStringKey` get rebuilt INSIDE the closure, (3) `@preconcurrency import` on every Apple-SDK with pre-Swift-6 delegate methods (already done round 29), (4) `@MainActor` on View structs + `MainActor.assumeIsolated` on synchronous UIKit dismiss sites (already done round 29) | `scripts/check-strict-concurrency.sh --files` reports 0/0 once the four classes are addressed; flip `SWIFT_VERSION: "5"` → `"6.0"` in `App/project.yml`, regenerate xcodeproj, run `check-tests.sh`. Round 36's full migration was a single round once the inventory was accurate (round 34 fixed the script's PROJECT path bug) |
| L012 | Strict-concurrency inventory regex must cover EVERY Swift 6 diagnostic shape, not just the keyword bag (`Sendable\|actor-isolated\|@MainActor\|...`). Round 36 reported 0/0 then CI failed on `sending value of non-Sendable type` (unit tests) + `actor-isolated initializer/instance method/property... in a synchronous nonisolated context` (UI tests) — phrases the regex didn't include. **Always extend the regex in the same commit that fixes the new diagnostic shape**, otherwise the next round hits the same blind spot. | Extended `CONCURRENCY_RE` adds `sending value of\|risks causing data races\|in a synchronous nonisolated context\|cannot be (referenced\|mutated\|sent) from\|across actor boundary`; cross-check warning total via `grep -cE "(error\|warning):"` and surface unmatched diagnostics. No automated test possible — the regex IS the test |
| L013 | Wiring `CODE_SIGN_ENTITLEMENTS` to a `.entitlements` file requesting iCloud Container + App Group BEFORE the matching App ID + container exist on developer.apple.com produces `SwiftDataError._Error.loadIssueModelContainer` for **every** test that builds a `ModelContainer` (even in-memory). SwiftData reads bundle entitlements at init and rejects setup when declared identifiers can't be validated. **Sequence:** portal resources → team switch → entitlements wiring (per `docs/round26-activation-checklist.md`). Skipping straight to wiring is the failure mode | Process guard: `App/project.yml` carries inline `CODE_SIGN_ENTITLEMENTS deferred — see L013` comments above each commented-out wiring line; load-bearing until portal step done. Rollback recipe: comment out the 4 wiring lines + `xcodegen generate` + tests green. No automated test possible (failure only manifests when portal records don't exist). Future: `scripts/check-entitlements-portal-state.sh` via App Store Connect API |

---

## 6. DEVELOPMENT WORKFLOW

### Branches
- `main` — always green.
- `feat/*`, `fix/*`, `chore/*`, `docs/*`, `ci/*`.

### Commits
[Conventional Commits](https://www.conventionalcommits.org/). Scopes match module names (`routine`, `medication`, `sleep`, `vacation`, `watch`, `prd`, `ci`, etc.).

### QA-mandatory protocol
**Every bug fix and every new feature MUST update tests in the same commit.** No exceptions. See [CONTRIBUTING.md § QA-mandatory rule](CONTRIBUTING.md).

### Pre-commit checklist
```
[ ] Tests added / updated for this change?
[ ] QA_MANUAL.md updated with [T-XXX] case?
[ ] ./scripts/check-tests.sh — green?
[ ] ./scripts/lint.sh — green?
[ ] If new class of bug: LNNN entry + guard test in LESSONS.md?
[ ] Updated ROADMAP.md / PRD.md if scope changed?
[ ] memory/session_handoff.md updated if shipping work?
```

### Push policy
**Default: do NOT push to `origin/main` without explicit user authorization.** Feature branches may push freely.

---

## 7. QA & TESTING STRATEGY

### Test pyramid

| Layer | Tool | Target |
|---|---|---|
| Unit | XCTest | Pure functions, view models, services |
| Integration | XCTest + in-memory CloudKit container | Persistence + sync flows |
| UI | XCUITest | Critical user flows only |
| Static scan | shell script under `scripts/check-*.sh` | Cross-cutting bug classes (L0NN guards) |

### What NOT to do
- ❌ Don't mock HealthKit at integration boundaries — use a fake real store with seeded data.
- ❌ Don't use `XCTSkip` to silently disable broken tests — fix or delete them.
- ❌ Don't auto-commit AI-generated tests without human review (hollow assertions risk).

---

## 8. WHAT'S BUILT

| Phase | Status | Acceptance |
|---|---|---|
| 0 — Bootstrap | ✅ Done | Repo + tooling ready · Xcode project · CI green |
| 1 — MVP daily routine | 🟡 ~99.9% | M1+M2+M3+M4 code-complete; rounds 12-25 polish layer. Round 27 = WS-A AI itinerary wizard + WS-B birthdays/important-days on Today + Settings IA collapse + autocomplete + map + home-location auto-detect + advisory reorder + chip-row redesign. Round 28 = lint cleanup (65 → 0 errors). Round 29 = CI fix (Swift 6 sendability) + Wizard v2 (persist + 3 deep-links) + App Store prep docs. Round 30 = drift cleanup + L009 + L010 promoted + `check-counts.sh` guard. Round 31 = rate-limit diagnostics (NetworkActivityCounter `Outcome` enum + Marine + Frankfurter HTTP-status partition + Diagnostics breakdown row) + xcstrings de-dup (3 dup `settings.theme.*` keys removed, constant 997 → 994) + `check-counts.sh` into CI hygiene + `check-strict-concurrency.sh` Batch Q preview. Rounds 32 + 33 = K01 closure (all 3 `swiftlint:disable` paragraphs retired across TripDetailView + TodayView + BackupService; `.swiftlint.yml` file_length 500 → 600 + function_body_length 50 → 80, hard errors unchanged). Round 34 = L009 formalized in tooling (`scripts/check-ci.sh` post-push verifier + `check-tests.sh` reminder) + `check-strict-concurrency.sh` PROJECT path bug fix + first successful Batch Q inventory (0 errors, 12 warnings across 3 files). Round 35 = Trip detail IA redesign (kid-friendly): 18 sections → ~12 via 2 new combined sections (`progressSection` rolls Completion+NextMilestone, `destinationInfoSection` collapses 4 single-row nav links + wizard); 3 round-12 section structs deleted; CO₂ kg/lb picker moved to Settings → Home & Travel (was already `@AppStorage`-global); 10 trip-section xcstrings localizations updated to kid-friendly EN/ES/FR (no key renames) + 6 new keys (994 → 1000). Round 36 = **Batch Q** Swift 6 strict-mode migration: walked 12 warnings → 0 across 3 files (4 fix-classes: redundant `@preconcurrency` on conformance × 2; capture Bool into `let` before SwiftUI ViewBuilder closure for MainActor-state read), flipped `SWIFT_VERSION` 5 → 6.0 in all 3 project.yml declarations; **Lessons 10 → 11** (L011 added). Round 37 = partial Swift 6 reach (host + UI tests on 6.0; unit tests retreated to 5 after `xcodebuild build-for-testing` revealed ~8+ unit test files mutating `@MainActor`-isolated state from nonisolated `setUp`/`func test*` context — mass `@MainActor` annotation deferred to round 38). Round 37 deliverables: L012 (strict-concurrency regex blind-spots; extended `CONCURRENCY_RE` + cross-check), `@preconcurrency import XCTest` blanket on 207 test files, `@MainActor` on `OnboardingUITests`, `scripts/check-strict-concurrency.sh` wired into CI as blocking job. **Lessons 11 → 12**. Round 38a = Apple Dev Program ACTIVATED (Team `XC79TD476V`); Phase 4 unblocked; tried entitlements wiring (CODE_SIGN_ENTITLEMENTS in 4 targets) → 13 SwiftDataError test failures → rolled back; **L013 captured** ("entitlements ↔ portal sequence — wire AFTER developer.apple.com resources exist, not before"); dependabot Swift block disabled; repo flipped PRIVATE → PUBLIC to bypass GitHub Actions billing on macos-latest. **Lessons 12 → 13**. Round 38b = unit-tests target SWIFT_VERSION 5 → 6.0 (closes round-37c retreat); 18 warnings → 0 across 8 test files via 2 fix-classes (delete defensive `tearDown` cleanup × 6 sites; convert sync setUp/tearDown to `async throws` × 2 sites with real side effects). `scripts/check-strict-concurrency.sh --with-tests` flag added — runs `build-for-testing` instead of `build`, covering test targets and closing the L012 blind spot. **Entire repo now on SWIFT_VERSION=6.0 end-to-end**. Bonus: `#file` → `#filePath` fix in `BlockNotificationIdentifierRoundTripTests:94`. Remaining: real-device validation + portal Step 1 (App IDs + App Group + iCloud Container) + entitlements re-wire (post-portal) + first TestFlight build. |
| 2 — Apple Watch | ✅ ~99% | Round 19 added complication line-3, theme/pause mirrors, snooze-30 + skip-dose actions. Round 21 added Hydration glance + Mood quick-log + complication pause badge + mark-done undo capsule. Round 22 added hydration goal proportion, pending count + clear, complication mood emoji, settings mood week strip, swipe-back haptic + iPhone-side reconciler. Round 23 added mood-streak chip, custom hydration stepper, pause-from-watch buttons, complication theme tint, swipe-up skip-rest-of-day. Round 24 added BlockDetail snooze menu (5/10/15), settings reset-pending-taps, complication line-3 day-completion %. Watch deployed live to round-24. |
| 3 — Secondary modules | ✅ Done | M5 hydration (+ watch reconciler), M6 housekeeping (+ streak + auto-snooze helper + completion log + banner), M7 birthdays (+ relationship/idea stores + gift CSV exporter UI + global lead-time stepper UI), M8 deep focus (+ right-now + conflict + ad-hoc toggle + auto-mirror mute to watch). |
| 4 — TestFlight beta | 🔒 Blocked | Needs paid Apple Developer Program ($99/yr). |
| 5 — Vacation module | ✅ Feature-complete | M9 closed: trips + milestones + reminders + scanner + AI itinerary + marine + currency (+ CSV table copy) + 5-source advisory + PDF + notes (Markdown + weather forecast template) + archive + cost log + carbon estimate + 30-day footprint summary + emergency contacts + monthly expense summary + duplicate-with-shifted-dates + duplicate-to-next-year + milestone-defaults bundle + WeatherKit scaffolding (entitlement-gated bridge, 6h cache, itinerary forecast chip). Real-trip validation pending. |
| 6 — App Store | ⬜ | Submission + listing in 3 languages. |

> Source of truth for phase progress — must match `memory/project_status.md` and `ROADMAP.md § Phases`. Last synced: round 38b.

---

## 9. DEV ENVIRONMENT COMMANDS

| User says | Action |
|---|---|
| `bootstrap` | Run `./scripts/bootstrap.sh` (install SwiftLint + swift-format) |
| `lint` | Run `./scripts/lint.sh` |
| `format` | Run `./scripts/format.sh` |
| `run all tests` / `check tests` | Run `./scripts/check-tests.sh` |
| `commit` (alone) | Confirm scope, commit, do NOT push unless asked |
| `push` | Push current branch to upstream (NEVER `main` without explicit OK) |
| `continua` | Read `memory/session_handoff.md` next-step and continue |

---

## 10. `ALL OK?` — UNIFIED SESSION PULSE (expanded round 25 / session 23)

When the user says `ALL OK?` / `pulse?` / `que tal va?` / `estado?`, run a full system audit. **Verify EVERYTHING.** No skipping. The user expects this to surface bugs, incoherencies, and a forward-looking 100-task backlog every time.

### A. Per-platform completion %

Compute the % of *acceptance criteria delivered* per surface, not lines of code. Cite source: `ROADMAP.md`, `memory/project_status.md`, what's actually built vs what's blocked.

| Surface | % | Source-of-truth |
|---|---|---|
| iPhone (iOS) | x% | Phase 1+3+5 acceptance |
| Apple Watch (watchOS) | x% | Phase 2 acceptance |
| Backend | x% | None planned (CloudKit private DB only — see PRD §2) |
| Web | x% | Phase 7d (future, not started) |
| Android | x% | Phase 7c (future, not started) |
| macOS | x% | Phase 7b (future, not started) |
| TestFlight beta | x% | Phase 4 acceptance |
| App Store listing | x% | Phase 6 acceptance |

If a surface is *deliberately not in scope*, mark **n/a** and link to the PRD line that excludes it.

### B. Drift + incoherencies + bugs check

- **Git state:** branch, ahead/behind, uncommitted, untracked.
- **Memory files:** `memory/MEMORY.md` index, `memory/session_handoff.md` recency, `memory/project_status.md` recency. Flag anything >7 days stale.
- **Doc parity:** `ROADMAP.md` phase line == `CLAUDE.md §8` line == `memory/project_status.md`. Three-way sync.
- **Test + lint state:** last `./scripts/check-tests.sh` exit; counts; process-crash count.
- **i18n parity:** `LocalizationKeyCount.total` matches `grep -c '"extractionState"' Localizable.xcstrings`.
- **Lessons:** any new class-of-bug observed without an L0NN entry?
- **Cross-code consistency:**
  - `XC79TD476V` (or current `DEVELOPMENT_TEAM`) — Personal vs paid?
  - `CommitSHA.txt` on device == HEAD?
  - `.tint(.tint)`, `Menu`, or other watchOS-incompatible API surfaces in `Shared/`?
  - `LocalizationKeyCount.total` constant updated since last xcstrings change?
- **Deferred items:** anything in `session_handoff.md` "next step" that didn't ship; carry forward.
- **Stale TODO/FIXME/XXX comments** in code.
- **Apple Developer Program gate:** which features are still gated? List explicitly.

### C. 100 forward tasks — classified + batched

Produce a numbered list of the next **≥100 tasks** classified by:
- **Surface** (iPhone / Watch / Backend / Web / Android / macOS / DevOps / Docs / QA)
- **Batch** (groups of tasks deliverable end-to-end at 100% with NO inter-batch dependencies — explicit "depends on Batch X" notes for sequencing)

Each task = one line: `[ID] [Surface] [Batch] short description`. Include an **estimated effort** when relevant (S/M/L). When a task is gated by Apple (entitlement, review), mark it 🔒.

Batch organization rules:
- A batch must be *atomically deliverable* — if you ship the batch, all its tasks land together with no half-finished state.
- A batch must explicitly state its dependencies (e.g., "Batch C depends on Batch B's Apple Developer Program activation").
- Each batch should fit in **one round** (≤40 slices) when possible.

### D. "Anything else?" — proactive flags

Before closing, surface anything observed during the audit that doesn't fit cleanly above:
- Risk: external dependencies (Apple review queues, weather API rate limits, Frankfurter availability).
- Tech debt: places where shortcuts were taken that should be paid back.
- L007+ candidates: bug classes that haven't earned an L-entry yet but probably should.
- Memory gaps: things that should be in `memory/` but aren't.
- Privacy / security: any change that touched user data flow worth reviewing.

### E. Action menu

Numbered options the user can pick (each a one-liner). Include "do nothing — just commit", "run round N", "validate on device", "deploy", etc.

### Subset triggers

- `no hace falta nada?` — runs only §B (drift half).
- `pulse?` (short) — §A + §B + §E only (skip 100-task list).
- `ALL OK ?` (full) — A through E, complete.

---

## 11. SESSION END CHECKLIST

When wrapping up:
1. Update `memory/session_handoff.md` — what was done, git state, next step.
2. Update `memory/project_status.md` if any phase status changed.
3. Update `LESSONS.md` if a new technical mistake was made + fixed.
4. Update `ROADMAP.md` if a phase advanced.
5. Verify `CHANGELOG.md` reflects any user-visible change.

---

## 12. DOCUMENTATION FILES

| File | Purpose |
|---|---|
| [README.md](README.md) | Project overview, status, getting started |
| [PRD.md](PRD.md) | Product requirements |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Technical architecture |
| [ROADMAP.md](ROADMAP.md) | Phase tracker |
| [LESSONS.md](LESSONS.md) | Numbered lessons |
| [QA_MANUAL.md](QA_MANUAL.md) | Manual test checklist |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Dev workflow |
| [SECURITY.md](SECURITY.md) | Vuln disclosure policy |
| [CHANGELOG.md](CHANGELOG.md) | Notable changes |

---
> Source: [tandori46001/personal-hygiene](https://github.com/tandori46001/personal-hygiene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
