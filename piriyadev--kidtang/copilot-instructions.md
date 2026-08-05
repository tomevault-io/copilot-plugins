## kidtang

> > **AI Instructions**: Read this file before making any changes. Follow the existing architecture. Prefer modifying existing patterns over creating new ones. Avoid unnecessary refactoring. Keep changes minimal and consistent. Reuse existing utilities whenever possible. Preserve backward compatibility unless explicitly requested. Search the repository before implementing duplicate functionality. Explain breaking changes before applying them.

# AGENTS.md — Kidtang Project Context

> **AI Instructions**: Read this file before making any changes. Follow the existing architecture. Prefer modifying existing patterns over creating new ones. Avoid unnecessary refactoring. Keep changes minimal and consistent. Reuse existing utilities whenever possible. Preserve backward compatibility unless explicitly requested. Search the repository before implementing duplicate functionality. Explain breaking changes before applying them.

---

## 1. Project Overview

- **Name**: Kidtang (กิดตัง) — "มาจ่ายเงินกัน" (Let's split the bill)
- **Purpose**: A bill-splitting app that lets users create bills, add members and line items, calculate each person's share (with VAT, service charge, tip, discount), track payments, and manage groups of friends.
- **Main Features**:
  - Create and manage bills with line items, custom splits, and per-item payer tracking
  - Bill lifecycle: `draft` → `pending_payment` → `completed`
  - Group management (shared bill spaces)
  - Friends system with invite/accept flow
  - In-app notifications (group invites, friend requests, bill events)
  - Firebase push notifications (FCM)
  - Multi-currency support (default THB)
  - Bilingual UI: Thai (default) and English
  - Dark/light theme
  - PromptPay QR code generation for payment
  - Google AdMob integration (remotely toggled via `app_config`)
- **Supported Platforms**: iOS, Android, Web (PWA installable)

---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| Language | Dart (SDK ≥3.0) |
| Framework | Flutter (Material 3) |
| State Management | `provider` ^6.1.2 — `ChangeNotifier` + `MultiProvider` |
| Navigation | `go_router` ^13.2.0 — `StatefulShellRoute` for bottom nav |
| Backend / DB | Supabase (Postgres + RLS + Realtime + Edge Functions) |
| Auth | Supabase Auth (email/password, Google, LINE, Apple) |
| Push Notifications | Firebase Messaging (FCM) + `flutter_local_notifications` |
| Charts | `fl_chart` ^0.68.0 |
| Fonts | `google_fonts` — Anuphan (headings), NotoSansThai (body) |
| i18n | Manual JSON files in `assets/i18n/` — no codegen |
| Ads | Google Mobile Ads (`google_mobile_ads`) — remotely toggled |
| Build / Deploy | `run.sh` (local), `deploy.sh` + Netlify (web), standard `flutter build` (mobile) |
| Env Config | `.env` (mobile local dev) + `--dart-define` (web/CI) |

---

## 3. Repository Structure

```
lib/
  main.dart              # App entry: init Supabase, Firebase, LINE SDK, providers
  router.dart            # GoRouter config — auth-gated redirects, all routes
  firebase_options.dart  # Generated Firebase config

  models/                # Pure data classes — no Flutter deps
    models.dart          # Barrel export for all models
    bill/                # Bill, BillItem, BillMember, BillSettings, BillCalculation
    group/               # Group, GroupMember
    friend/              # Friend
    me/                  # Profile
    shared/              # AppConfig, AppNotification

  providers/             # ChangeNotifier providers (auth, theme, locale, notifications)
    auth_provider.dart   # Auth state, profile loading, social sign-in delegation
    theme_provider.dart  # Dark/light toggle, persisted via SharedPreferences
    locale_provider.dart # TH/EN locale, synced from profile
    notifications_provider.dart  # In-app notification list + unread count

  stores/                # ChangeNotifier stores — in-memory cache + optimistic updates
    bills_store.dart     # Single source of truth for all bills; Realtime subscriptions
    groups_store.dart    # Groups + group members
    friends_store.dart   # Friends list + pending requests

  repositories/          # Pure Supabase I/O — no state, no Flutter deps
    bills_repository.dart
    groups_repository.dart
    friends_repository.dart

  services/              # Stateless helpers and platform-conditional services
    app_config_service.dart      # Remote feature flags (ads_enabled) from Supabase
    social_auth_service.dart     # Google + LINE sign-in orchestration
    profile_repository.dart      # Profile CRUD (used by AuthProvider)
    push_notification_service.dart  # FCM token save/clear, notification handling
    line_web_auth_service.dart   # LINE PKCE web flow
    line_web_platform.dart       # Conditional import facade (web vs stub)
    google_web_button.dart       # Conditional import facade (web vs stub)
    ios_install_prompt.dart      # Conditional import facade (web vs stub)

  screens/               # Full-page widgets, one per route
    shared/              # login, onboarding, main_shell, line_web_return
    home/                # HomeScreen
    bill/                # BillsScreen, BillDetailScreen, CreateBillScreen
    group/               # GroupsScreen, GroupDetailScreen, CreateGroupScreen
    friend/              # FriendsScreen, NotificationsScreen
    me/                  # MeScreen, ProfileScreen

  widgets/               # Reusable sub-widgets, co-located with their screen
    home/                # HeroBalanceCard, StatsGrid, RecentBillsList, etc.
    bill/                # AnalyticsTab, analytics_tab/ sub-widgets
    friends/             # FriendRow, AddFriendPanel, PendingRequestsCard, etc.
    group/               # Group-specific widgets
    me/                  # ProfileHeader, SettingsTile, PasswordField, etc.
    shared/              # EmojiPickerGrid, ToggleCard, BillFormConstants

  theme/
    app_theme.dart       # AppColors, AppGradients, AppSpacing, AppRadii, ThemeColors, AppTheme

  utils/
    bill_utils.dart      # calculateBill(), simplifyDebts(), formatCurrency(), formatDate(), etc.

assets/
  i18n/en.json           # English strings
  i18n/th.json           # Thai strings
  images/                # logo, google-logo, line-logo

supabase/
  schema.sql             # Full DB schema (authoritative — run to recreate from scratch)
  migrations/            # Incremental SQL migrations
  functions/
    line-auth/           # Edge Function: LINE PKCE token exchange
    send-push/           # Edge Function: FCM push dispatch

web/
  index.html             # PWA shell — env vars injected by scripts/inject_web_env.sh
  firebase-messaging-sw.js  # FCM service worker

tasks/                   # Feature task specs (TASK_01 … TASK_09)
```

---

## 4. Architecture

### Overall Pattern
4-layer architecture: **Models → Repositories → Stores/Providers → Screens/Widgets**

### Data Flow
1. **Supabase** is the single backend. All reads/writes go through `supabase_flutter`.
2. **Repositories** (`lib/repositories/`) contain only raw Supabase queries — no state, no `ChangeNotifier`.
3. **Stores** (`lib/stores/`) hold in-memory maps (e.g. `Map<String, Bill> _byId`), call repositories, apply **optimistic updates** (mutate local state immediately, persist in background, roll back on failure), and subscribe to **Supabase Realtime** channels.
4. **Providers** (`lib/providers/`) handle cross-cutting concerns: auth lifecycle, theme, locale, notifications.
5. **Screens** consume stores/providers via `context.watch`, `context.select`, or `context.read`. They never call Supabase directly.
6. **Widgets** are dumb — they receive data and callbacks as constructor parameters.

### State Management
- `MultiProvider` at root wraps: `ThemeProvider`, `AuthProvider`, `BillsStore`, `FriendsStore`, `NotificationsProvider`, `GroupsStore`, `LocaleProvider`.
- Stores use `context.select` / `Selector` for narrow rebuilds to avoid unnecessary widget churn.
- Cached computed lists in `BillsStore` use `listEquals` to preserve reference identity when content is unchanged.

### Routing
- `go_router` with `StatefulShellRoute.indexedStack` for the 5-tab bottom nav (`/home`, `/bills`, `/groups`, `/friends`, `/me`).
- Auth-gated redirect logic in `AppRouter.router()` using `_AuthRouteState` enum: `loading → /splash`, `unauthenticated → /login`, `needsOnboarding → /onboarding`, `ready → /home`.
- `refreshListenable: authProvider` — router re-evaluates redirects on every auth state change.
- Static routes (`/bills/create`, `/groups/create`) are declared **before** wildcard routes (`/bills/:id`).

### Authentication
- **Email/password**: Supabase Auth `signInWithPassword` / `signUp` + OTP verification.
- **Google**: Native `google_sign_in` on mobile; GIS `renderButton()` on web (popup flow deprecated).
- **LINE**: Native `flutter_line_sdk` on mobile; custom PKCE OAuth2 web flow with iOS PWA handoff via `line_login_handoffs` DB table (iOS 17+ does not share localStorage between PWA and Safari).
- **Apple**: `sign_in_with_apple` (mobile only).
- Profile is auto-created by a Supabase DB trigger (`handle_new_user`) on `auth.users` INSERT.
- `AuthProvider` delegates social sign-in to `SocialAuthService` and profile CRUD to `ProfileRepository`.
- On sign-out: FCM token is cleared, stores are reset (`clear()`).

### Realtime
- `BillsStore.subscribeRealtime()` opens 3 Postgres change channels: `bills`, `bill_members`, `bill_items`.
- Changes are debounced (400 ms per-bill, 800 ms full reload) to avoid hammering the DB on rapid edits.
- Called after profile loads; unsubscribed on sign-out.

### Platform-Conditional Services
Services that differ between web and native use the conditional import pattern:
```
lib/services/foo.dart          # Conditional import facade
lib/services/foo_web.dart      # Web implementation
lib/services/foo_stub.dart     # Mobile/desktop stub
```
Examples: `line_web_platform`, `google_web_button`, `ios_install_prompt`.

### Environment Variables
- **Mobile/desktop local dev**: loaded from `.env` via `flutter_dotenv`.
- **Web**: injected as `--dart-define` at build time (Netlify env vars) or via `run.sh` locally. `.env` is never loaded on web to avoid exposing secrets at `/assets/.env`.
- Required vars: `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `LINE_CHANNEL_ID`.

---

## 5. Coding Conventions

### Naming
- Files: `snake_case.dart`
- Classes: `PascalCase`
- Variables/methods: `camelCase`
- Private fields: `_camelCase`
- Constants: `camelCase` (not `SCREAMING_SNAKE`)
- DB columns: `snake_case` (Supabase convention); Dart model fields: `camelCase` with `fromJson`/`toJson` mapping.

### File Organization
- Screens own their route-level widget. Complex screens decompose into sub-widgets in `lib/widgets/<feature>/`.
- Widget folders export via `index.dart` barrel files (e.g. `lib/widgets/home/index.dart`).
- Models are pure Dart — no Flutter imports. Each model has `fromJson`, `toJson`, `copyWith`.
- All models are re-exported from `lib/models/models.dart`.

### Error Handling
- Repositories let exceptions propagate to stores.
- Stores catch exceptions, log with `debugPrint('[StoreName.method]: $e')`, and roll back optimistic updates.
- Auth methods return `String?` (error message) or `null` (success) — never throw to the UI.
- Error strings are in Thai (e.g. `'เกิดข้อผิดพลาด กรุณาลองใหม่'`).

### Logging
- `debugPrint('[ClassName.method]: $e')` — always prefixed with class and method name.
- No external logging library detected.

### Optimistic Updates Pattern (Stores)
```dart
final prev = _byId[id];          // 1. Save previous state
_byId[id] = updated;             // 2. Apply optimistic change
_notifyAndInvalidate();          // 3. Notify UI immediately
try {
  await _repo.update(...);       // 4. Persist to Supabase
} catch (e) {
  _byId[id] = prev;              // 5. Roll back on failure
  _notifyAndInvalidate();
  rethrow;
}
```

### Theme Usage
- Always use `AppColors`, `AppSpacing`, `AppRadii`, `AppGradients` from `lib/theme/app_theme.dart`.
- For dark/light adaptive colors, use `ThemeColors.bg(isDark)`, `ThemeColors.surface(isDark)`, etc.
- Font: `GoogleFonts.notoSansThai(...)` for body text; `GoogleFonts.anuphan(...)` for display/headline.
- No hardcoded hex colors in widgets — use `AppColors.*` constants.

### i18n
- Strings are in `assets/i18n/th.json` and `assets/i18n/en.json`.
- Loaded via `LocaleProvider`. Access via the locale provider — not hardcoded in widgets.
- Thai is the default locale. Many UI strings are still hardcoded in Thai in the current codebase.

---

## 6. Important Modules

| Module | Location | Description |
|---|---|---|
| Bill Calculation | `lib/utils/bill_utils.dart` | `calculateBill()` computes subtotal, VAT, service, tip, discount, per-member shares. `simplifyDebts()` runs minimum-transactions algorithm. |
| Bills Store | `lib/stores/bills_store.dart` | Single in-memory source of truth for all bills. Optimistic CRUD, Realtime subscriptions, debounced reloads. |
| Auth Provider | `lib/providers/auth_provider.dart` | Auth lifecycle, profile loading, social sign-in delegation, onboarding gate. |
| Social Auth | `lib/services/social_auth_service.dart` | Google + LINE sign-in, LINE web PKCE flow, iOS PWA handoff polling. |
| Router | `lib/router.dart` | All routes, auth-gated redirects, `_AuthRouteState` enum. |
| App Theme | `lib/theme/app_theme.dart` | All design tokens: colors, spacing, radii, gradients, text theme. |
| Push Notifications | `lib/services/push_notification_service.dart` | FCM token management, foreground/background notification handling. |
| App Config | `lib/services/app_config_service.dart` | Remote feature flags from `app_config` table (e.g. `ads_enabled`). |
| Groups Store | `lib/stores/groups_store.dart` | Groups + members CRUD, invite flow. |
| Friends Store | `lib/stores/friends_store.dart` | Friend requests, accept/decline, friends list. |

---

## 7. Important Models

### `Bill`
- Fields: `id`, `title`, `emoji`, `tags[]`, `status` (`draft`|`pending_payment`|`completed`), `ownerId`, `groupId`, `groupName`, `groupEmoji`, `settings` (BillSettings), `paidMemberIds[]`, `members[]` (BillMember), `items[]` (BillItem), `createdAt`, `updatedAt`
- Computed getters: `isCompleted`, `isPendingPayment`, `isDraft`

### `BillSettings`
- Fields: `serviceCharge` (%), `vat` (%), `tip` (flat), `discount` (flat), `currency` (default `'THB'`), `isVat` (bool), `isService` (bool)
- Stored as JSONB in `bills.settings`

### `BillMember`
- Fields: `id`, `billId`, `userId` (nullable — null = external/guest), `name`, `color` (hex), `promptpay`, `isExternal`
- Linked profile data (avatar_url) is attached in-memory, not stored in `bill_members`

### `BillItem`
- Fields: `id`, `billId`, `name`, `price`, `quantity`, `memberIds[]`, `customShares` (Map<memberId, weight>), `paidBy` (memberId)
- `splitWeights` getter: returns `customShares` if non-empty, else equal weight for each `memberIds` entry

### `BillCalculation`
- Computed result of `calculateBill()`: `subtotal`, `serviceAmount`, `vatAmount`, `tipAmount`, `discountAmount`, `total`, `memberSummaries[]`
- `MemberSummary`: `member`, `total`, `items[]` (MemberItemShare)
- `DebtTransaction`: `from` (BillMember), `to` (BillMember), `amount`

### `Profile`
- Fields: `id`, `username`, `displayName`, `avatarUrl`, `promptpay`, `onboardingCompleted`, `fcmToken`, `locale` (`'th'`|`'en'`), `createdAt`, `updatedAt`

### `Group`
- Fields: `id`, `name`, `description`, `emoji`, `tags[]`, `ownerId`, `members[]` (GroupMember), `createdAt`, `updatedAt`

### `GroupMember`
- Fields: `id`, `groupId`, `userId`, `role` (`'owner'`|`'member'`), `status` (`'pending'`|`'accepted'`|`'declined'`), `invitedBy`

### `Friend`
- Fields: `id`, `requesterId`, `addresseeId`, `status` (`'pending'`|`'accepted'`|`'declined'`), `createdAt`
- Plus joined profile data: `displayName`, `username`, `avatarUrl`

### `AppNotification`
- Fields: `id`, `userId`, `type` (`group_invite`|`friend_request`|`friend_accepted`|`bill_paid`|`bill_completed`), `data` (JSONB), `read`, `createdAt`

---

## 8. APIs

### Supabase (Primary Backend)
- **Base URL**: configured via `SUPABASE_URL` env var
- **Auth**: Supabase anon key (`SUPABASE_ANON_KEY`) for client; service role key for Edge Functions
- **Client access**: `Supabase.instance.client` — available globally after `Supabase.initialize()`
- **Auth flow**: `AuthFlowType.implicit` (configured in `main.dart`)
- **RLS**: All tables have Row Level Security enabled. Key helper functions: `is_group_member()`, `is_group_owner()`, `can_access_bill()` (all `SECURITY DEFINER` to avoid recursion)
- **Realtime**: Postgres change subscriptions on `bills`, `bill_members`, `bill_items` tables

### Supabase Edge Functions
| Function | Trigger | Purpose |
|---|---|---|
| `line-auth` | HTTP POST | LINE PKCE token exchange — exchanges `code` for LINE access token, then signs into Supabase |
| `send-push` | DB trigger on `notifications` INSERT | Sends FCM push notification via Firebase Admin SDK |

### Repository Query Pattern
Repositories use Supabase's PostgREST client with embedded selects:
```dart
// Bills with nested members, items, and group info
'*, bill_members(*, profiles(id, username, display_name, avatar_url)), bill_items(*), groups!bills_group_id_fkey(id, name, emoji)'
```
No REST base URL is called directly — all access is through `supabase_flutter`.

---

## 9. Development Workflow

### Adding a New Feature
1. **Model**: Add/extend a model in `lib/models/<feature>/`. Export from `lib/models/models.dart`.
2. **Repository**: Add Supabase queries to the relevant repository in `lib/repositories/`. Keep it pure I/O — no state.
3. **Store/Provider**: Add methods to the relevant store in `lib/stores/`. Follow the optimistic update pattern. Call `_notifyAndInvalidate()` after mutations.
4. **Screen**: Add a new screen in `lib/screens/<feature>/`. Register the route in `lib/router.dart`.
5. **Widgets**: Extract reusable sub-widgets into `lib/widgets/<feature>/`. Add to `index.dart` barrel if one exists.
6. **DB changes**: Write a migration in `supabase/migrations/` with a timestamp prefix. Update `supabase/schema.sql` to reflect the new state.

### Where New Code Belongs
- Business logic / calculations → `lib/utils/`
- Supabase queries → `lib/repositories/`
- In-memory state + mutations → `lib/stores/` or `lib/providers/`
- Platform-specific code → conditional import pattern in `lib/services/`
- Design tokens → `lib/theme/app_theme.dart`
- Shared UI components → `lib/widgets/shared/`

### Running Locally
- **Mobile**: `flutter run` (loads `.env` automatically)
- **Web**: `./run.sh` (injects `--dart-define` from `.env`)
- **Deploy web**: `./deploy.sh` (builds + deploys to Netlify)

### Testing
- Unit tests in `test/` — currently `bill_utils_test.dart` covers `calculateBill()` and `simplifyDebts()`.
- Run: `flutter test`

---

## 10. AI Instructions

- **Read this file first** before making any changes to the codebase.
- **Follow the existing 4-layer architecture**: Models → Repositories → Stores → Screens/Widgets.
- **Never call Supabase directly from screens or widgets** — always go through a store or provider.
- **Use the optimistic update pattern** for all store mutations (save prev, apply, persist, roll back on error).
- **Prefer modifying existing patterns** over creating new abstractions.
- **Avoid unnecessary refactoring** — do not rename, reorganize, or restructure code unless explicitly asked.
- **Keep changes minimal and consistent** with the surrounding code style.
- **Reuse existing utilities** — check `lib/utils/bill_utils.dart` before writing new calculation or formatting logic.
- **Use `AppColors`, `AppSpacing`, `AppRadii`** — never hardcode colors, sizes, or radii in widgets.
- **Preserve backward compatibility** unless explicitly requested to break it.
- **Search the repository** before implementing any new functionality to avoid duplication.
- **For platform-specific code**, use the conditional import facade pattern (`foo.dart` / `foo_web.dart` / `foo_stub.dart`).
- **For DB changes**, write a migration file AND update `supabase/schema.sql`.
- **Error messages** in Thai for user-facing strings; English for `debugPrint` logs.
- **Explain breaking changes** (RLS policy changes, schema changes, store API changes) before applying them.
- **Do not add new dependencies** without confirming with the user first.

---
> Source: [PiriyaDEV/kidtang](https://github.com/PiriyaDEV/kidtang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
