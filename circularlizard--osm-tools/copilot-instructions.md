## osm-tools

> SEEE Expedition Dashboard testing and QA rules


# SEEE Expedition Dashboard Testing Rules

## 1. Testing Stack
- **Unit & component tests:** Jest + React Testing Library.
- **Integration tests:** Jest against API routes and the safety layer (proxy, bottleneck, Redis, Zod parsing).
- **End-to-end tests:** Playwright (desktop + mobile profiles, HTTPS dev server).
- **BDD tests:** Playwright with `playwright-bdd` for Gherkin-based scenarios.
- **Mutation testing:** Stryker for detecting test gaps.
- **Mocking network:** MSW (Mock Service Worker) for simulating OSM API behavior and safety-layer scenarios.
  - Prefer MSW handlers over global `fetch` mocks. Do **not** globally mock `fetch` when MSW can intercept the request.
- **Requirement traceability:** All tests (unit, integration, E2E) should reference the relevant `REQ-<domain>-<nn>` identifiers from `docs/SPECIFICATION.md`. Feature files use `@REQ-...` tags; unit/integration tests include the ID in the `describe`/`it` names.

## 2. General Testing Principles
- **Safety-critical focus:**
  - Prioritize tests around the safety layer, rate limiting, circuit breaker, and validation.
  - Ensure read-only and proxy constraints are never violated in tests.
- **Deterministic tests:**
  - Use sanitized mock JSON from `src/mocks/data`.
  - Avoid flakiness by controlling timeouts, retries, and randomization.

## 2.1 Hook Mocking Guidelines
- Prefer mocking key hooks rather than standing up full provider trees unless the provider itself is under test:
  - Mock `useSession` from NextAuth to simulate authenticated/unauthenticated states.
  - Mock TanStack Query hooks (e.g. `useQuery`, feature-specific wrappers) when verifying component behavior, rather than wiring full QueryClient providers for every test.

## 3. Zod Validation Strategy
- **Tier 1 (Strict) validation:**
  - For core identifiers and structural fields (IDs, names, key relationships): schemas must `parse` or throw.
  - Tests should assert that invalid Tier 1 data surfaces clear, fail-fast errors (and does **not** render misleading UI).
- **Tier 2 (Permissive) validation:**
  - For optional or flexible logistics/flexi data (e.g. tent groups, walking groups, additional notes): schemas may treat fields as optional/nullable.
  - Tests should confirm that malformed or missing Tier 2 data results in `null`/empty columns, not runtime errors.
- **Helpers:**
  - Use and test `parseStrict` and `parsePermissive` from `src/lib/schemas.ts`.

## 4. Safety Layer & Proxy Testing
- **Unit/integration coverage for:**
  - `src/app/api/proxy/[...path]/route.ts` behavior:
    - Rate limiting and reservoir updates.
    - Soft lock and hard lock behavior (Redis keys and TTLs).
    - Read-through caching (cache hits/misses, corrupted cache recovery).
    - Error mapping (error codes, HTTP status, `retryAfter`).
  - `src/lib/bottleneck.ts`:
    - Rate limit header parsing.
    - Handling of `X-RateLimit-Remaining` and dynamic pausing.
- **Scenarios to simulate with MSW:**
  - 429 responses and exhausted quota → soft lock.
  - `X-Blocked` (or similar) headers → hard lock.
  - HTML or malformed payloads in cache → treated as cache miss.

## 5. Auth & Access Control Testing
- **NextAuth:**
  - Unit/integration tests for token refresh logic and JWT callbacks in `src/lib/auth.ts`.
  - Verify role/scopes are correctly attached for `osm-admin` vs `osm-standard`.
- **Access control selectors (Zustand):**
  - Unit tests for `getFilteredMembers`, `getFilteredEvents`, `getFilteredLogistics` to ensure:
    - Admin users see all permitted data.
    - Standard viewers are restricted by configured strategy (patrol-based or event-based).
- **Route protection:**
  - Tests for middleware and any client guards protecting `/dashboard` and `/dashboard/admin` routes.

## 6. UI & UX Testing
- **Component tests:**
  - Focus on data views: events list, event detail, per-person attendance, readiness views.
  - Check loading, error, and empty states are rendered as specified.
- **Responsive behavior:**
  - For table-heavy views, ensure:
    - Desktop: tables visible with `text-sm`, consistent padding, and hover states.
    - Mobile: card-based layouts replace tables (hide table on small screens, show cards).
- **Accessibility basics:**
  - Smoke tests for important ARIA attributes and keyboard navigation on key dialogs (e.g. column mapping dialog, section picker).

## 7. End-to-End (Playwright) Rules
- **Core flows to cover:**
  - Login with role selection (Standard vs Admin) → redirect to dashboard.
  - Section picker behavior for multi-section vs single-section users.
  - Events list loading from mock or real API (depending on environment flag).
  - Event detail navigation (from list to detail, participants visible).
  - Admin routes: Admin allowed, standard viewer blocked.
