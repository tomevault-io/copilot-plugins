## seminary-sidekick-flutter

> > **Single entry point for AI agents.** Read this file before touching any code.

# CLAUDE.md — Seminary Sidekick

> **Single entry point for AI agents.** Read this file before touching any code.
> For the task board, see `TODO.md`.

---

## What This App Is

A focused scripture mastery tool for the 100 Doctrinal Mastery scriptures of The Church of Jesus Christ of Latter-day Saints.

The core loop is **Study → Build → Prove → Master**. Users study the text, then use **Word Builder** (the primary mastery tool) to progressively prove they can reproduce it from memory. Supplementary practice tools (Scripture Match, Quick Quiz) help with recognition and comprehension but do not gate mastery.

**Key UX principle**: The path to mastery must be obvious. When a user opens a scripture, they should immediately see where they are on the mastery path, what to do next, and what “mastered” means. Word Builder lives directly under each scripture as the central mastery tool.

**Design philosophy**: Fun first with warm, satisfying feedback (animations, haptics, confetti, progressive difficulty). The experience is reverent and purposeful while remaining engaging for seminary students.

**Status**: Free-tier MVP is complete. The app is now moving into the **Premium tier**, which unlocks the **Seminary Sidekick** — an AI companion powered by Grok.  
Premium features focus on deeper understanding and application through AI-generated journal prompts, reflection questions, smart goals, timeline insights, and subtle engagement enhancements that make diligent study feel natural and rewarding.

**Business model**: Freemium.  
- **Free tier** = Full mastery loop (Word Builder, Practice tools, spaced repetition, progress tracking, activity feed).  
- **Premium tier** = Seminary Sidekick AI (JSON snapshot → structured response), journal with dynamic prompts, curriculum awareness, gentle reminders, and light engagement layers.

---

## Tech Stack

| Choice | Why |
|--------|-----|
| **Flutter + Dart** | Pixel-level animation control, game-quality performance |
| **Riverpod** (StateNotifier) | Predictable state, testable, no context dependency |
| **GoRouter** (StatefulShellRoute) | Bottom nav with preserved tab state |
| **Hive** | Lightweight local persistence |
| **Google Fonts** | Merriweather (headings) + Inter (body) |
| **flutter_animate** | Animations |
| **confetti** | Celebration effects |
| **audioplayers** | Sound effects |
| **purchases_flutter** (RevenueCat) | In-app subscriptions for freemium model |

---

## Project Structure

**Important update for Premium**:
New files and folders will be added. See the full structure below.
lib/
├── main.dart                    # Entry: Hive init, orientation lock, ProviderScope
├── app.dart                     # GoRouter config, shell with bottom nav
├── models/
│   ├── enums.dart
│   ├── scripture.dart
│   ├── user_progress.dart
│   ├── scripture_mastery.dart
│   ├── sidekick_snapshot.dart      # JSON sent to Grok
│   ├── sidekick_response.dart      # Structured response from Grok
│   └── journal_entry.dart
├── data/
│   └── scriptures_data.dart
├── providers/
│   ├── scripture_provider.dart
│   ├── progress_provider.dart
│   ├── scripture_mastery_provider.dart
│   ├── mastery_dates_provider.dart
│   ├── matching_game_provider.dart
│   ├── word_builder_provider.dart
│   ├── quiz_game_provider.dart
│   ├── notes_provider.dart
│   ├── sidekick_provider.dart      # Main AI orchestration
│   ├── subscription_provider.dart  # Freemium state + RevenueCat
│   ├── goals_provider.dart
│   └── journal_provider.dart
├── screens/
│   ├── home_screen.dart
│   ├── scripture_list_screen.dart
│   ├── scripture_detail_screen.dart
│   ├── memorize_screen.dart
│   ├── practice_hub_screen.dart    # Renamed from games_hub
│   ├── progress_screen.dart
│   ├── journal_screen.dart
│   ├── sidekick_chat_screen.dart
│   ├── upgrade_screen.dart
│   └── premium/                    # Optional folder for premium-only screens
├── services/
│   ├── audio_service.dart
│   ├── speech_service.dart
│   └── sidekick_service.dart       # Grok API calls + snapshot logic
├── widgets/
│   ├── scripture_card.dart
│   ├── mastery_badge.dart
│   ├── progress_ring.dart
│   └── premium_teaser.dart
└── theme/
└── app_theme.dart


