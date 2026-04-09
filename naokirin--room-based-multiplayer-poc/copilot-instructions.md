## room-based-multiplayer-poc

> Auto-generated from all feature plans. Last updated: 2026-02-12

# room-based-multiplayer-poc Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-12

## Active Technologies

- TypeScript 5.9, React 19, PixiJS 8.16, Zustand 5.0, Phoenix WS 1.8 (client)
- Ruby 3.3+, Rails 8.0+, rspec-openapi (api-server)
- Elixir 1.17+, Phoenix 1.7+ (game-server)
- MySQL 8.0+, Redis 7+ (datastores)
- Docker / Docker Compose (infra)

## Project Structure

```text
client/              # TypeScript/React/PixiJS web client
api-server/          # Ruby on Rails API + admin UI
game-server/         # Elixir/Phoenix game server (WebSocket)
infra/               # Docker Compose, MySQL/Redis config
specs/               # Feature specifications and plans
docs/                # Project documentation
```

## Commands

```bash
# Start all services
docker compose up -d

# Rails
cd api-server && bin/rails server -p 3001
cd api-server && bundle exec rspec
cd api-server && OPENAPI=1 bundle exec rspec  # Regenerate OpenAPI definitions

# Phoenix
cd game-server && mix phx.server
cd game-server && mix test

# Client
cd client && npm run dev
cd client && npm run typecheck
cd client && npm run biome:check
cd client && npm test
```

## Code Style

- Ruby: Standard Rails conventions, RSpec for tests
- Elixir: Standard Elixir/Phoenix conventions, ExUnit for tests
- TypeScript: Biome (lint + format), Vitest for tests (React 19, strict mode)

## Architecture Principles

- **Client**: Display only. No game logic or validation.
- **Rails (api-server)**: Operations — auth, matchmaking, persistence, admin.
- **Phoenix (game-server)**: Game runtime — rooms, game state, chat, reconnect.
- All game actions validated server-side (server-authoritative).

## Recent Changes

- 2026-02-21: Volume mounts added to docker-compose.yml for live code reload
  - api-server: `../api-server:/rails` + `bundle_gems` named volume for `/usr/local/bundle`; set `BUNDLE_DEPLOYMENT: ""` and `BUNDLE_WITHOUT: ""`
  - game-server: `../game-server:/app` + anonymous volumes for `/app/deps` and `/app/_build`
  - client: `../client:/app` + anonymous volume for `/app/node_modules` (replaces individual file mounts)
  - First run with empty `bundle_gems` requires: `docker compose -f infra/docker-compose.yml run --rm api-server bundle install`
- 2026-02-19: OpenAPI support added (003-openapi-api-split)
  - rspec-openapi gem generates OpenAPI 3.0.3 definitions from RSpec request specs
  - External API: doc/openapi/external.yaml (11 endpoints)
  - Internal API: doc/openapi/internal.yaml (5 endpoints)
- 2026-02-12: All 8 phases of MVP implementation complete (T001-T122)
  - Phase 1-2: Infrastructure setup, Docker Compose, DB models, auth, matchmaking
  - Phase 3 (US1): Full match-and-play flow — Rails matchmaking, Phoenix rooms, client UI
  - Phase 4 (US2): Reconnection with reconnect_token, localStorage persistence, 60s timeout
  - Phase 5 (US3): In-game chat via Phoenix channels, Zustand chat store
  - Phase 6 (US4): Matchmaking cancellation with timeout countdown
  - Phase 7 (US5): Admin panel (users, rooms, announcements), room commands via Redis PubSub
  - Phase 8: Audit logging, error handling middleware, PlugAttack rate limiting

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---

## From template: general

# General Project

Language-agnostic guidelines for any repository. Customize with project-specific layout and tooling.

## Principles

- **Immutability**: Prefer new objects over mutating existing ones; avoid hidden side effects.
- **Small units**: Many small files (200–400 lines typical, 800 max); small functions (<50 lines).
- **Error handling**: Handle errors at every level; never swallow errors silently.
- **Input validation**: Validate at system boundaries; never trust external data.
- **Security**: No hardcoded secrets; validate input; use parameterized queries; sanitize output.

## Standard Layout

Adjust to your project:

| Path | Purpose |
|------|---------|
| `src/` or app root | Application source |
| `tests/` or `*_test.*` | Unit/integration tests |
| Config files | Dependencies, lint, format, test (per language) |

## Verification

After code changes:

- Run tests (project’s test command).
- Run lint/format if configured.
- Use `/code-review` to check uncommitted changes for security and quality.

