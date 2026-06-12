## asset-tracker

> This file governs how the AI agent writes, reviews, and refactors code for this project.

# CLAUDE.md — AI Agent Coding Guide
## Asset Tracker · Flutter + Supabase Cloud

This file governs how the AI agent writes, reviews, and refactors code for this project.
Read it fully before generating any code. Follow every rule without exception unless
explicitly overridden by the user in the current session.

---

## 1. Project Identity

| Field | Value |
|---|---|
| App name | Asset Tracker (working title) |
| Purpose | Family asset manager with billing and document expiry tracking |
| Users | Private family use (owner + spouse), ~2 active users |
| Platforms | iOS and Android (Flutter) |
| Backend | Supabase Cloud (free tier, `ap-southeast-1` Singapore region) |
| Timezone | `Asia/Jakarta` (WIB, UTC+7) — all date logic must respect this |
| Language | Dart (Flutter), SQL (Supabase migrations) |
| Min SDK | Flutter 3.19+, Dart 3.3+, iOS 16+, Android API 26+ |

---

## 2. Architecture Principles

### 2.1 Folder Structure

```
lib/
├── main.dart                  # Entry point only — no business logic
├── app.dart                   # MaterialApp, ThemeData, routing setup
├── core/
│   ├── constants/             # App-wide constants (strings, colors, sizes)
│   ├── errors/                # Failure classes, AppException
│   ├── extensions/            # Dart extension methods
│   ├── theme/                 # ThemeData, text styles, color scheme
│   └── utils/                 # Pure utility functions (date, format, etc.)
├── data/
│   ├── datasources/
│   │   ├── local/             # SQLite / Drift local DB
│   │   └── remote/            # Supabase client calls
│   ├── models/                # JSON-serializable data models
│   └── repositories/          # Concrete repository implementations
├── domain/
│   ├── entities/              # Pure domain objects (no JSON, no DB)
│   ├── repositories/          # Abstract repository interfaces
│   └── usecases/              # Single-responsibility use case classes
├── presentation/
│   ├── pages/                 # Full screens
│   ├── widgets/               # Reusable UI components
│   └── providers/             # Riverpod providers / state notifiers
└── services/
    ├── notification_service.dart
    ├── sync_service.dart
    └── auth_service.dart
```

### 2.2 State Management

- Use **Riverpod 2.x** (`flutter_riverpod`, `riverpod_annotation`) exclusively.
- No `setState` outside of purely local, ephemeral UI state (e.g., a text field focus).
- All providers must be typed. Never use `dynamic` or `Object` as provider return types.
- Use `AsyncNotifier` for async operations; `Notifier` for synchronous state.
- Keep providers in `presentation/providers/` colocated with the feature they serve.

### 2.3 Repository Pattern

- Every data operation goes through a repository interface in `domain/repositories/`.
- Concrete implementations live in `data/repositories/`.
- Use cases (`domain/usecases/`) call repositories only — never Supabase directly.
- Presentation layer calls use cases only — never repositories directly.
- This layering is non-negotiable. Do not shortcut it even for simple reads.

### 2.4 Offline-First

- Local SQLite (via **Drift**) is the single source of truth for the UI.
- All reads come from local DB. All writes go to local DB first, then sync to Supabase.
- Sync is eventual — the app must be fully functional with no internet connection.
- Mark records with `sync_status`: `synced | pending_create | pending_update | pending_delete`.
- Sync service runs on app foreground and on connectivity restore.

---

## 3. Security Rules — Non-Negotiable

### 3.1 Secrets and Credentials

- **Never hardcode** Supabase URL, anon key, or any secret in Dart source files.
- Store secrets in `--dart-define` at build time or in a `.env` file loaded via `flutter_dotenv`.
- `.env` must be in `.gitignore`. Commit a `.env.example` with placeholder values only.
- Never log secrets, tokens, or user PII to the console — not even in debug mode.

```dart
// CORRECT
final supabaseUrl = const String.fromEnvironment('SUPABASE_URL');
final supabaseAnonKey = const String.fromEnvironment('SUPABASE_ANON_KEY');

// WRONG — never do this
final supabaseUrl = 'https://xxxx.supabase.co';
```

### 3.2 Supabase Row Level Security (RLS)