---

## Data Model
### Scripture (immutable)

Fields: `id` (String, '1'..'100'), `book` (ScriptureBook enum), `volume`, `reference`, `name` (topic), `keyPhrase`, `fullText`, `words` (pre-split, auto-computed), `wordCount` (auto-computed).
**New models for Premium**:
- `SidekickSnapshot`: Contains current mastery state, seminary curriculum week, goals, recent activity, etc.
- `SidekickResponse`: Structured JSON from Grok that triggers app actions (daily prompt, quick win, goal suggestion, etc.).

(Keep all existing models unchanged)

**Premium models** (implemented):
- `SidekickSnapshot`: JSON payload sent to Grok on app launch. Contains `MasteryStats` (per-level counts), `List<ScriptureProgressSummary>` (up to 8 needing attention), recent activity strings, curriculum week, goals, streak, days active. Built by `SidekickNotifier._buildSnapshot()` from existing providers.
- `SidekickResponse`: Structured JSON from Grok. All fields optional: `dailyPrompt`, `suggestedGoal` (SidekickGoal), `quickWin` (QuickWin with scriptureId + actionType), `timelineInsight`, `reminder`, `reflectionPrompts`, `encouragement`, `connections` (ScriptureConnection). Has `fromJson`/`toJson` and `offlineFallback()` factory.
- `SidekickMessage`: Chat message with `role` (user/assistant), `content`, `timestamp`. Has `toApiMessage()` for API calls.

### UserProgress (per scripture × game type)

Fields: `scriptureId`, `gameType`, `highestDifficultyCompleted`, `totalAttempts`, `correctAttempts`, `currentStreak`, `bestStreak`, `bestTime`, `lastPracticed`, `accuracy`, `masteryLevel`, `needsReview`, `consecutivePerfectMaster`.

Storage key format: `{scriptureId}_{gameType.name}`


### Enums

- **ScriptureBook**: 4 values with `displayName` and `abbreviation`
- **MasteryLevel**: newScripture → learning → familiar → memorized → mastered → eternal (with color, icon, minAccuracy)
- **GameType**: matching, wordOrder, quiz (with displayName, description, icon) — note: "GameType" is a legacy name in code; conceptually wordOrder=mastery tool, matching/quiz=practice quizzes
- **DifficultyLevel**: beginner → intermediate → advanced → master (with scriptureCount, hasTimer, allowRetry, extraDistractors)

### Mastery System

Mastery is driven entirely by Word Builder progression:

- **New** (gray): Haven't started Word Builder
- **Learning** (orange): Completed WB Beginner
- **Familiar** (yellow): Completed WB Intermediate
- **Memorized** (green): Completed WB Advanced
- **Mastered** (blue): 3 consecutive perfect WB Master completions
- **Eternal** (gold): Sustained Mastered for 6 months (permanent, never decays)

**Shortcut**: If you can nail Master difficulty without doing lower tiers, you've proven it. (Planned — TASK-031.)

**Gentle decay**: 14+ days → "Needs Review" flag. 30+ days → drops one tier. Floor at Familiar. Eternal never decays.

**Why only Word Builder?** It's the only tool that tests *production* — can you produce the words from memory? Match/Quiz test recognition (different cognitive skill). You haven't "mastered" a scripture until you can type it cold.

---

## Mastery Tool: Word Builder

Word Builder is the PRIMARY mastery tool. It lives under each scripture (accessed from scripture detail), not in the quizzes/games hub. It's how you prove you know a scripture. The four difficulty tiers map directly to mastery levels:

