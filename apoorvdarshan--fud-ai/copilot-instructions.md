## fud-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Fud AI is an open-source iOS calorie tracker (SwiftUI, iOS 17.6+). Snap/speak/type a meal, an AI provider returns nutrition JSON, the user reviews it, and it lands in `FoodStore` + Apple Health. There's also a "Coach" tab — multi-turn AI chat that sees the user's full profile, weight history, and food log and answers questions like "what's my expected weight in 30 days?". Bring-your-own-key model; all data is local. No subscriptions, no sign-in, no cloud sync.

## Build, Install, Launch

The app is tested on Apoorv's physical iPhone (iPhone 16, device ID `E2095CDC-E117-527C-818A-9F741A145103`). After every change run all three commands. The Release config is intentional — it matches what users actually see.

```bash
# Build
xcodebuild -scheme calorietracker -destination 'id=E2095CDC-E117-527C-818A-9F741A145103' build

# Install
xcrun devicectl device install app --device E2095CDC-E117-527C-818A-9F741A145103 \
  ~/Library/Developer/Xcode/DerivedData/calorietracker-gyjqfuacfxocddfrskbcdsbwqhxa/Build/Products/Release-iphoneos/calorietracker.app

# Launch
xcrun devicectl device process launch --device E2095CDC-E117-527C-818A-9F741A145103 com.apoorvdarshan.calorietracker

# Pass --reset-onboarding to test the onboarding flow:
xcrun devicectl device process launch --device E2095CDC-E117-527C-818A-9F741A145103 com.apoorvdarshan.calorietracker -- --reset-onboarding
```

## Tests

`calorietrackerTests` and `calorietrackerUITests` targets exist but only contain Xcode boilerplate — there are no real tests. Verify behavior by hand on device. If you do add tests, run them with:

```bash
xcodebuild test -scheme calorietracker -destination 'id=E2095CDC-E117-527C-818A-9F741A145103'
```

## Code Review

Use Codex CLI before each PR / after each commit cluster:

```bash
codex exec review --commit <SHA> --full-auto
```

Address P1 and P2 findings. P3 is judgment-call.

## Architecture

### State / Dependency Injection

All stores use Swift's `@Observable` macro (not `ObservableObject`) and are injected with `.environment(...)` (not `.environmentObject(...)`). Created once in `calorietrackerApp.swift` and shared:

- `FoodStore` — food entries, favorites, macro aggregates
- `WeightStore` — weight entries; `addEntry` auto-syncs `profile.weightKg` to latest
- `ProfileStore` — **source of truth for `UserProfile`**. All reads/writes go through `profileStore.profile`. It listens for `.userProfileDidChange` and reloads so external writers (WeightStore, HealthKit observer) propagate to every view.
- `ChatStore` — Coach conversation history (persisted in UserDefaults as JSON, capped at last 20 messages in LLM payload)
- `NotificationManager`, `HealthKitManager`

Build setting `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor` means most types are main-actor isolated by default. New files are auto-discovered via `PBXFileSystemSynchronizedRootGroup` — **do not** edit `project.pbxproj` to register source files. (The `knownRegions` entry in pbxproj *is* edited when adding a new localization.)

### AI / LLM Routing (13 providers, 3 formats)

Two services, both route to the same 13 providers via `AIProvider.apiFormat`:

- **`GeminiService`** (`Services/GeminiService.swift`) — single-shot food/label analysis. Methods: `analyzeFood`, `analyzeTextInput`, `autoAnalyze`, `analyzeNutritionLabel`. All funnel through `callAI`.
- **`ChatService`** (`Services/ChatService.swift`) — multi-turn Coach chat. Builds a fresh system prompt every turn from the live profile + forecast + recent weights/foods, sends history + new user message.