- Every Supabase table MUST have RLS enabled. No exceptions.
- Default policy: deny all. Only grant what is explicitly needed.
- Every user can only access rows where `owner_id = auth.uid()` OR where they belong to the same `family_group_id`.
- Write migration SQL for every policy alongside the table creation migration.
- Never disable RLS even temporarily for debugging — use the Supabase dashboard to inspect data instead.

```sql
-- Example RLS policy pattern
alter table assets enable row level security;

create policy "Users can view their own family assets"
  on assets for select
  using (
    owner_id = auth.uid()
    or family_group_id in (
      select family_group_id from users where id = auth.uid()
    )
  );
```

### 3.3 Authentication

- Use Supabase Auth exclusively. No custom auth logic.
- Support email/password and Google OAuth.
- On logout: clear local DB sensitive data, invalidate all Riverpod providers, navigate to login.
- Session tokens are managed by the Supabase Flutter client — do not store them manually.
- Use `supabase.auth.onAuthStateChange` stream to reactively handle session changes.

### 3.4 Local Storage Security

- Use **flutter_secure_storage** for any sensitive data that must persist locally (refresh tokens, encryption keys).
- Never use `SharedPreferences` for sensitive values.
- Local Drift database should be encrypted using `sqlcipher_flutter_libs` if storing document metadata.
- Document files (PDFs, images) stored on-device must go in the app's private documents directory, not external storage.

### 3.5 File and Document Security

- Files uploaded to Supabase Storage must go into a private bucket (not public).
- Generate signed URLs with short expiry (max 1 hour) for display/download — never expose permanent public URLs.
- Validate file type and size before upload: max 10 MB per file, allow only `pdf`, `jpg`, `jpeg`, `png`, `heic`.
- Strip EXIF metadata from images before upload using `flutter_exif_rotation` or equivalent.

### 3.6 Input Validation

- Validate all user input on the client before any DB write.
- Validate again via Supabase database constraints and RLS (defense in depth).
- Sanitize text fields — trim whitespace, enforce max lengths matching DB column constraints.
- Never construct SQL strings manually. Always use parameterized queries via the Supabase client.

### 3.7 Biometric Lock

- Implement app-level biometric lock using `local_auth`.
- Lock the app after 5 minutes of backgrounding.
- Biometric prompt must appear on app resume before any data is visible.
- Store the lock preference in `flutter_secure_storage`.

---

## 4. Code Quality Rules

### 4.1 Dart Style