- **Beginner** (chunk-tap): 3-word chunks, tap in order → earns **Learning** mastery
- **Intermediate** (chunk-tap): 2-word chunks + distractors from other scriptures → earns **Familiar** mastery
- **Advanced** (typing): Type the passage with first-letter hints. Wrong char turns red, must backspace → earns **Memorized** mastery
- **Master** (typing): Blind typing (all underscores). Any wrong char resets everything. Speech-to-text available → 3 consecutive perfect runs earns **Mastered**

**Shortcut rule**: If a user can complete Master difficulty perfectly, they've proven mastery regardless of whether they did Beginner/Intermediate/Advanced. The system should recognize this (planned — see TODO.md TASK-031).

### Study Tool: Memorize
Study aid accessed from scripture detail. Two modes: First Letter (progressive shrink to first letter then underscore) and Full Hide (straight to underscores). Tap words to toggle, or use Hide Next / Reveal All / Hide All. This is for studying — no mastery progression attached.

### Supplementary Quizzes (not mastery-gating)

These help with recognition and comprehension but do NOT drive mastery. They live in a "Practice" or "Quizzes" tab (currently called "Games" — rename planned).

- **Scripture Match**: Two columns — key phrases vs references. Tap-to-select or drag-and-drop to match. Tests reference recognition.
- **Quick Quiz**: Given a passage/reference, select correct key phrase/reference from 4 choices. Three question types rotate. Tests comprehension.

---

## Conventions

### Naming

| Type | Convention | Example |
|------|-----------|---------|
| Files | `snake_case.dart` | `matching_game_screen.dart` |
| Classes | `PascalCase` | `MatchingGameScreen` |
| Providers | `camelCaseProvider` | `wordBuilderProvider` |
| Notifiers | `[Feature]Notifier` | `WordBuilderNotifier` |
| State classes | `[Feature]State` | `WordBuilderState` |

### Theme — Never Hardcode Colors or Text Styles

```dart
// Colors — always use AppTheme.*
AppTheme.primary          // Warm rust (#D9805F) — main actions, headings
AppTheme.secondary        // Sage green (#618C84) — secondary actions
AppTheme.accent           // Calm blue (#5B8ABF) — highlights, links
AppTheme.success          // Correct feedback
AppTheme.error            // Incorrect feedback
AppTheme.gold             // Stars, achievements
AppTheme.dark             // Text
AppTheme.surface          // Card backgrounds
AppTheme.offWhite         // Scaffold background
AppTheme.bookColor('oldTestament')  // Book-specific
AppTheme.premiumGold      // Premium badges, upgrade CTAs (#D4A843)
AppTheme.premiumGoldLight // Premium background tint (#F5E6B8)
AppTheme.premiumGradientStart / premiumGradientEnd  // Premium icon gradients

// Typography — always use Theme.of(context).textTheme.*
displayMedium   // Big headings (Merriweather)
headlineMedium  // Section headings
titleLarge      // Card titles (Inter)
bodyLarge       // Body text
bodySmall       // Captions

// Spacing
AppTheme.spacingSm (8), spacingMd (16), spacingLg (24), spacingXl (32)

// Radii
AppTheme.radiusSm (8), radiusMd (12), radiusLg (16)
```

### Provider Pattern

Every stateful feature uses `StateNotifier<FeatureState>` with an immutable state class that has `copyWith`:

```dart
class FeatureState {
  final /* fields */;
  const FeatureState({/* required fields */});
  FeatureState copyWith({/* optional overrides */}) { ... }
}

class FeatureNotifier extends StateNotifier<FeatureState> {
  FeatureNotifier() : super(FeatureState(/* defaults */));
  void someAction() { state = state.copyWith(/* changes */); }
}

final featureProvider = StateNotifierProvider<FeatureNotifier, FeatureState>(
  (ref) => FeatureNotifier(),
);
```