The three API dialects are:
- **Gemini** (`.gemini`): `POST /models/{model}:generateContent` with `systemInstruction` + `contents[{role, parts}]`. API key goes in `X-goog-api-key` header, not the URL.
- **Anthropic Messages** (`.anthropic`): `POST /messages` with `system` + `messages` array, `x-api-key` header + `anthropic-version: 2023-06-01`.
- **OpenAI-compatible** (`.openaiCompatible`): `POST /chat/completions` with `messages` array (system + user/assistant). Used by OpenAI, xAI Grok, OpenRouter, Together AI, Groq, **Hugging Face** (router for open-weight models — Gemma, Qwen VL, Llama Vision), **Fireworks AI**, **DeepInfra** (open-weight hosts), **Mistral** (Pixtral vision), Ollama (local), and the **Custom (OpenAI-compatible)** provider where the user supplies their own base URL + free-form model name. OpenRouter and Hugging Face both set `supportsCustomModelName = true` so users can type any model ID alongside the preset list.

Adding a provider: add a case to `AIProvider` in `Models/AIProvider.swift`, set `baseURL`/`models`/`apiFormat`/`apiKeyPlaceholder`. If `apiFormat` is `.openaiCompatible` it works automatically; otherwise add a branch in both `GeminiService.callAI` and `ChatService.sendMessage`.

Transient 503/529/429 responses auto-retry with 1s/2s/4s backoff before surfacing the error — so "model overloaded" spikes usually resolve invisibly. This applies to **both** services (`GeminiService.makeRequest` and `ChatService.send`); if you add a third LLM entry point, port the same loop. On final failure, both services convert status codes to user-friendly copy ("The AI provider is overloaded right now. We retried a few times…") — don't surface raw `HTTP 503` / provider JSON messages.

### Speech-to-Text Routing (5 providers)

`VoiceInputView` branches on `SpeechSettings.selectedProvider`:
- **Native iOS** — `SFSpeechRecognizer` live streaming with partial results. One-tap: tap Analyze to stop + submit.
- **Remote** (OpenAI Whisper / Groq / Deepgram / AssemblyAI) — `AVAudioRecorder` writes 16 kHz mono AAC to a temp m4a, uploads on stop, `SpeechService.transcribe` returns final text. Two-tap flow so user can review the transcription before Analyze.

OpenAI + Groq share `/v1/audio/transcriptions` (multipart). Deepgram takes raw audio body with `Token <key>` auth. AssemblyAI is a 3-step flow: upload → submit → poll every 1s up to 60s.

### Coach chat (`ChatView` + `ChatStore` + `ChatService`)

- 5th tab in `ContentView` TabView (Home / Progress / Coach / Settings / About).
- `ChatStore` persists the full conversation in UserDefaults. `contextMessages()` returns the last 20 for the LLM payload (token-cost cap); the full history stays visible locally regardless.
- `ChatService.buildSystemPrompt` includes: profile (gender/age/height/weight/activity/goal + body fat if set), **which BMR formula is active** (Katch-McArdle if body fat known, else Mifflin-St Jeor), BMR/TDEE numbers, macro targets, `WeightAnalysisService.compute` output (predicted/observed trends, 30/60/90-day weight, days-to-goal, under-logging flag), last 10 weight entries, last 7 days of daily calorie totals.
- Goal-aware prompt chips — `ChatView.promptChips` returns a different set for Lose / Gain / Maintain.

### Weight forecast math (`WeightAnalysisService`)

Pure function. Uses up to 90 days of available data (auto-scales to however much the user actually has). Returns a `WeightForecast` with:
- `predictedWeeklyChangeKg` — from energy balance vs TDEE (7700 kcal ≈ 1 kg)
- `observedWeeklyChangeKg` — linear regression on weight entries in window (nil if <2 entries)
- 30/60/90-day predictions, `daysToGoal` if direction matches, `trendsDisagree` flag when predicted and observed differ by >0.3 kg/week

Used exclusively as context for `ChatService` — no standalone UI card (there used to be one; it was removed in favor of the Coach tab).

### FoodStore → HealthKit callbacks