- Follow the official [Dart style guide](https://dart.dev/guides/language/effective-dart/style).
- Run `dart analyze` and `dart format` before every commit. Zero warnings permitted.
- Enable strict analysis in `analysis_options.yaml`:

```yaml
analyzer:
  strong-mode:
    implicit-casts: false
    implicit-dynamic: false
  errors:
    missing_required_param: error
    missing_return: error
    todo: warning
linter:
  rules:
    - always_declare_return_types
    - avoid_dynamic_calls
    - avoid_print
    - avoid_slow_async_io
    - cancel_subscriptions
    - close_sinks
    - prefer_const_constructors
    - prefer_final_locals
    - unawaited_futures
```

### 4.2 Naming Conventions

| Item | Convention | Example |
|---|---|---|
| Classes | `UpperCamelCase` | `AssetRepository` |
| Files | `snake_case` | `asset_repository.dart` |
| Variables | `lowerCamelCase` | `expiryDate` |
| Constants | `lowerCamelCase` with `const` | `const defaultReminderDays = 30` |
| Private | Prefix with `_` | `_syncStatus` |
| Riverpod providers | Suffix with `Provider` | `assetListProvider` |

### 4.3 Error Handling

- Never use bare `catch (e)` without handling or rethrowing.
- Use a typed `AppException` sealed class for domain errors:

```dart
sealed class AppException {
  const AppException();
}

class NetworkException extends AppException {
  final String message;
  const NetworkException(this.message);
}

class AuthException extends AppException {
  final String message;
  const AuthException(this.message);
}

class StorageException extends AppException {
  final String message;
  const StorageException(this.message);
}
```

- All repository methods return `Either<AppException, T>` using `fpdart` or wrap in `AsyncValue`.
- Never show raw exception messages to users. Map exceptions to user-friendly strings.

### 4.4 Async and Concurrency

- Every `async` function must handle errors. No fire-and-forget unless explicitly intentional.
- Mark intentional fire-and-forget with `unawaited()` from `dart:async`.
- Cancel stream subscriptions in `dispose()`. Use Riverpod's `ref.onDispose` for provider cleanup.
- Never `await` inside a loop for sequential network calls — use `Future.wait` for parallel calls.

### 4.5 Widget Rules

- Every widget file contains exactly one primary widget class.
- Prefer `const` constructors everywhere possible.
- Extract any widget subtree exceeding ~50 lines into its own named widget class.
- Never put business logic (API calls, DB queries) inside `build()` methods.
- Use `ListView.builder` / `SliverList` for any list that could exceed 20 items — never `Column` with many children.

### 4.6 Date and Time

- Store all datetimes in UTC in the database.
- Convert to `Asia/Jakarta` only at the display layer using the `timezone` package.
- Never use `DateTime.now()` directly for expiry logic — always use `DateTime.now().toUtc()`.
- Use the `intl` package for all date formatting. Never manually format dates with string interpolation.

```dart
// CORRECT
import 'package:intl/intl.dart';
import 'package:timezone/timezone.dart' as tz;

final jakarta = tz.getLocation('Asia/Jakarta');
final localNow = tz.TZDateTime.now(jakarta);
final formatted = DateFormat('dd MMM yyyy', 'id_ID').format(localNow);

// WRONG
final formatted = '${date.day}/${date.month}/${date.year}';
```

### 4.7 Comments and Documentation

- Every public class and public method must have a doc comment (`///`).
- Explain *why*, not *what* — the code shows what; comments explain reasoning.
- Mark unfinished work with `// TODO(username): description` — never leave silent dead code.
- Do not comment out code — delete it. Git history preserves it.

---

## 5. Flutter-Specific Rules

### 5.1 Theming

- All colors, text styles, and spacing must come from `ThemeData` — no hardcoded values in widgets.
- Define a single `AppTheme` class in `core/theme/` that exports `lightTheme` and `darkTheme`.
- Support dark mode from day one. Never assume `Colors.white` as a background.
- Use Material 3 (`useMaterial3: true`). Do not use deprecated Material 2 components.

### 5.2 Navigation

- Use **GoRouter** for all navigation. No `Navigator.push` directly.
- Define all routes as named constants in a `AppRoutes` class.
- Handle deep links and auth redirect guards in the GoRouter `redirect` callback.
- Never pass large objects through route parameters — pass IDs only, load data in the destination.

### 5.3 Performance

- Use `flutter_hooks` or Riverpod's `select` to prevent unnecessary widget rebuilds.
- Profile with Flutter DevTools before declaring any screen "done."
- Image assets: use WebP format, provide 1x/2x/3x variants, use `cached_network_image` for remote images.
- Avoid `Opacity` widget for animations — use `AnimatedOpacity` or `FadeTransition`.

### 5.4 Accessibility

- All interactive elements must have a minimum touch target of 48×48dp.
- Every image must have a `semanticLabel`. Every icon button must have a `Tooltip`.
- Support font scaling — never use hardcoded font sizes that ignore `MediaQuery.textScaleFactor`.
- Test with TalkBack (Android) and VoiceOver (iOS) before each release.

---

## 6. Supabase Rules

### 6.1 Client Initialization

```dart
// in main.dart — initialize before runApp
await Supabase.initialize(
  url: const String.fromEnvironment('SUPABASE_URL'),
  anonKey: const String.fromEnvironment('SUPABASE_ANON_KEY'),
  authOptions: const FlutterAuthClientOptions(
    authFlowType: AuthFlowType.pkce,  // PKCE for mobile security
  ),
  realtimeClientOptions: const RealtimeClientOptions(
    logLevel: RealtimeLogLevel.info,
  ),
);
```

### 6.2 Database Migrations

- Every schema change must be a versioned SQL migration file in `supabase/migrations/`.
- Never modify an existing migration — always create a new one.
- Migration filenames: `YYYYMMDDHHMMSS_description.sql`
- Include rollback SQL as a comment at the bottom of every migration.
- Test migrations on a local Supabase instance (`supabase start`) before applying to cloud.

### 6.3 Realtime Subscriptions

- Subscribe to changes per `family_group_id` — never subscribe to entire tables.
- Always unsubscribe on widget/provider dispose using `channel.unsubscribe()`.
- Handle subscription errors and reconnect gracefully — network drops are normal on mobile.

### 6.4 Edge Functions

- Use Edge Functions only for: expiry check cron jobs, FCM push dispatch.
- Edge Functions must validate the calling user's JWT before doing anything.
- Keep functions small and single-purpose. No function should exceed ~150 lines.

---

## 7. Testing Requirements

### 7.1 What Must Be Tested

| Layer | Tool | Coverage target |
|---|---|---|
| Domain use cases | `flutter_test` | 100% |
| Repository implementations | `mocktail` mocks | 80%+ |
| Expiry engine logic | `flutter_test` | 100% |
| Date/timezone utils | `flutter_test` | 100% |
| Critical widgets | `flutter_test` widget tests | Key user flows |

### 7.2 Test Conventions

- Test file mirrors source file: `lib/domain/usecases/get_assets.dart` → `test/domain/usecases/get_assets_test.dart`
- Use `mocktail` for mocking — not `mockito` (requires code gen).
- Arrange / Act / Assert structure in every test.
- Test edge cases: empty lists, null fields, network errors, timezone boundaries.

---

## 8. Git and Commit Rules

- Branch naming: `feat/`, `fix/`, `chore/`, `refactor/` prefixes.
- Commit messages follow Conventional Commits: `feat(assets): add swipe-to-delete on asset card`
- Never commit: `.env`, `*.jks`, `google-services.json`, `GoogleService-Info.plist`, `*.keystore`
- These files go in `.gitignore` before the first commit.
- Run `flutter test` locally before pushing. Never push a failing test.

---

## 9. Dependencies — Approved List

Only use packages from this list unless explicitly approved by the user.

### Core
| Package | Purpose |
|---|---|
| `flutter_riverpod` + `riverpod_annotation` | State management |
| `go_router` | Navigation |
| `drift` + `drift_flutter` | Local SQLite ORM |
| `supabase_flutter` | Backend client |
| `flutter_dotenv` | Environment variables |

### Security
| Package | Purpose |
|---|---|
| `flutter_secure_storage` | Encrypted key-value store |
| `local_auth` | Biometric authentication |
| `sqlcipher_flutter_libs` | SQLite encryption |

### UI
| Package | Purpose |
|---|---|
| `table_calendar` | Calendar for expiry dates |
| `fl_chart` | Charts/statistics |
| `cached_network_image` | Remote image caching |
| `shimmer` | Loading skeleton screens |
| `flutter_slidable` | Swipe actions on list items |

### Utilities
| Package | Purpose |
|---|---|
| `timezone` | Timezone-aware datetime |
| `intl` | Date/number formatting |
| `fpdart` | Functional error handling (`Either`) |
| `freezed` + `json_serializable` | Immutable models + JSON |
| `file_picker` | Document/file selection |
| `open_filex` | Open documents natively |
| `connectivity_plus` | Network status detection |
| `flutter_local_notifications` | Local scheduled reminders |
| `firebase_messaging` | FCM push notifications |
| `image_picker` | Camera / gallery access |

### Dev
| Package | Purpose |
|---|---|
| `mocktail` | Mocking for tests |
| `build_runner` | Code generation |
| `flutter_lints` | Lint rules |

---

## 10. What the Agent Must Never Do

- Never disable RLS on any Supabase table.
- Never store secrets in source code or commit them.
- Never use `dynamic` types — always use explicit types.
- Never call Supabase directly from a widget — always go through use cases → repositories.
- Never use `DateTime.now()` for expiry comparisons without UTC conversion.
- Never use `print()` — use a proper logger (`logger` package) with log levels.
- Never generate placeholder/dummy screens without wiring up real providers.
- Never skip error handling with empty `catch {}` blocks.
- Never use `Column` for long scrollable lists — always use `ListView.builder`.
- Never hardcode Indonesian strings — use `intl` ARB files for all user-facing text (i18n-ready from day one even if only Indonesian is supported initially).

---
> Source: [neromorph/asset-tracker](https://github.com/neromorph/asset-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