## Workflows

### Planning

For complex features or refactors: use `/plan` to get an implementation plan, then confirm before coding.

### Implementation

1. Plan (for non-trivial work).
2. Write tests first when TDD applies (RED → GREEN → REFACTOR).
3. Implement and verify (tests, lint, typecheck per project).
4. Run `/code-review` before commit.

### Commits

Use conventional commits: `type: short description` (feat, fix, refactor, docs, test, chore, perf, ci).

## Code Review Guidelines

When reviewing code (or using the code-reviewer agent), check: security (no secrets, validation, injection/XSS prevention), size (functions &lt;50 lines, files &lt;800 lines), error handling, tests, and naming. Categorize as **Critical** (must fix), **Warning** (should fix), or **Suggestion** (optional).

## Reference

- [everything-claude-code](https://github.com/affaan-m/everything-claude-code) — common rules and agents (source of this template’s patterns)

---

## From template: ruby_on_rails

# Ruby on Rails Project

## Principles

- **Convention over Configuration**: Follow Rails conventions for file placement, naming, and structure. Do not invent custom patterns when Rails provides a standard way.
- **DRY**: Extract shared logic into concerns, helpers, services, or base classes. Avoid copy-pasting code across controllers or models.
- **Fat Model, Skinny Controller**: Keep business logic in models (or service objects); controllers should only coordinate request/response flow.

## Standard Directory Layout

| Directory | Purpose |
|-----------|---------|
| `app/models/` | ActiveRecord models and concerns |
| `app/controllers/` | Request handling; keep actions thin |
| `app/views/` | ERB/HTML templates and partials |
| `app/helpers/` | View helper methods |
| `app/services/` | Service objects for complex business logic |
| `app/jobs/` | Active Job classes for async processing |
| `app/mailers/` | Action Mailer classes |
| `app/channels/` | Action Cable channels for WebSocket |
| `app/javascript/controllers/` | Stimulus controllers (Rails 7+) |
| `config/routes.rb` | Route definitions |
| `db/migrate/` | Database migrations |
| `spec/` or `test/` | Test suite |

## Style and Linting

- **Style**: Follow [Ruby Style Guide](https://rubystyle.guide), [Rails Style Guide](https://rails.rubystyle.guide), and [RSpec Style Guide](https://rspec.rubystyle.guide). When using Cursor, follow the project rules in `.cursor/rules/` (each .mdc). When using Claude Code only, use these guides and any project-specific rules in this file.
- **Linter**: Use RuboCop (`rubocop`, `rubocop-rails`, `rubocop-rspec`) to enforce style. Run before committing.
- **Tests**: Follow RSpec conventions when using RSpec; for Minitest use `test/` and the Rails Testing Guide.

## Verification

After making code changes, verify with:

- **Tests**: Run the relevant tests (`bundle exec rspec [path]` or `bin/rails test`). Prefer single files or scoped paths for speed when appropriate.
- **RuboCop**: Run `bundle exec rubocop [path]` and fix offenses. Use `/run-tests-rails` and `/run-rubocop-rails` (or equivalent commands) to drive verification.

## Reference

- [Rails Guides](https://guides.rubyonrails.org) for implementation details.
- [Rails API docs](https://api.rubyonrails.org) for class and method references.

---

## Workflows

### Rails Development

When adding or modifying resources (routes, controllers, models, views), migrations, validations, associations, or endpoints:

1. Follow project style rules (see Style and Linting above and `.cursor/rules/` if present).
2. **Routing**: Use `namespace` for admin/API. Run `bin/rails routes` after changes.
3. **Controllers**: Use strong parameters. Prefer `before_action` with lexical scope.
4. **Models**: When adding associations or validations, run the relevant specs or tests.
5. **Migrations**: Prefer DB-enforced defaults. Run `bin/rails db:migrate:status` after migration.
6. **Views**: Prefer partials and helpers; use route/path helpers. **Quick checks**: `bin/rails routes`, `bin/rails db:migrate:status`, run relevant specs.

### Rails Refactor

When simplifying code or applying style rules:

1. **Behavior first**: Refactors must not change observable behavior. Rely on existing tests.
2. **Incremental**: Prefer small, reviewable steps.
3. **Verification**: Run tests and `bundle exec rubocop [path]`; confirm no new failures.

### RSpec Testing

When writing or editing specs:

1. Follow project RSpec conventions (see `.cursor/rules/rspec-style.mdc` if present, or [RSpec Style Guide](https://rspec.rubystyle.guide)).
2. Create spec files under `spec/` mirroring `app/` structure. Run `bundle exec rspec [path]` to verify.

| Spec type | Directory | Tests |
|-----------|-----------|-------|
| Model | `spec/models/` | Validations, associations, scopes, methods |
| Request | `spec/requests/` | HTTP status, response body, redirects |
| System/Feature | `spec/system/` | Full browser-driven user flows |
| Helper | `spec/helpers/` | View helper methods |
| Mailer | `spec/mailers/` | Email delivery and content |
| Job | `spec/jobs/` | Active Job behavior |

---

## Code Review Guidelines

When reviewing Rails code, check: Ruby style (indentation, naming); Rails style (routing, skinny controllers, model macros, queries, migrations); RSpec style if specs in scope; ERB/Hotwire if views/JS in scope; security (strong params, no SQL interpolation, auth); maintainability (duplication, services). Categorize feedback as **Critical** (must fix), **Warning** (should fix), or **Suggestion** (optional).

---

## From template: phoenix

# Phoenix Project

## Principles

- **Idiomatic Elixir**: Prefer pipe `|>`, pattern matching, and immutable data. Use `with` for multi-step success/failure; avoid deep nesting. Keep functions small and single-purpose.
- **Phoenix conventions**: Use Contexts to group domain logic; keep Controllers/LiveView thin and delegate to contexts. Prefer `lib/my_app/` for business logic and `lib/my_app_web/` for web-facing code (controllers, live views, components).
- **Ecto**: Use schemas, changesets, and the repo for data; keep queries in context modules or dedicated query modules. Use migrations for schema changes; never edit migration files after they have been run in production.
- **LiveView**: Keep state in assigns; use `send(self(), ...)` for async work; avoid large inline markup—extract to function components or LiveComponents when it helps readability.
- **Error handling**: Return `{:ok, result}` / `{:error, reason}` from context functions; use `raise` only for truly exceptional cases. Handle errors at the controller/LiveView boundary (flash, redirect, or assign).

## Standard Layout

| Path | Purpose |
|------|---------|
| `lib/my_app/` | Contexts, domain logic, Ecto schemas (application name may vary) |
| `lib/my_app_web/` | Controllers, LiveView, components, routers, views |
| `lib/my_app_web.ex` | Web-facing function components and helpers |
| `lib/my_app_web/router.ex` | Routes (pipelines, scopes, live sessions) |
| `test/` | ExUnit tests; `support/` for ConnCase, DataCase, etc. |
| `priv/repo/migrations/` | Ecto migrations (do not modify after running in prod) |
| `mix.exs` | Project manifest (deps, project, aliases) |
| `config/` | Runtime config (config.exs, dev/test/prod) |
| `.formatter.exs` | Format options |

## Tooling and Style

- **Style**: Follow [Elixir style guide](https://hexdocs.pm/elixir/style-guide.html) and project rules in `.cursor/rules/` if present (phoenix-core, phoenix-style, ecto-style, liveview-style, exunit-style).
- **Format**: Use `mix format` (default or `.formatter.exs`). Run before committing.
- **Lint**: Use `mix credo` when available; fix or allow with justification.
- **Tests**: `mix test` for unit and integration tests; use `mix test path/to/file_test.exs` or `mix test --only tag` for scoped runs. Phoenix provides ConnCase, DataCase, and LiveView test helpers in `test/support/`.

## Verification

After code changes, verify with:

- **Tests**: `mix test` (optionally with path or `--only`). Use `/run-tests-phoenix` when available.
- **Credo**: `mix credo`; use `/run-credo-phoenix` when available.
- **Format**: `mix format` or rely on the format hook if using Cursor.
- **Compile**: `mix compile --warnings-as-errors` when you want to treat warnings as failures.

## Reference

- [Phoenix Guides](https://hexdocs.pm/phoenix/overview.html)
- [Ecto](https://hexdocs.pm/ecto/Ecto.html)
- [Phoenix LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html)
- [Elixir Style Guide](https://hexdocs.pm/elixir/style-guide.html)
- [ExUnit](https://hexdocs.pm/ex_unit/ExUnit.html)

---

## Workflows

### Phoenix Development

When adding or changing web features: follow context boundaries (logic in `lib/my_app/`, web in `lib/my_app_web/`); use `mix phx.gen.*` for scaffolds when appropriate; run `mix compile`, `mix test`, and `mix credo` after changes.

### Ecto / Data Layer

When adding schemas or migrations: create migration with `mix ecto.gen.migration`; define schema and changeset; add context functions; write DataCase tests. Do not edit existing migrations that have been applied.

### LiveView

When adding or changing LiveView: keep mount/update/handle_event thin; delegate to context; use assigns for UI state; add LiveView tests in `test/my_app_web/live/` when applicable.

### ExUnit Testing

When adding or fixing tests: place tests in `test/` with `*_test.exs`; use ConnCase for HTTP, DataCase for repo, LiveView test helpers for live; prefer `describe` and `test` with clear descriptions. Run `mix test` to verify.

---

## Code Review Guidelines

When reviewing Phoenix code, check: context boundaries and thin controllers/LiveViews; idiomatic Elixir (pipe, pattern match, `with`); Ecto usage (changesets, queries, no N+1); LiveView best practices (assigns, async); test coverage and clarity. Categorize feedback as **Critical** (must fix), **Warning** (should fix), or **Suggestion** (optional).

---

## From template: typescript

# TypeScript Project

## Principles

- **Strict TypeScript**: Prefer strict mode; avoid `any`; use explicit types for public APIs and boundaries. Let the type system catch errors at compile time.
- **ES Modules**: Use `import`/`export` syntax; prefer destructuring when importing named exports (e.g. `import { foo } from 'bar'`). Do not use `require()` unless integrating with CommonJS.
- **Single Responsibility**: Keep functions and modules focused; prefer small, testable units. Use composition over large classes.

## Standard Layout

| Path | Purpose |
|------|---------|
| `src/` | Application source (`.ts`, `.tsx`) |
| `src/**/*.ts` | TypeScript modules |
| `src/**/*.tsx` | React/JSX components (if applicable) |
| `test/` or `__tests__/` or `*.test.ts` | Unit/integration tests |
| `package.json` | Dependencies and scripts |
| `tsconfig.json` | TypeScript compiler options; recommend `compilerOptions.strict: true` |
| `biome.json` | Biome configuration (linter and formatter) |

### Introducing Biome

To add Biome to a project: install with `npm install -D @biomejs/biome`, then run `npx biome init` to create `biome.json`. Add scripts: `"lint": "biome lint ."`, `"lint:fix": "biome lint --write ."`, `"format": "biome format --write ."`. The format hook and `/run-lint-typescript` / `/run-format-typescript` use these when available.

## Tooling and Style

- **Style**: Follow project rules in `.cursor/rules/` if present (typescript-core, typescript-style, typescript-idioms). Prefer Biome over pasting full style guides into instructions.
- **Format**: Use Biome formatter (project config or default). Run before committing; use the format hook when available.
- **Lint**: Use Biome linter; fix or disable with justification when necessary.
- **Typecheck**: Run `npm run typecheck` or `tsc --noEmit` (or equivalent) after code changes.
- **Tests**: Use Vitest, Jest, or project test runner; keep tests next to code or in a dedicated test directory per project convention.

## Verification

After code changes, verify with:

- **Typecheck**: `npm run typecheck` or `npx tsc --noEmit`. Use `/run-typecheck-typescript` when available.
- **Tests**: `npm test` or `npm run test` (optionally with path or pattern). Use `/run-tests-typescript` when available.
- **Lint**: `npm run lint` or `npx biome lint .` (optionally `--write` to fix); use `/run-lint-typescript` when available.
- **Format**: `npm run format` or `npx biome format --write .`; use `/run-format-typescript` when available.

## Reference

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [Biome](https://biomejs.dev/)
- [Vitest](https://vitest.dev/) / [Jest](https://jestjs.io/)

---

## Workflows

### TypeScript Development

When writing or modifying TypeScript code (modules, types, APIs, components): follow project style and idiom rules (`.cursor/rules/` or above); prefer strict types and ES modules; avoid `any`; run typecheck, tests, and lint after changes.

### TypeScript Refactor

When simplifying code or applying style/idiom rules: preserve observable behaviour; rely on existing tests; prefer small steps; after refactor run typecheck, tests, and lint and confirm no new failures.

### Testing (Vitest / Jest)

When adding or fixing tests: place tests per project convention (`*.test.ts`, `__tests__/`, or `test/`); use `describe` and `it`/`test` with clear descriptions; prefer explicit assertions. Run the test script to verify.

---

## Code Review Guidelines

When reviewing TypeScript code, check: strict typing and no unnecessary `any`; ES module usage and import style; naming (camelCase for variables/functions, PascalCase for types/classes); error handling and async patterns; test coverage and clarity. Categorize feedback as **Critical** (must fix), **Warning** (should fix), or **Suggestion** (optional).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naokirin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/naokirin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
