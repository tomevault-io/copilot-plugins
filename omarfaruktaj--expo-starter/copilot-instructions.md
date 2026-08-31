## expo-starter

> > The single source of truth for any AI coding agent (Codex, Claude Code,

# AGENTS.md

> The single source of truth for any AI coding agent (Codex, Claude Code,
> Cursor, GitHub Copilot, etc.) working in this repository. Read this **first**,
> before writing or modifying any code.

This template is **expo-starter** — a production-ready Expo SDK 57 starter.
The goal of this document is to let you make correct, consistent changes
without guessing where code belongs or inventing new patterns.

Companion documents (read the relevant ones before non-trivial work):

| Doc                                                                    | When to read                    |
| ---------------------------------------------------------------------- | ------------------------------- |
| [`docs/AI_QUICK_REFERENCE.md`](./docs/AI_QUICK_REFERENCE.md)           | Always — the 1-page cheat sheet |
| [`docs/AI_ARCHITECTURE.md`](./docs/AI_ARCHITECTURE.md)                 | Before any structural change    |
| [`docs/AI_DECISION_RULES.md`](./docs/AI_DECISION_RULES.md)             | When deciding where code goes   |
| [`docs/AI_FEATURE_GUIDE.md`](./docs/AI_FEATURE_GUIDE.md)               | When adding a feature           |
| [`docs/AI_DATA_GUIDE.md`](./docs/AI_DATA_GUIDE.md)                     | When integrating an API         |
| [`docs/AI_UI_GUIDE.md`](./docs/AI_UI_GUIDE.md)                         | When building UI                |
| [`docs/AI_NAVIGATION_GUIDE.md`](./docs/AI_NAVIGATION_GUIDE.md)         | When adding screens/routes      |
| [`docs/AI_TESTING_GUIDE.md`](./docs/AI_TESTING_GUIDE.md)               | When writing tests              |
| [`docs/AI_SECURITY.md`](./docs/AI_SECURITY.md)                         | When touching auth/env/storage  |
| [`docs/AI_PRODUCTION_READINESS.md`](./docs/AI_PRODUCTION_READINESS.md) | Before declaring done           |
| [`docs/AI_PRODUCTION_AUDIT.md`](./docs/AI_PRODUCTION_AUDIT.md)         | Final audit                     |
| [`docs/AI_FEATURE_CHECKLIST.md`](./docs/AI_FEATURE_CHECKLIST.md)       | Per-feature checklist           |
| [`docs/AI_CONTEXT_MAP.md`](./docs/AI_CONTEXT_MAP.md)                   | Task → file map                 |
| [`docs/AI_MODULES.md`](./docs/AI_MODULES.md)                           | Opt-in module deep-dive         |
| [`docs/DEFINITION_OF_DONE.md`](./docs/DEFINITION_OF_DONE.md)           | "Done" definition               |
| [`docs/PROJECT_CONSTITUTION.md`](./docs/PROJECT_CONSTITUTION.md)       | Non-negotiable principles       |
| [`docs/AI_APP_BUILD_WORKFLOW.md`](./docs/AI_APP_BUILD_WORKFLOW.md)     | Building a whole app            |

> `docs/AI_MODULES.md` is produced by task DOCS-B; if the link 404s when you
> read this, it is being written in parallel — the entry paths below remain
> authoritative.

---

## 1. Project purpose

A modern, scalable, **production-ready Expo SDK 57 starter template** that an
AI agent (or human) can take and turn into a real application. It is **not** a
tutorial app and **not** a feature grab-bag — it is the minimal, opinionated
foundation (auth, data, forms, theming, i18n, navigation, dev tooling) that
every serious RN app needs, wired together consistently.

Built on the official Expo SDK 57 default template, with the most useful
architecture and patterns from the [Obytes RN template][obytes] adapted and
modernized — **not a copy** of either.

[obytes]: https://github.com/obytes/react-native-template-obytes

## 2. Tech stack

| Concern                  | Choice                                                                                                                                             | Notes                                                                                                   |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Framework                | Expo SDK 57 / React Native 0.86 / React 19                                                                                                         | `expo ~57.0.14`                                                                                         |
| Routing                  | Expo Router 7 (typed routes)                                                                                                                       | `experiments.typedRoutes: true`                                                                         |
| Language                 | TypeScript 6 (strict)                                                                                                                              | `extends: expo/tsconfig.base`                                                                           |
| Styling                  | `StyleSheet.create` + typed design tokens                                                                                                          | **No NativeWind/Tailwind.**                                                                             |
| Client state             | Zustand 5 + `createSelectors` helper                                                                                                               |                                                                                                         |
| Server state             | TanStack React Query 5                                                                                                                             |                                                                                                         |
| HTTP                     | Dependency-free `fetch` client (`apiClient`)                                                                                                       | **No axios.** Auth injection + 401 sign-out                                                             |
| Forms                    | React Hook Form + Zod 4 + `@hookform/resolvers` (+ `useZodForm`, `<FormField>`)                                                                    |                                                                                                         |
| i18n                     | i18next 26 + react-i18next 17                                                                                                                      | Typed translation keys                                                                                  |
| Secure storage           | `expo-secure-store` (tokens)                                                                                                                       | Keychain/Keystore                                                                                       |
| Prefs storage            | `@react-native-async-storage/async-storage` (~2.2.0)                                                                                               | Non-sensitive prefs                                                                                     |
| Icons                    | `expo-symbols`                                                                                                                                     | SF / Material symbols                                                                                   |
| Animations               | `react-native-reanimated` 4                                                                                                                        |                                                                                                         |
| Errors/observability     | `AppError` + `ErrorBoundary` (in-repo class) + `logger` + `ErrorReporter` interface                                                                | Barrel: `src/lib/errors/`, `src/lib/logging/logger.ts`. Sentry/Bugsnag via `setErrorReporter()`         |
| Auth (provider-agnostic) | `AuthService` interface + `DemoAuthService` (dummyjson) + `setAuthService()`                                                                       | Tokens persist via `expo-secure-store`. `src/features/auth/auth-service.ts` (replaces the old `api.ts`) |
| Feature flags            | `FeatureFlagsProvider` interface + `LocalFeatureFlags` (per-env map + dev overrides)                                                               | `src/lib/feature-flags/`; no vendor                                                                     |
| Analytics                | `AnalyticsProvider` interface + `NoopAnalyticsProvider` + `ConsoleAnalyticsProvider`                                                               | `src/lib/analytics/`; no vendor                                                                         |
| Notifications            | `NotificationService` interface + `ExpoNotificationsService` + `setNotificationsProvider()`                                                        | `src/lib/notifications/`; opt-in, web-guarded                                                           |
| Offline                  | `useNetworkStatus()` + `MutationQueue` (AsyncStorage) + `useQueuedMutation()`                                                                      | `src/lib/offline/`; opt-in                                                                              |
| Media                    | `useImagePicker()` (gated on permissions) + `UploadClient` interface + `FetchUploadClient` (XHR+progress+cancel)                                   | `src/lib/media/`; opt-in                                                                                |
| AI (backend-proxied)     | `AiClient` interface + `BackendAiClient` (calls the **app backend**, never the AI provider) + Zod-validated `generateStructuredOutput` + streaming | `src/lib/ai/`; opt-in; provider secrets live on the backend                                             |
| Tests                    | Jest 29 + jest-expo + Testing Library RN                                                                                                           |                                                                                                         |
| Build/ship               | EAS Build / Submit                                                                                                                                 | dev / preview / staging / production profiles                                                           |
| Tooling                  | Bun, ESLint flat (`eslint-config-expo`), Prettier, Husky, lint-staged, commitlint                                                                  |                                                                                                         |
| CI                       | GitHub Actions                                                                                                                                     | lint + type-check + test + doctor                                                                       |

New SDK-57-compatible deps added (no vendor secrets shipped):

| Package                           | Version    | Used by                          |
| --------------------------------- | ---------- | -------------------------------- |
| `expo-notifications`              | `~57.0.12` | `src/lib/notifications` (opt-in) |
| `@react-native-community/netinfo` | `^12.0.1`  | `src/lib/offline` (opt-in)       |
| `expo-image-picker`               | `~57.0.11` | `src/lib/media` (opt-in)         |
| `expo-file-system`                | `~57.0.4`  | `src/lib/media` (opt-in)         |

`react-error-boundary` was **not** added — the in-repo `ErrorBoundary` class
(`src/lib/errors/error-boundary.tsx`) is used instead (no extra dep for a
~40-line component).

Package manager: **Bun**. All commands are `bun run <script>`.

## 3. Architecture

**Feature-first.** Each product area owns its screens, components, API and
types behind a single folder (`src/features/<name>/`). Cross-cutting
infrastructure lives in `src/lib/`. Reusable, feature-agnostic UI primitives
live in `src/components/ui/`. Routing is file-based via Expo Router in
`src/app/`, where each route file is a **thin re-export** of a feature screen
(never business logic in route files).

See [`docs/AI_ARCHITECTURE.md`](./docs/AI_ARCHITECTURE.md) for the
directory-by-directory guide.

## 4. Directory structure

```
expo-starter/
├── app.config.ts            # env-driven Expo config (NOT app.json) — notifications plugin
├── env.ts                   # zod-validated runtime env: APP_ENV ∈ development|preview|staging|production; isDev/isPreview/isStaging/isProd; BUNDLE_IDS
├── eas.json                 # dev / preview / staging / production build profiles
├── tsconfig.json            # strict TS + path aliases (@/*, @env)
├── eslint.config.mjs        # flat ESLint (eslint-config-expo + prettier)
├── jest.config.js           # jest-expo + @ alias mapping
├── jest-setup.ts            # native-module mocks (secure-store, async-storage, netinfo, expo-image-picker, expo-file-system, expo-notifications…)
├── .husky/                  # pre-commit (lint-staged) + commit-msg (commitlint)
├── .github/workflows/ci.yml # lint + type-check + test + doctor
├── assets/                  # icons, splash, adaptive icon
├── docs/                    # ← this AI instruction system (incl. AI_MODULES.md)
└── src/
    ├── app/                 # Expo Router routes (thin re-exports)
    │   ├── _layout.tsx      #   root providers + bootstrap gate (splash) + <ErrorBoundary>
    │   ├── login.tsx        #   /login
    │   ├── sign-up.tsx      #   /sign-up
    │   ├── +not-found.tsx   #   404
    │   └── (app)/           #   authenticated group (Tabs)
    │       ├── _layout.tsx  #   Tabs + auth redirect
    │       ├── index.tsx    #   /        → Home
    │       ├── feed.tsx     #   /feed    → Feed (react-query demo)
    │       └── settings.tsx #   /settings
    ├── components/
    │   └── ui/              # design system (Text, Button, Card, Input, Switch, Badge, Avatar, Divider, Skeleton, TextArea, Select, Checkbox, RadioGroup, IconButton, Modal, Alert, BottomSheet, FormField, …)
    ├── config/              # STORAGE_KEYS (+OFFLINE_QUEUE, +FEATURE_FLAGS_OVERRIDES), SECURE_STORE_KEYS, QUERY_KEYS (+ai), NETWORK, FEATURE_FLAGS typed map
    ├── constants/theme.ts   # design tokens (colors, fonts, spacing, radius)
    ├── features/            # feature modules (auth + auth-service, feed, home, settings)
    ├── hooks/               # use-color-scheme, use-theme-mode, use-theme
    ├── lib/
    │   ├── api/             # fetch client, query provider, types (ApiError), validate.ts (Zod), pagination.ts
    │   ├── auth/storage.ts  # secure token get/set/clear
    │   ├── errors/          # app-error.ts, error-boundary.tsx, error-reporter.ts, index.ts
    │   ├── logging/logger.ts # dev-gated logger
    │   ├── forms/           # use-zod-form.ts (typed RHF+Zod wrapper)
    │   ├── i18n/            # i18next init, resources, storage, typed keys
    │   ├── storage/         # AsyncStorage wrapper (non-sensitive prefs)
    │   ├── offline/         # network.ts, mutation-queue.ts, offline-indicator.tsx, index.ts (opt-in)
    │   ├── notifications/   # notifications.ts, hooks.ts, index.ts (opt-in)
    │   ├── permissions/     # permissions.ts, hooks.ts, index.ts (opt-in)
    │   ├── media/           # image-picker.ts, upload.ts, file-system.ts, index.ts (opt-in)
    │   ├── analytics/       # analytics.ts, hooks.ts, index.ts (opt-in, no vendor)
    │   ├── feature-flags/   # feature-flags.ts, hooks.ts, index.ts (opt-in, no vendor)
    │   ├── ai/              # ai-client.ts, errors.ts, hooks.ts, index.ts (opt-in, backend-proxied)
    │   ├── __tests__/       # unit tests for lib/utils, lib/feature-flags
    │   ├── test-utils.tsx   # customRender (providers wired)
    │   └── utils.ts         # cn(), createSelectors(), formatDate(), sleep()
    └── translations/        # en.json, es.json
```

## 5. Repository exploration process (MANDATORY before significant changes)

Never start implementing immediately after reading only the user's request.
Follow this order:

1. **Read this `AGENTS.md`.**
2. **Read the relevant companion doc** (Context Map → which doc).
3. **Inspect the existing implementation** — read the files you'll touch and
   their neighbors.
4. **Search for similar functionality** (`features/`, `components/ui/`, `lib/`).
5. **Reuse existing patterns** — prefer extending over creating.
6. **Plan the smallest architectural change.**
7. **Implement.**
8. **Add tests** for new business logic.
9. **Run validation commands** (see §22).
10. **Review the final diff** against §6 "DO NOT" and `docs/DEFINITION_OF_DONE.md`.

## 6. Core conventions

### Imports & path aliases

- `@/*` → `src/*`, `@env` → `env.ts` (configured in `tsconfig.json` and
  `jest.config.js`).
- Group imports: external packages → `@/`-aliased → relative. ESLint enforces.
- **Never** import `env.ts` from `app.config.ts` — the config runs in a Node
  context that is not bundled by Metro. Read `process.env` inline in
  `app.config.ts`.

### File naming

- Screens: `<name>-screen.tsx` (default export).
- Components: kebab-case `<name>.tsx`.
- Hooks: `use-<name>.ts(x)`.
- Zustand stores: `use-<name>-store.ts`.
- Tests: `<name>.test.ts(x)` next to the code.

### TypeScript

- **Strict mode is mandatory.** Never disable it to make code compile.
- Never use `any` to bypass typing; if unavoidable, add an inline
  `eslint-disable-next-line @typescript-eslint/no-explicit-any` with a comment
  explaining why (the only justified current use is `cn()`'s style merger in
  `src/lib/utils.ts`).
- Validate external data (API responses, env, deep-link params) at the
  boundary with Zod.
- Prefer `type` for aliases and `interface` for extensible props.

### Styling & theming

- Use **`StyleSheet.create` + typed tokens** from `src/constants/theme.ts`.
- Use the `cn()` helper (`src/lib/utils.ts`) to merge styles conditionally.
- **Do not** add NativeWind, Tailwind RN, Restyle, or similar.
- Colors: read from the active palette via `useTheme()` —
  `const { colors } = useTheme()`; reference semantic tokens
  (`colors.primary`, `colors.mutedForeground`, …), never hardcode hex in
  components (the one exception is the splash background in `app.config.ts`).
- Light/dark is automatic via `useResolvedColorScheme()` (system or persisted
  user override). Persisted preference is handled by `src/hooks/use-theme-mode.ts`.

### State management

- **Client state:** Zustand, wrapped with `createSelectors()` so fields are read
  via `useStore.use.<field>()`. Mutate via `store.getState().<action>()` or
  `store.setState(...)`.
- **Server state:** React Query (`useQuery`/`useMutation`). **Never** mirror
  server data into Zustand.
- **Local UI state:** `useState`/`useReducer` inside the component. Do **not**
  create a global store for screen-local state.
- **URL state:** use route params / query via Expo Router (`useLocalSearchParams`,
  `useGlobalSearchParams`).

### API & data-fetching

- All HTTP goes through `apiClient` (`src/lib/api/client.ts`). **Never** call
  `fetch` directly from a screen or component; **never** add axios. This
  includes AI calls — the `BackendAiClient` (`src/lib/ai/ai-client.ts`) calls
  the **app backend** through `apiClient` (bearer injected, 401 handled). It
  never talks to the AI provider directly; provider secrets live on the
  backend.
- API hooks live in `src/features/<name>/api.ts` as `useQuery`/`useMutation`.
  Query keys in `src/config/index.ts` (`QUERY_KEYS`, incl. `QUERY_KEYS.ai`).
- Validate untrusted API responses at the boundary with
  `validateResponse(schema, data)` from `src/lib/api/validate.ts` (Zod; throws
  `AppError({ kind: 'validation' })` on a bad shape; `safeValidate` returns a
  `SafeParseReturnType`). Paginated endpoints use `PaginatedResponse<T>` +
  `getNextPageParam` + `pagedQuery` from `src/lib/api/pagination.ts`. The
  `RequestOptions` now accepts `signal?: AbortSignal` and `client.ts`
  forwards an external `signal` to `fetch` (React Query cancellation works
  end-to-end).
- The client injects the bearer token and calls the registered 401 handler
  (which signs the user out via `AuthService.signOut()`) — don't re-implement
  auth in features.
- See [`docs/AI_DATA_GUIDE.md`](./docs/AI_DATA_GUIDE.md) and
  [`docs/AI_MODULES.md`](./docs/AI_MODULES.md).

### Forms & validation

- React Hook Form + `zodResolver` + the `<Input>` UI component (wired via
  `<Controller>`, forwarding `value`/`onChangeText`). For typed forms across
  multiple fields, use the `useZodForm({ schema, defaultValues })` wrapper
  (`src/lib/forms/use-zod-form.ts`) and the `<FormField as="input|text-area|select|checkbox|switch" control name label helper />`
  primitive (`src/components/ui/form-field.tsx`).
- Define the Zod schema inside the form module.
- See `src/features/auth/login-form.tsx` and `sign-up-form.tsx` for canonical
  examples.

### Navigation

- Expo Router file-based. Route files in `src/app/` are **thin re-exports** of
  feature screens (`export { default } from '@/features/<name>/<name>-screen'`).
- Protected routes: the `(app)/_layout.tsx` returns `<Redirect href="/login" />`
  when `useAuthStore.use.status() !== 'authenticated'`.
- See [`docs/AI_NAVIGATION_GUIDE.md`](./docs/AI_NAVIGATION_GUIDE.md).

### i18n

- `useTranslation()` from react-i18next. Keys are type-checked against
  `src/translations/en.json` (via the augmentation in `src/lib/i18n/types.ts`).
- Add keys to **all** locale files (`en.json`, `es.json`) — the ESLint/lint
  pipeline expects them to stay in sync.
- Hardcoded user-facing strings are not allowed in screens; route through `t()`.

### Error handling

- All errors normalize to `AppError`
  (`src/lib/errors/app-error.ts`) — `kind: network | api | auth | authorization
| validation | offline | unexpected`, plus optional `code`, `statusCode`,
  `data`, `userMessage`. Use `toAppError(error)` to wrap an unknown thrown
  value and `AppError.is(err, kind)` to narrow. `ApiError` (HTTP layer) is one
  input; the 401 handler emits `kind: 'auth'`.
- HTTP errors throw `ApiError` (status + data) — catch in the mutation's
  `onError` (normalize with `toAppError`) or render via the `<ErrorState>`
  component for queries.
- Every list/screen with async data must show **loading, empty, and error**
  states (use `<List>` which handles all three, or compose
  `<Spinner>`/`<EmptyState>`/`<ErrorState>`).
- The root Stack in `src/app/_layout.tsx` is wrapped in `<ErrorBoundary>`
  (in-repo class component, `src/lib/errors/error-boundary.tsx`) with a themed
  fallback that forwards the caught error to the registered `ErrorReporter`
  (`setErrorReporter()` in `src/lib/errors/error-reporter.ts`; default
  `ConsoleErrorReporter`; wire Sentry/Bugsnag here). For risky screens you may
  add a route-local `<ErrorBoundary>` around the screen body.
- Production logs go through `logger` (`src/lib/logging/logger.ts`) —
  `debug`/`info` are dev-gated, `warn`/`error` are always on. **Never**
  `console.log`/`console.error` directly in production code paths.
- Surface user-facing errors via `useToast()` (`ToastProvider`), not `alert`.

### Testing

- Unit-test pure logic and hooks (e.g. `src/lib/__tests__/utils.test.ts`).
- Component-test presentational UI via `customRender` (`src/lib/test-utils.tsx`).
- Mock native modules in `jest-setup.ts`; mock API calls per-test with
  `jest.mock` of the feature's `api.ts`.
- See [`docs/AI_TESTING_GUIDE.md`](./docs/AI_TESTING_GUIDE.md).

### Environment & configuration

- Client-exposed vars **must** be prefixed `EXPO_PUBLIC_` and validated by the
  zod schema in `env.ts`. Access via `import { env } from '@env'`. `.env.example`
  documents the **public-vs-secret split** — only non-secret, public values
  use `EXPO_PUBLIC_*`; everything privileged lives on the backend.
- `env.ts` `APP_ENV` enum is `development | preview | staging | production`
  (each with its own `BUNDLE_IDS` entry — staging is
  `com.expostarter.staging` / scheme `expostarter-staging`; `eas.json` has a
  matching `staging` build profile). Convenience flags: `isDev`, `isPreview`,
  `isStaging`, `isProd`.
- Per-environment bundle IDs/schemes live in `BUNDLE_IDS` (defined in **both**
  `app.config.ts` and `env.ts` — keep them in sync).
- `.env` is gitignored; `.env.example` is committed.

### Optional modules (opt-in)

The repo ships provider-agnostic, vendor-free module folders under
`src/lib/`. **They are opt-in: importing a module "turns it on" — no module
loads or initializes another unless you import it.** The only sanctioned
cross-module dependencies are notifications → permissions and media →
permissions (both go through the `PermissionHandler` registry in
`src/lib/permissions`).

| Module        | Entry path              | Default impl                                                                                                                                                                 | Swap point (provider injection)                    |
| ------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| Offline       | `src/lib/offline`       | `MutationQueue` (AsyncStorage), `useNetworkStatus()` (NetInfo)                                                                                                               | n/a (extend the conflict hook)                     |
| Notifications | `src/lib/notifications` | `ExpoNotificationsService` (web-guarded)                                                                                                                                     | `setNotificationsProvider()`                       |
| Permissions   | `src/lib/permissions`   | `notifications` handler registered by default; `camera`/`photoLibrary`/`location`/`microphone` are documented extension points                                               | `registerPermissionHandler()`                      |
| Media         | `src/lib/media`         | `FetchUploadClient` (XHR + progress + AbortController cancel)                                                                                                                | `setUploadClient()`                                |
| Analytics     | `src/lib/analytics`     | `NoopAnalyticsProvider`; `ConsoleAnalyticsProvider` in dev                                                                                                                   | `setAnalyticsProvider()`                           |
| Feature-flags | `src/lib/feature-flags` | `LocalFeatureFlags` (reads `FEATURE_FLAGS` from `src/config/index.ts`, per-env via `env.APP_ENV`; dev-only AsyncStorage overrides in `STORAGE_KEYS.FEATURE_FLAGS_OVERRIDES`) | n/a (remote provider = documented extension point) |
| AI            | `src/lib/ai`            | `BackendAiClient` (calls the **app backend** via `apiClient`; bearer injected; never the AI provider)                                                                        | `setAiClient()` (still backend-only)               |

> **AI provider secrets NEVER live in the client.** `BackendAiClient` calls
> the app backend; the backend holds the provider key. The pipeline is
> AI output → parse → **validate (Zod)** → normalize → use.

See [`docs/AI_MODULES.md`](./docs/AI_MODULES.md) for the per-module
deep-dive.

### Security

- **Never** put secrets in `EXPO_PUBLIC_*` vars or client source. Those are
  baked into the bundle and readable by anyone with the app.
- Auth tokens go in `expo-secure-store` (Keychain/Keystore) via
  `src/lib/auth/storage.ts` (now consumed by `AuthService`).
  **Never** in AsyncStorage.
- AI provider secrets live on the **app backend**, never in the client —
  `BackendAiClient` calls your backend through `apiClient`; the backend holds
  the provider key. (See [`docs/AI_SECURITY.md`](./docs/AI_SECURITY.md) §AI.)
- See [`docs/AI_SECURITY.md`](./docs/AI_SECURITY.md).

### Performance

- Memoize expensive computations (`useMemo`); avoid memoizing trivial values.
- Use `FlashList`/`FlatList` recycling for long lists (the `<List>` component
  wraps `FlatList`).
- Use `expo-image` (already a dep) for remote images — it caches and decodes
  off the main thread.
- Keep server state in React Query; don't duplicate into Zustand.
- Prefer the smallest global-state surface; scope state as locally as possible.

### Accessibility

- Use semantic components (`Pressable` with `accessibilityRole`, `Text`, etc.).
- Every interactive element needs an `accessibilityLabel` (and
  `accessibilityHint` when non-obvious).
- Hit targets ≥ 44×44pt (`hitSlop` for small icons).
- Respect color contrast in the palette; never communicate state by color alone.

### Dependencies

- Before adding any package: (1) check the codebase for an existing solution,
  (2) check Expo/React Native for a built-in, (3) check if an existing dep
  covers it, (4) **prefer an in-repo abstraction/interface over a new dep**
  (`AuthService`, `AnalyticsProvider`, `FeatureFlagsProvider`,
  `NotificationService`, `UploadClient`, `AiClient` all exist as interfaces —
  swap an impl, don't add a vendor SDK when an interface covers the need).
  Only then add a new one.
- Verify Expo SDK 57 compatibility (run `bun run doctor` after).
- Prefer mature, maintained packages. Avoid trivial-functionality deps.
- Pin Expo-managed modules with `~57.0.x`; community modules with the version
  `expo-doctor` accepts.

### Git & commits

- Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`,
  `test:`, `perf:`, `style:`, `build:`, `ci:`, `revert:`). Enforced by
  commitlint on `commit-msg`.
- `pre-commit` runs `lint-staged` (ESLint --fix + Prettier on staged files).
- Keep commits atomic and focused.

## 7. Production-readiness requirements

A change is production-ready only when it satisfies
[`docs/DEFINITION_OF_DONE.md`](./docs/DEFINITION_OF_DONE.md). In short:

- TypeScript passes (`bun run type-check`).
- ESLint passes (`bun run lint`).
- Tests pass (`bun run test`).
- No dead code, no debug logging, no TODO-as-production.
- Loading / empty / error states handled for every async feature.
- Accessibility considered; security reviewed.
- No leaked secrets; valid EAS config; correct env.

## 8. DO NOT (forbidden behaviors)

- **DO NOT** introduce a new state library when Zustand already solves it.
- **DO NOT** create duplicate components — search `src/components/ui/` first.
- **DO NOT** put business logic directly inside large screen components.
- **DO NOT** call `fetch`/HTTP directly from a screen or UI component — use
  `apiClient` + a `useQuery`/`useMutation` hook in the feature's `api.ts`.
  This applies to AI calls too — `BackendAiClient` goes through `apiClient`.
- **DO NOT** put AI provider secrets in the client (no `EXPO_PUBLIC_OPENAI_*`,
  no hardcoded key). `BackendAiClient` calls the app backend; the backend
  holds the provider key.
- **DO NOT** store secrets in source code or `EXPO_PUBLIC_*` vars.
- **DO NOT** disable TypeScript strictness to fix errors.
- **DO NOT** use `any` to bypass typing unless explicitly justified inline.
- **DO NOT** ignore or blindly suppress lint errors.
- **DO NOT** add dependencies without checking Expo SDK 57 compatibility.
- **DO NOT** add a vendor SDK where an in-repo interface exists
  (`AuthService`, `AnalyticsProvider`, `FeatureFlagsProvider`,
  `NotificationService`, `UploadClient`, `AiClient`, `PermissionHandler`,
  `ErrorReporter`). Implement the interface and call `set*()` instead.
- **DO NOT** make optional modules part of the core bundle — they are
  opt-in: importing a module turns it on; not importing keeps the bundle lean.
- **DO NOT** let one opt-in module silently pull another (the only sanctioned
  cross-imports are notifications → permissions and media → permissions).
- **DO NOT** invent a per-screen fetch pattern — all data goes through
  `apiClient` + react-query hooks in the feature's `api.ts`.
- **DO NOT** skip response validation at the boundary — use
  `validateResponse(schema, data)` (Zod) for untrusted API shapes.
- **DO NOT** use `console.log`/`console.error` in production code paths —
  use `logger` (`src/lib/logging/logger.ts`).
- **DO NOT** reference the deleted `src/features/auth/api.ts` — auth now
  lives in `src/features/auth/auth-service.ts` (`AuthService` interface,
  `DemoAuthService` default, `setAuthService()` swap point).
- **DO NOT** rewrite working architecture unnecessarily.
- **DO NOT** create global state for screen-local UI state.
- **DO NOT** copy patterns from outdated examples — read the current code.
- **DO NOT** leave production functionality as `TODO`.
- **DO NOT** remove tests because they're inconvenient.
- **DO NOT** add NativeWind/Tailwind/Restyle — styling is StyleSheet + tokens.
- **DO NOT** add axios — HTTP goes through `apiClient`.
- **DO NOT** mirror server state into Zustand.
- **DO NOT** hardcode user-facing strings — use `t()`.

## 9. Verification commands

Run these before considering work done:

```bash
bun run check-all      # = lint + type-check + test  (fastest sanity check)
bun run lint           # ESLint (flat, expo config)
bun run type-check     # tsc --noEmit
bun run test           # jest
bun run format:check    # prettier --check
bun run doctor         # expo-doctor (SDK/dep compatibility)
bun run preflight      # lint + type-check + test + format:check + doctor + expo config --type prebuild
```

`bun run preflight` is the one-command pre-merge/pre-release gate — it runs
everything `check-all` does plus `format:check`, `doctor`, and
`expo config --type prebuild` (validates `app.config.ts` + plugins resolve).

Before a release: `bunx eas build --profile production --platform ios` (and
android) and `bunx expo config --type prebuild` (validate config loads).

## 10. Adding a feature (quick version)

1. Create `src/features/<name>/` with `<name>-screen.tsx` (default export),
   `api.ts` (react-query hooks), `types.ts`, and a `components/` folder if
   needed.
2. Add a route re-export in `src/app/(app)/<name>.tsx`:
   `export { default } from '@/features/<name>/<name>-screen';`
3. Register the tab/stack screen in `src/app/(app)/_layout.tsx`.
4. Add query keys to `src/config/index.ts` (`QUERY_KEYS`).
5. Add i18n keys to `src/translations/{en,es}.json`.
6. Reuse `src/components/ui/*` — don't build one-off primitives.
7. Add loading/empty/error states; add tests; run `bun run check-all`.

Full process: [`docs/AI_FEATURE_GUIDE.md`](./docs/AI_FEATURE_GUIDE.md).

---
> Source: [omarfaruktaj/expo-starter](https://github.com/omarfaruktaj/expo-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
