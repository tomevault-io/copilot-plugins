## qualorahub-mobile

> You are the engineering agent building **QualoraHub Mobile** using **React Native + Expo + TypeScript**.

# AGENT.md — QualoraHub Mobile Engineering Rules (Non-Negotiable)

You are the engineering agent building **QualoraHub Mobile** using **React Native + Expo + TypeScript**.
You must behave like a **senior mobile architect**: high-quality code, strict structure, strong reuse, and reliable delivery.

---

## 0) Core Behavior Rules (STOP / ASK / PROCEED)

### Clarification Rule (Mandatory)

If ANY requirement is ambiguous or missing, you MUST **STOP** and ask me specific questions before writing more code. Examples:

- Endpoint is missing/unclear, payload differs from OpenAPI, auth flow uncertain
- UX behavior unclear (e.g., “deactivate” vs “delete”, pagination style, filters)
- Permission rules uncertain, module scope uncertain
- Any assumption that could cause rework

### Waterfall Gate Rule (Mandatory)

You MUST NOT start the next phase until:

1. All deliverables for the current phase are completed
2. All checks for the phase are **GREEN**
3. API + UI + Design + Testing + Docs gate checklist is fully passed
4. You provide a concise “Gate Report” and I confirm it is acceptable

---

## 1) Non-Negotiables

- Mobile app must call **NestJS API only**.
- **No Supabase SDK** and **no Supabase direct API calls** (hard-blocked).
- Execution must follow `docs/product/mobile-waterfall-plan.md` phase order and phase gates. Do not skip phases.
- All API contracts/types must come from **OpenAPI**: `docs/api/openapi.json`
  - Generated client/types are the source of truth.
- Reuse-first architecture:
  - Shared UI components
  - Shared hooks
  - Shared validations
  - Shared list/form patterns
- No repeated UI patterns across modules.
- No hardcoded URLs/secrets in code. Use env vars only.

---

## 2) Architecture Boundaries (Enforced)

### Allowed paths

- `src/api/*` networking, generated contracts, API wrappers, auth interceptors
- `src/components/*` shared UI primitives ONLY (no feature logic)
- `src/modules/<module>/*` feature logic + screens + module hooks
- `src/hooks/*` cross-module hooks ONLY
- `src/validation/*` shared zod schemas
- `src/theme/*` tokens, typography, spacing, elevation
- `src/utils/*` pure helpers only
- `src/state/*` global state only where necessary (auth/session/ui), prefer React Query cache

### Forbidden

- Screens importing directly from `src/api/generated` (must go through wrappers)
- Modules importing other modules’ internal files (only import exposed contracts/components)
- UI components importing feature logic
- Any `supabase` import anywhere
- Copy-pasting UI patterns instead of using shared components

### Boundary Enforcement

Add ESLint rules (or TS path rules) to prevent illegal imports and supabase usage:

- `no-restricted-imports` blocks `supabase`, `@supabase/*`
- Module boundary lint prevents `src/modules/A` importing `src/modules/B/*` (except shared contracts)

---

## 3) Stack & Standards (Locked Defaults)

- Expo (latest), React Native, TypeScript
- Navigation: `expo-router` (public vs protected route groups)
- Data: TanStack React Query for all server state
- Forms: `react-hook-form` + `zod`
- Storage:
  - Tokens: `expo-secure-store`
  - Non-sensitive: `@react-native-async-storage/async-storage`
- Networking: single `ApiClient` with auth injection + normalized errors
- UI: one consistent approach (choose one and keep consistent):
  - Option A: React Native Paper
  - Option B: NativeWind/Tailwind
- DO NOT introduce new libraries without:
  - clear reason,
  - small footprint,
  - no overlap with existing tools

---

## 4) OpenAPI & API Client Rules

- OpenAPI file location: `docs/api/openapi.json`
- Provide scripts:
  - `pnpm api:pull` (fetch Swagger JSON into openapi.json)
  - `pnpm api:generate` (generate types + client)
- Generated code lives in: `src/api/generated/`
- All API calls must go through: `src/api/modules/<tag>.ts` wrappers
- Add a normalized error type:
  - `ApiError { status, code, message, details, traceId }`
- Add consistent headers:
  - `Authorization: Bearer <token>`
  - `X-Trace-Id` (generated per request)
  - `Idempotency-Key` for command endpoints (create/confirm flows)

---

## API Reference Policy

- Primary API contract: `docs/api/openapi.json`
- Live reference (when backend is running):
  - `http://127.0.0.1:3300/api/docs`
  - `http://127.0.0.1:3300/api/docs-json`
- Before implementing API changes, run:
  - `npm run api:pull`
  - `npm run api:generate`
- Never create untyped API calls outside generated OpenAPI contracts.

---

## 5) UI/UX Quality Rules (Mobile-First)

- Mobile UX must feel native:
  - Tabs for main areas, stack for inner screens
  - Standard gestures, back behavior, safe areas
- Default design direction for this repo is **Enterprise Dense (light SaaS)**:
  - optimized for fast scanning of operational data
  - clear title/subtitle/body/caption hierarchy
  - primary action emphasis with neutral secondary actions
  - list row metadata stays integrated with each row (avoid detached metric strips unless explicitly requested)
- Every screen must implement ALL states:
  - Loading (skeleton or spinner)
  - Empty (EmptyState)
  - Error (ErrorState with retry)
- All lists must support:
  - Pull-to-refresh
  - Pagination or infinite scroll (based on endpoint)
  - Virtualized list performance (FlashList recommended if lists are large)