`FoodStore` exposes four hooks that `calorietrackerApp.wireUpHealthKit()` wires to `HealthKitManager`:
- `onEntryAdded` → `writeNutrition(for:)` (immediate, synchronous)
- `onEntryDeleted` → `deleteNutrition(entryID:)`
- `onEntryUpdated` → `updateNutrition(for:)` (delete-then-write, awaited so they don't race)
- `onEntriesChanged` → notification rescheduling

Edits use `onEntryUpdated` rather than back-to-back delete+add so HealthKit can serialize the two operations atomically.

### WeightStore → HealthKit callbacks

- `onEntryAdded` → `writeWeight(for:)` — tags each HK sample with `fudai_weight_id = entry.id.uuidString`
- `onEntryDeleted` → `deleteWeight(entryID:)` — deletes by metadata predicate, bypasses the `healthKitEnabled` flag

`WeightStore.addEntry` also detects goal-weight crossings (previous-on-wrong-side → new-on-correct-side) and posts `.weightGoalReached` — the Progress tab listens and shows "Congratulations!".

**`WeightStore.init()` does NOT seed a starter entry.** An earlier version did, falling back to `UserProfile.default` (70 kg) when the store initialized before onboarding finished — that dropped a phantom 70 kg entry onto every fresh user's chart regardless of their real starting weight. The seed now runs from `calorietrackerApp.onChange(of: hasCompletedOnboarding)` via `weightStore.seedInitialWeightFromProfileIfEmpty(profile.weightKg)` once the profile is real. The method is idempotent (no-ops when `entries` is non-empty). Don't reintroduce an init-time seed.

`HealthKitManager.writeWeight(kg:date:)` (the profile-state push used by onboarding/Settings, not the per-entry variant) also tags samples with a synthetic `fudai_weight_id = UUID().uuidString`. Untagged samples are invisible to `deleteWeight(entryID:)`'s metadata predicate.

### HealthKit Conventions

`HealthKitManager` (`Stores/HealthKitManager.swift`) is the only HealthKit boundary.

- **`typesVersion`** (renamed from the old `authVersion` to dodge a CodeQL heuristic on "auth" keywords) is bumped when new HealthKit types are added. `needsReauthorization` returns `max(typesVersionKey, legacy healthKitAuthVersion)` < current so existing users aren't re-prompted after the rename.
- `requestAuthorization` only persists the new version via `persistCurrentTypesVersion()` when **all** dietary share types are `.sharingAuthorized`, so users who deny nutrition can re-prompt.
- Each nutrition sample carries `fudai_entry_id` metadata; each weight sample carries `fudai_weight_id`. Deletion uses metadata predicates.
- `writeNutrition` guards on `healthKitEnabled`. `deleteNutrition` and the delete half of `updateNutrition` always run (even with sync off) so in-app edits/deletes still clean up samples that were exported before the user flipped sync off — otherwise those would orphan in Apple Health forever.
- `backfillNutritionIfNeeded` is idempotent (queries Apple Health for each entry's UUID before writing) and is guarded by `isBackfillingNutrition` so scene-phase re-entry can't spawn overlapping scans. The caller passes `currentEntryIDs: () -> Set<UUID>` so a meal deleted mid-backfill won't be re-exported as a phantom sample.
- The body-measurements observer skips adding weights whose sample metadata contains `fudai_weight_id` — those are our own writes and are handled by `WeightStore.addEntry` directly. External samples (Apple Watch, scale, Health app entry) go through the observer's date+value dedup and get added to `WeightStore` with the sample's real date.
- `startBodyMeasurementObserver()` calls `stopObserver()` **first** before registering the three `HKObserverQuery` instances. `wireUpHealthKit()` runs on every scene-active, so without the tear-down the `observerQueries` array kept growing and one HK change would fire the callback N times per session.

Clear Food Log keeps Apple Health samples (per product spec — only saves storage). **Delete All Data is local-only too**: it wipes in-memory stores, UserDefaults, Keychain API keys, the Coach chat history, and the widget App Group snapshot — but intentionally does NOT touch Apple Health. Health data is user-owned and personal; if they want it gone they can use the Health app's Sources → Fud AI panel. (Earlier revisions purged HK nutrition in this flow; that was reverted in favor of treating HK as read/write-while-synced but untouched on resets.)

### Widgets (`FudAIWidgetsExtension` target + App Group)

The widget extension lives in the `FudAIWidgets/` folder as its own target (`com.apoorvdarshan.calorietracker.FudAIWidgets`). It's embedded into the main app via the `Embed Foundation Extensions` copy phase. Five supported families: `.systemSmall`, `.systemMedium`, `.accessoryCircular`, `.accessoryRectangular`, `.accessoryInline`.

Because widgets run in a separate process, they can't read the main app's `UserDefaults`. Data flows through an **App Group** shared container:
- **App Group ID**: `group.com.apoorvdarshan.calorietracker` (declared in both `calorietracker.entitlements` and `FudAIWidgets/FudAIWidgets.entitlements` — must match exactly).
- **`WidgetSnapshot`** is a small Codable struct (today's totals + goals) written by the main app into the shared suite under key `widget_snapshot_v1`.
- **Duplicated file**: `calorietracker/Services/WidgetSnapshot.swift` and `FudAIWidgets/WidgetSnapshot.swift` are identical copies. The widget target can't see the main app's sources (auto-discovery via `PBXFileSystemSynchronizedRootGroup` is per-target), so we keep two files in sync manually. If you change one, change both.
- **`WidgetSnapshotWriter.publish(...)`** (main app only) recomputes today's totals, writes the snapshot, and calls `WidgetCenter.shared.reloadAllTimelines()`. Called from three places in `calorietrackerApp.swift`: on `foodStore.onEntriesChanged`, on `.userProfileDidChange` notification (goal edits), and on scene-phase `.active` (so midnight rollover doesn't require an explicit food change).
- **Callback-wiring gotcha**: `wireUpFoodStoreCallback()` (where the `onEntriesChanged` closure gets installed) must run on **every** scene-active, not just the onboarding `false→true` transition. The `.onChange(of: hasCompletedOnboarding)` branch only fires once ever; if it were the sole wire-up site, existing users who completed onboarding before this code landed would never get the widget-refresh callback installed and would have to open the app to see new entries. Closure assignment is idempotent, so re-wiring on scene-active is safe.
- **Timeline policy**: `CalorieProvider.getTimeline` emits one entry for "now" and refreshes after 30 minutes as a safety net for days when the user doesn't log anything.
- **Freshness rules baked into the data layer** (don't regress these):
  1. `FudAIWidgets/WidgetSnapshot.read()` returns `nil` when `snapshot.dayStart` isn't today. The widget then falls back to `.empty` (zeroed today). Without this the 30-min timeline refresh kept showing yesterday's totals past midnight.
  2. `WidgetSnapshotWriter.publish(...)` filters entries with `Calendar.isDate($0.timestamp, inSameDayAs: Date())` — NOT a plain `>= startOfDay`. The latter would fold tomorrow-dated entries (pre-logged via the week strip) into today's widget totals.
  3. `WidgetSnapshot.clear()` is called from Delete All Data and from `refreshWidgetSnapshot()` when no profile is loaded. The App Group container sits outside `UserDefaults.standard`, so `removePersistentDomain` doesn't touch it — without `clear()` the widget would keep showing the previous profile's numbers after a reset.

Adding a new widget: add a new `Widget` conforming type in `FudAIWidgets/`, add it to `FudAIWidgetsBundle.body`, extend `CalorieWidgetView`'s `@Environment(\.widgetFamily)` switch if you're adding a new family. If you need additional data, extend `WidgetSnapshot` in **both** files (add fields with Codable defaults so old snapshots still decode).

### Localization (15 languages)

The app ships with `calorietracker/Localizable.xcstrings` (String Catalog) — ~200 UI strings × 15 locales: `en` (source), `ar`, `az`, `de`, `es`, `fr`, `hi`, `it`, `ja`, `ko`, `nl`, `pt-BR`, `ro`, `ru`, `zh-Hans`.

No in-app language picker. iOS auto-selects from the device language (matches Cal AI / MyFitnessPal / Yazio).

**Rule when adding UI strings**: every new `Text("...")`, `Button("...")`, `Section("...")`, `.alert("...")`, `.navigationTitle("...")`, placeholder, etc. must land in the catalog with translations for all 14 non-English locales before commit. For batches of 10+ strings spawn a general-purpose agent with the translation prompt (see prior commits for the format), merge the JSON into the catalog via a small Python script. `SWIFT_EMIT_LOC_STRINGS = YES` is set — Xcode auto-extracts new English strings on build, but will leave non-English entries empty; fill them in manually before shipping. Adding a new language requires a new code in the catalog + a new `knownRegions` entry in `project.pbxproj`.

### UI Structure

- `ContentView` hosts a **5-tab layout**: Home, Progress, Coach, Settings, About.
- `OnboardingView` is the first-run flow including an AI-provider-setup step.
- Sheets and pickers route through a single `.sheet(item: $activeSheet)` driven by an enum to avoid SwiftUI's stacked-sheet bugs.
- `Views/Theme.swift` (`AppColors`) holds the gradient palette used across the app.
- Picker sheets (height, weight, body-fat, calories/macros) seed their `@State` in `init()`, not `.onAppear`, to avoid a "flash to default value" on open.
- **Share sheets use `ActivityShareSheet`** (a `UIViewControllerRepresentable` over `UIActivityViewController` defined in `ContentView.swift`) — **not** SwiftUI's `ShareLink`. SwiftUI's `ShareLink(item: URL, message:)` silently drops the `message` arg for most share targets (Messages, Mail, X), so only the URL gets shared. Wrapping `UIActivityViewController` with `[String, URL]` in `activityItems` forwards both: iMessage shows the text plus the URL preview, Mail uses the text as body. Used by About → Share the App.

## Gotchas

- **SourceKit false positives**: editing surfaces "no module 'UIKit'" / "Cannot find type 'FoodEntry' in scope" errors that are not real. Build with `xcodebuild` to verify.
- **`.buttonStyle(.plain)` kills row tap-highlight** in a `List`. Use `.tint(.primary)` if you want the highlight while keeping primary text color.
- **Multiple `.sheet()` modifiers** on the same view cause white/black-screen bugs. Always use a single `.sheet(item:)` driven by an enum.
- **`ProgressView`** is renamed to `ProgressTabView` to avoid clashing with SwiftUI's built-in `ProgressView`.
- **`@Observable` tracking can miss property access buried in computed vars.** HomeView, ProgressTabView, and NutritionDetailView each read `let _ = profileStore.profile` at the top of `body` to force observation tracking. Don't remove those lines.
- **Dead files** (kept for git history but not referenced anywhere): `StoreManager.swift`, `PaywallView.swift`, `SpinWheelView.swift`, `CloudKitService.swift`. Don't add new code to them.
- **Persistent state** lives in two places: `UserDefaults` (preferences + JSON-encoded `entries`/`weights`/`favorites`/`coachChatHistory` arrays + `*Enabled`/`*Reminder*` keys read via `@AppStorage`) and iOS Keychain (LLM + STT API keys via `KeychainHelper`). There is no Core Data / SwiftData / iCloud.
- **CodeQL is not configured** anymore — the workflow was removed because it produced almost entirely false positives on UserDefaults writes near "auth"-keyword variables for a local-only app with no auth flows.

## Commit Style

- Plain factual messages. No co-author trailer. No marketing language.
- Commit and push immediately after each working change.
- When a commit adds user-facing strings, the message should mention the catalog was updated.

## Release Artifacts

- **`ASO.md`** at repo root holds the App Store listing copy (name, subtitle, promo text, keywords, What's New, full description, reviewer notes). Update it whenever the version bumps; the current header is `v3.0`. App Store Connect uploads happen by hand-pasting from this file — don't let it drift from the code.
- **App Store screenshots** live in `~/Documents/fud ai/appstore screenshots/` (raw 1179×2556 captures from device) and get composited into 1242×2688 marketing PNGs by ad-hoc Python scripts in `/tmp/`. The scripts are not in the repo — they're rebuilt per release. The current iteration uses PIL gradient backgrounds + a pixel-perfect iPhone 15 Pro Max frame + Bricolage Grotesque ExtraBold typography.
- Bump `MARKETING_VERSION` in `project.pbxproj` (two occurrences — main app + widget extension) before each App Store submission. `CURRENT_PROJECT_VERSION` is the build number.

## Identity

- Website: https://fud-ai.app
- App Store: https://apps.apple.com/us/app/fud-ai-calorie-tracker/id6758935726
- Email: apoorv@fud-ai.app
- X: @apoorvdarshan
- Donations: https://paypal.me/apoorvdarshan

---
> Source: [apoorvdarshan/fud-ai](https://github.com/apoorvdarshan/fud-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