Read-only providers use `Provider` or `Provider.family`.

### Quiz/Practice Screen Pattern

`ConsumerStatefulWidget` with `TickerProviderStateMixin`. Start game in `postFrameCallback`. Use `ref.listen()` for state transitions. Timer + shake/pulse animation controllers + haptic feedback on every action. Exit confirmation dialog. Navigate to shared `GameResultsScreen` via `Navigator.pushReplacement`.

Word Builder follows this same pattern but is launched from scripture detail, not the practice hub.

### Navigation

| Route type | Method | When |
|-----------|--------|------|
| Tab navigation | GoRouter `context.go('/path')` | Main tabs |
| Scripture browsing | GoRouter `context.go('/scripture/$id')` | Detail screens |
| Game screens | `Navigator.of(context).push()` | Transient overlays |
| Game results | `Navigator.of(context).pushReplacement()` | Replace game with results |
| Memorize tool | `Navigator.of(context).push()` | From scripture detail |
| Upgrade screen | GoRouter `context.go('/upgrade')` or `Navigator.push()` | From teasers/prompts |

### Feedback on Every Action

| Action | Visual | Haptic |
|--------|--------|--------|
| Correct | Green + pulse (300ms) | `lightImpact()` |
| Incorrect | Red + shake (~400ms) | `mediumImpact()` |
| Scripture complete | Green checkmark | `heavyImpact()` |
| Game complete | Navigate to results | `heavyImpact()` |

Use `AnimatedContainer` for smooth state transitions.

### Data Access

```dart
ref.watch(scripturesProvider)                          // All scriptures
ref.watch(scripturesByBookProvider('oldTestament'))     // By book
ref.watch(scriptureByIdProvider('42'))                  // By ID
ref.watch(searchScripturesProvider('nephi'))            // Search
ref.watch(scriptureMasteryProvider('42'))               // Holistic mastery (Word Builder-driven)
ref.watch(holisticStatsProvider)                        // Aggregate mastery stats
ref.watch(masteryLevelProvider(('42', GameType.matching)))  // Per-game mastery (legacy, still used by game screens)
ref.watch(userStatsProvider)                            // Overall stats
ref.read(progressProvider.notifier).recordAttempt(...)  // Record attempt

// Subscription (freemium)
ref.watch(subscriptionProvider)                        // Full subscription state
ref.watch(isPremiumProvider)                           // bool — is user premium?
ref.watch(canShowUpgradePromptProvider)                // bool — rate-limited prompt check
ref.read(subscriptionProvider.notifier).purchasePlan(plan)  // Trigger purchase
ref.read(subscriptionProvider.notifier).dismissUpgradePrompt()  // Record dismissal

// Sidekick AI (premium)
ref.watch(sidekickProvider)                            // Full sidekick state
ref.watch(sidekickResponseProvider)                    // Latest SidekickResponse
ref.watch(dailyPromptProvider)                         // String? — daily prompt
ref.watch(quickWinProvider)                            // QuickWin? — next action
ref.watch(reflectionPromptsProvider)                   // List<String> — journal prompts
ref.watch(chatHistoryProvider)                         // List<SidekickMessage>
ref.watch(isChatLoadingProvider)                       // bool — chat in flight?
ref.read(sidekickProvider.notifier).refreshSession()   // Re-fetch from Grok
ref.read(sidekickProvider.notifier).sendMessage(text)  // Chat message
```

---

## Adding a New Quiz/Practice Tool — Checklist

1. Create `lib/providers/[quiz]_provider.dart` — State class with `copyWith`, Notifier with `startGame()`, game actions, `nextScripture()`, `clearFeedback()`, `StateNotifierProvider`
2. Create `lib/screens/games/[quiz]_screen.dart` — `ConsumerStatefulWidget` with `TickerProviderStateMixin`, timer, animations, haptics, exit dialog, navigate to `GameResultsScreen`
3. Update `lib/screens/games_hub_screen.dart` (will become practice hub) — import, `isAvailable` check, `_launchGame()` switch
4. Update `TODO.md`
5. Ensure `GameType` enum entry exists in `enums.dart`