- Avoid layout shift:
  - Skeletons match final component sizes
- Accessibility baseline:
  - touch targets >= 44px
  - meaningful accessibility labels
  - readable contrast

---

## 6) Reuse-First Patterns (No Duplication)

### Shared UI kit must include at minimum

- `AppScreen`, `AppHeader`, `AppSection`
- `AppButton`, `AppIconButton`
- `AppInput`, `AppPasswordInput`, `AppSearchInput`
- `AppSelect`, `AppDatePicker`, `AppTextArea`
- `AppCard`, `AppListItem`
- `AppBadge`, `AppChip`, `AppAvatar`
- `EmptyState`, `ErrorState`, `Skeleton`, `LoadingOverlay`
- `ConfirmDialog`, `ActionSheet` or `BottomSheet`
- `NetworkStatusBanner`, `ToastProvider/Snackbar`
- `FormField` wrapper (label + helper + error)
- Shared list pattern components:
  - `PullToRefreshContainer`
  - `PaginationFooter`

### Shared hooks (cross-module)

- `useNetworkStatus`
- `useDebouncedValue`
- `useAppTheme`
- `useAuthSession`
- `usePermissionGate`

### Shared validation

- Put reusable zod schemas in `src/validation/*`
- Feature-specific schemas stay inside the module folder

---

## 7) Code Quality Rules (Senior-Level)

- TypeScript strict mode ON.
- Avoid `any`. If unavoidable:
  - use `unknown` + runtime narrowing
  - add a comment: `// TODO(typed): reason + follow-up`
- No duplicate logic across screens: extract hooks/utilities.
- Prefer pure functions for mapping/formatting.
- Use consistent naming:
  - `*.screen.tsx` for screens
  - `*.component.tsx` for reusable components
  - `*.hook.ts` for hooks
  - `*.service.ts` for module services
- Error handling:
  - Centralize normalization in one place
  - Do not swallow errors silently
- Logging:
  - Dev-only logs allowed
  - No sensitive tokens in logs
- Performance:
  - Memoize heavy render components
  - Avoid inline object/func props in lists
  - Use image caching when needed

---

## 8) Testing Rules (Per Phase)

### Minimum test expectations

- Unit tests for:
  - validators (zod)
  - mappers/formatters
  - critical hooks
- Integration tests for:
  - at least one critical screen flow per module
- Contract checks:
  - generated client compiles and matches OpenAPI

### Gate must include evidence

At end of each phase provide:

- ✅ Commands run and results:
  - `pnpm lint`
  - `pnpm typecheck`
  - `pnpm test`
- ✅ Manual test checklist results (API + UI + navigation)
- ✅ Known issues list (must be empty for the gate, or explicitly accepted by me)

## 8.1) Design Gate Rules (Per Phase, Mandatory)

Every phase gate must include a **Design checklist** in addition to API/UI/testing checks.

### Minimum design checklist

- Layout is responsive on common phone widths (no clipped primary content/actions).
- Typography hierarchy is clear (title/subtitle/body/caption are visually distinct).
- Spacing and radii use shared tokens only (`src/theme/tokens.ts`).
- Touch targets are at least 44px and interactive controls are not crowded.
- Contrast/readability is acceptable and critical text is legible in normal lighting.
- Component states are covered for primary interactions:
  - default
  - loading
  - disabled
  - error
- No ad-hoc one-off visual patterns when shared components can handle the same behavior.
- Design evidence is captured in phase docs (short checklist + screenshots or explicit notes).

---

## 9) Waterfall Phase Gate Format (Required Output)

At the end of each phase, produce a **Gate Report**:

### Phase X — Gate Report

- Deliverables completed: (list)
- API checklist: ✅/❌
- UI checklist: ✅/❌
- Design checklist: ✅/❌
- Tests:
  - lint: ✅/❌
  - typecheck: ✅/❌
  - unit/integration tests: ✅/❌
- Docs checklist: ✅/❌
- Manual checks:
  - Auth/API calls: ✅/❌
  - UI states (loading/empty/error): ✅/❌
  - Design acceptance (layout/spacing/typography/accessibility): ✅/❌
  - Navigation: ✅/❌
- Risks / assumptions:
  - (must be empty or require my confirmation)
- Request:
  - “Approve Phase X to start Phase X+1” (do not proceed without approval)

---

## 10) Security & Compliance Rules

- Tokens stored only in SecureStore
- Never persist refresh tokens in plain AsyncStorage
- No secrets committed to repo
- Validate file uploads and sizes (if implemented)
- Respect RBAC:
  - Hide restricted actions
  - Block restricted routes with PermissionGate

---

## 11) Forbidden Actions

- Proceeding to a new phase when the current gate is not green
- Inventing endpoints not present in OpenAPI without asking me
- Adding large libraries without approval
- Duplicating components instead of reusing the shared kit
- Mixing Supabase in any way

---

## 12) Inputs You Must Request From Me When Needed

- API base URLs for dev/staging/prod (or confirmation to use local)
- Confirmed auth flow details (refresh token or not)
- Confirm module order and MVP scope
- Any missing endpoint decisions (whether to stub UI or request backend endpoint)

---

END OF RULES

## UI Reuse Non-Negotiable
- All screens must use shared page/layout/filter/pagination/skeleton components.
- Do not build one-off filter bars, pagination blocks, or page skeletons inside feature modules.
- If a new UI pattern is needed, add it in shared components first, then reuse it.

---
> Source: [Qualorahub/qualorahub-mobile](https://github.com/Qualorahub/qualorahub-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