- **Environment assumptions:**
  - Redis running via `docker-compose` (see README instructions).
  - MSW enabled when testing against mocks; disabled when testing real OSM integration.
- **Reporting:**
  - Use HTML reports and traces for flakiness investigations.

## 8. PII & Mock Data Rules for Tests
- **Never** use real personal data in any test.
- Always source test payloads from:
  - `src/mocks/data/*.json` (sanitized), or
  - Inline generic placeholders ("Scout A", "Scout B", etc.).
- When adding new mocks:
  - Run or follow `scripts/sanitize_data.py` patterns.
  - Confirm that no emails, phone numbers, or addresses are present.

## 9. Pre-PR Validation
- Before opening a PR (or merging significant changes), run:
  - `npx tsc --noEmit` (or `npm run tscheck`).
  - `npm run lint`.
  - `npm run test` (unit/integration).
  - Optionally: `npm run test:e2e` or a targeted subset of Playwright tests.

## 10. When adding or changing features
- **Add tests alongside features:**
  - For new data views, add at least one component test and one E2E scenario.
  - For new schema or API changes, add/update Zod and proxy tests.
- **Refactor with safety:**
  - When refactoring the safety layer or auth:
    - Update or add tests first.
    - Run the full safety validation script (`npm run validate:safety`) where appropriate.

## 11. Testing Workflows

### 11.1 Windsurf Workflows

The following Windsurf workflows are available for local testing:

- **`/test-stack`** - Run the full test stack locally
  - Executes: lint → TypeScript check → unit tests → BDD E2E tests (instrumented) → coverage merge
  - Opens merged coverage report in browser
  - Location: `.windsurf/workflows/test-stack.md`

- **`/mutation-scan`** - Run mutation testing
  - Executes: unit tests → Stryker mutation testing
  - Opens mutation report in browser
  - Identifies surviving mutants and test gaps
  - Location: `.windsurf/workflows/mutation-scan.md`

- **`/bdd-fix`** - Debug failing BDD scenarios
  - Run specific feature or scenario by tag
  - View Playwright report with traces
  - Optional: Use `playwright codegen` for selector discovery
  - Location: `.windsurf/workflows/bdd-fix.md`

- **`/test-fix`** - Run tests and fix common issues
  - Quick test execution and common fix patterns
  - Location: `.windsurf/workflows/test-fix.md`

- **`/file-completed-plan`** - Archive completed plans
  - Move plan files to `docs/completed-plans/`
  - Update `COMPLETED_PHASES.md`
  - Update `SPECIFICATION.md`, `ARCHITECTURE.md`, `README.md`
  - Location: `.windsurf/workflows/file-completed-plan.md`

### 11.2 GitHub Actions Workflows

The following CI/CD workflows are configured:

- **`CI – Tests`** (`.github/workflows/ci-test.yml`)
  - Trigger: Pull requests, manual dispatch
  - Steps: lint → TypeScript check → unit tests → BDD E2E tests → coverage merge
  - Artifacts: Coverage reports (unit, E2E, merged), Playwright reports
  - Required for PR approval

- **`CI – Mutation Testing`** (`.github/workflows/ci-mutation.yml`)
  - Trigger: Nightly cron (2 AM), manual dispatch
  - Steps: unit tests → Stryker mutation testing
  - Artifacts: Mutation report with score
  - Optional (reports blockers)

- **`CI – Deploy`** (`.github/workflows/ci-deploy.yml`)
  - Trigger: Release published, manual dispatch
  - Steps: lint → TypeScript check → build
  - Artifacts: Build output
  - Depends on CI – Tests success

### 11.3 Test Commands

```bash
# Unit tests
npm run test:unit

# BDD E2E tests
npm run test:bdd

# BDD E2E tests with instrumentation
cross-env INSTRUMENT_CODE=1 npm run test:bdd

# Merge coverage
npm run test:merge

# Mutation testing
npm run test:mutation

# Full test stack (local)
npm run lint && npx tsc --noEmit && npm run test:unit && cross-env INSTRUMENT_CODE=1 npm run test:bdd && npm run test:merge
```

### 11.4 Coverage Targets

- **Numerical coverage**: ≥80% line coverage (unit + E2E merged)
- **Mutation score**: ≥80% killed mutants
- **BDD coverage**: All `REQ-*` requirements have at least one scenario
- **Safety layer**: 100% coverage for proxy, bottleneck, rate limiting

### 11.5 Workflow Best Practices

- Use `/test-stack` before committing to verify all tests pass locally
- Use `/mutation-scan` weekly or before major releases to find test gaps
- Use `/bdd-fix` when BDD scenarios fail to quickly debug and fix
- Use `/file-completed-plan` after completing major features to keep documentation current
- Review CI – Mutation Testing reports to prioritize test improvements
- Ensure all new features include BDD scenarios with `@REQ-*` tags

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/circularlizard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