**Note**: Word Builder is NOT a quiz — it's the mastery tool and lives under scripture detail. Only supplementary practice tools go in the practice hub.

## Adding a New Provider — Checklist

1. Create file in `lib/providers/`
2. Define immutable state class with `copyWith`
3. Define notifier extending `StateNotifier<YourState>`
4. Export provider as top-level `final`
5. Add convenience `Provider.family` providers if needed

---

## Agent Coordination

### Quick Start

```
1. Read this file (CLAUDE.md)      → Understand everything
2. Read TODO.md                    → Find an open task
3. Claim the task                  → Write your agent ID into TODO.md, commit
4. Do the work                     → Follow the conventions above
5. Mark task done                  → Update TODO.md, commit
```

### Claiming a Task

1. Read `TODO.md` fresh (never rely on cached state)
2. Find a task with `status: open`
3. Check `depends_on` — don't start blocked work
4. Edit `TODO.md`: set `status: in_progress`, `claimed_by: [your-id]`, `started: [ISO timestamp]`
5. **Commit the claim before writing code**: `git add TODO.md && git commit -m "claim TASK-XXX: [description]"`

If two agents claim the same task, the second commit fails with a merge conflict. Pull, see it's taken, pick another task.

### Completing a Task

1. Finish code changes
2. Edit `TODO.md`: set `status: done`, `completed` timestamp, check acceptance criteria, add notes
3. Commit everything: `git add -A && git commit -m "complete TASK-XXX: [what was done]"`

### Blocked or Abandoned

- **Blocked**: Set `status: blocked`, add `blocked_by` note, commit, move on
- **Abandoned**: Set `status: open`, clear `claimed_by`, add notes explaining why, commit

### File Ownership

Two agents should never edit the same file concurrently. Check `files_to_touch` for conflicts.

**Shared files (extra caution)**: `enums.dart`, `games_hub_screen.dart`, `main.dart`, `pubspec.yaml`, `TODO.md`

### Commit Format

```
[verb] TASK-XXX: [concise description]
```

Verbs: `claim`, `complete`, `fix`, `add`, `update`, `refactor`, `block`

---

## Testing

### Setup

Dev dependencies: `mockito ^5.4.4`, `fake_async ^1.3.1` (already added).

```
test/
├── models/
│   ├── scripture_test.dart
│   ├── user_progress_test.dart
│   ├── enums_test.dart
│   └── scripture_mastery_test.dart  # Holistic mastery: linear path, decay, Eternal, requirements
├── providers/
│   ├── scripture_provider_test.dart
│   ├── progress_provider_test.dart  # Includes consecutivePerfectMaster tracking tests
│   ├── matching_game_provider_test.dart
│   └── word_builder_provider_test.dart
├── screens/
│   └── (widget tests — lower priority)
└── helpers/
    └── test_helpers.dart       # 5 shared test scriptures
```

### Test Priorities

1. **Models** (fast, no deps): word splitting, verse stripping, difficulty tiers, copyWith, equality, enum values
2. **Scripture mastery model** (core mastery logic): linear path levels, Word Builder-driven computation, gentle decay, Eternal tier, sub-progress, requirements generation
3. **Progress provider** (core business logic): recordAttempt, accuracy, streaks, mastery thresholds, game type isolation, consecutivePerfectMaster tracking
4. **Scripture provider** (read-only queries): all 100 scriptures, by book, by ID, search
5. **Matching game provider**: game init, selection, matching, completion, star rating
6. **Word builder provider** (most complex): chunk-tap mode + typing mode across all 4 difficulty tiers
7. **Widget tests** (lowest priority): MemorizeScreen, GamesHubScreen, GameResultsScreen

### Testing StateNotifiers Directly

```dart
late ProgressNotifier notifier;
setUp(() { notifier = ProgressNotifier(); });

test('first correct attempt creates progress', () {
  notifier.recordAttempt(scriptureId: 'test-1', gameType: GameType.matching, correct: true);
  final progress = notifier.getProgress('test-1', GameType.matching);
  expect(progress!.totalAttempts, 1);
  expect(progress.accuracy, 100.0);
});
```

### Testing Providers with ProviderContainer

```dart
late ProviderContainer container;
setUp(() { container = ProviderContainer(); });
tearDown(() { container.dispose(); });

test('scripturesProvider returns all 100', () {
  expect(container.read(scripturesProvider).length, 100);
});
```

### Test Conventions

- One test file per source file
- Group related tests with `group()`
- Test names describe behavior: `'recordAttempt with correct answer increments streak'`
- Fresh notifier in `setUp`, no shared state
- Use fixtures from `test_helpers.dart`
- Assert state, not implementation

### Running Tests

```bash
flutter test                    # All tests
flutter test test/providers/    # Specific directory
flutter test --coverage         # With coverage
flutter analyze                 # Must pass with no errors
```

---

## Build & Run

```bash
flutter pub get          # Install deps
flutter analyze          # Must pass with no errors
flutter test             # Run all tests
flutter run              # Run app
```

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `CLAUDE.md` | This file — single agent entry point (mastery spec is inline here) |
| `TODO.md` | Task board (claim/complete tasks here) |
| `app_theme.dart` | Single source of truth for colors and spacing |
| `scriptures_data.dart` | All 100 scripture entries |
| `subscription_provider.dart` | Freemium state, RevenueCat integration, prompt rate-limiting |
| `upgrade_screen.dart` | Full-screen premium upgrade experience (plan selection, purchase) |
| `premium_teaser.dart` | Reusable upgrade prompt widgets (PremiumTeaser, PremiumInlineLink, PremiumGate) |
| `sidekick_service.dart` | Grok/xAI API client, system prompts, JSON parsing |
| `sidekick_provider.dart` | AI orchestration: snapshot building, session refresh, chat, caching |
| `sidekick_snapshot.dart` | JSON payload model sent to Grok (MasteryStats, ScriptureProgressSummary) |
| `sidekick_response.dart` | Structured response model from Grok (SidekickGoal, QuickWin, ScriptureConnection, SidekickMessage) |

## Current Task Status

**Free-tier MVP** (completed 2026-04-06):
- All core tasks done.
- UX restructure complete: Word Builder is the hero on scripture detail.

**Freemium infrastructure** (completed 2026-04-06):
- TASK-033 done: SubscriptionProvider, UpgradeScreen, PremiumTeaser widgets, rate-limited prompts.

**Seminary Sidekick AI core** (completed 2026-04-06):
- TASK-034 done: SidekickService (Grok/xAI), SidekickProvider (orchestration + caching), snapshot/response models, chat support.

**Premium teaser & upgrade experience** (completed 2026-04-06):
- TASK-039 done: Premium teasers placed in home screen (after stats), scripture detail (inline link + mastery-level teaser), and onboarding (Sidekick mention on final page). All rate-limited and dismissible.

**Active direction** (Premium Tier – Seminary Sidekick):
- **P1**: TASK-035 (AI-powered journal & dynamic reflection prompts)
- **P1**: TASK-036 (AI-driven goals, timeline & gentle reminders)
- **P1**: TASK-037 (“Ask Your Sidekick” chat)
- **P2**: TASK-038 (Premium polish)
- **P2**: TASK-040 (Subtle engagement enhancements)

---

## Available Skills
- `/grill-me` — Use this when you need deep, relentless questioning on game mechanics, mastery logic, UX flows, or architecture decisions.
- `/request-refactor-plan` — Before any major refactor (especially providers or game screens).
- `/sidekick-prompt` — Ask for help crafting or refining the Seminary Sidekick system prompt for Grok.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sophomorica)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sophomorica)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
