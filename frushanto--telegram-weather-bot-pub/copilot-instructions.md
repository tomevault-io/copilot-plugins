## telegram-weather-bot-pub

> > **Project:** Telegram Weather Bot

# AGENTS.md — Project Agent Guide

> **Project:** Telegram Weather Bot
> **Repo root:** `telegram-weather-bot-pub/`
> **Primary entrypoint:** `app.py`
> **Architecture:** Clean Architecture (DDD, DI, event-driven modules)
> **Languages:** Python 3.12+, async-first, i18n: ru/en/de
> **License:** MIT

---

## Table of Contents

1. [Agent System Prompt](#agent-system-prompt)
2. [Mission & Scope](#mission--scope)
3. [Repository Map (Authoritative)](#repository-map-authoritative)
4. [Operational Constraints & Budgets](#operational-constraints--budgets)
5. [Security, Privacy, Compliance](#security-privacy-compliance)
6. [Environment & Configuration](#environment--configuration)
7. [Coding Standards](#coding-standards)
8. [Internationalization & UX](#internationalization--ux)
9. [Rate Limiting & Quotas](#rate-limiting--quotas)
10. [Jobs, Backups & Admin](#jobs-backups--admin)
11. [Error Handling & Observability](#error-handling--observability)
12. [CI/CD & Tooling](#cicd--tooling)
13. [Makefile Commands Reference](#makefile-commands-reference)
14. [Git Conventions](#git-conventions)
15. [Task Execution Protocol](#task-execution-protocol)
16. [Do / Don't](#do--dont)
17. [Change Placement Decision Tree](#change-placement-decision-tree)
18. [Machine-Readable Policy](#machine-readable-policy)
19. [Storage Schema Contracts & Migrations](#storage-schema-contracts--migrations)
20. [i18n Style Guide](#i18n-style-guide)
21. [Pre-PR Checklist](#pre-pr-checklist)
22. [Incident Playbooks](#incident-playbooks)
23. [Observability Contract](#observability-contract)
24. [Common Tasks Cookbook](#common-tasks-cookbook)
25. [Minimal E2E Smoke (Local)](#minimal-e2e-smoke-local)
26. [PR Template](#pr-template)
27. [Issue Template](#issue-template)
28. [Release & Versioning](#release--versioning)
29. [Glossary](#glossary)
30. [Escalation & Non-Goals](#escalation--non-goals)

---

## Agent System Prompt

* You are the **“WeatherBot Project Agent.”**
* **Objectives:** adhere to Clean Architecture boundaries; keep diffs minimal and reversible; maintain multilingual UX (ru/en/de); never leak secrets; keep tests green.
* **When unsure:** document reasoning briefly in the PR description; propose the smallest viable option.
* **PR output style:** concise summary listing touched layers and exact commands run.
* **Hard constraints:**
  * Do **not** modify storage schemas without a version bump, migration, and tests.
  * Do **not** commit `.env` files, tokens, or secrets; avoid logging secrets/PII.
  * Keep per-chat command localization in sync whenever language-facing strings change.
* **Acceptance criteria (default):**
  * Tests pass and coverage is not reduced on touched files.
  * Admin gating, rate limits, and quotas remain intact.
  * Clean Architecture layering respected; no inward dependency leaks.

---

## Mission & Scope

**Mission:** Deliver a reliable, privacy-respecting Telegram weather bot with delightful multilingual UX and predictable operational costs.

**In scope:**

* Telegram handlers, keyboards, onboarding flows, admin tools.
* Weather queries, geocoding, subscriptions, backups, observability.
* Tests, documentation, CI/CD refinements, refactors aligning with architecture.
* Localization improvements for Russian, English, and German.

**Out of scope (unless explicitly requested):**

* Switching persistence away from JSON without a migration plan.
* Massive architectural rewrites or multi-instance scaling.
* Introducing paid/closed APIs by default.

---

## Repository Map (Authoritative)

```
weatherbot/
├─ core/            # Value objects, entities, configuration contracts
├─ domain/          # Ports/interfaces, domain services
├─ application/     # Use cases, orchestrators, conversation logic
├─ infrastructure/  # External adapters (HTTP, storage, DI container)
├─ presentation/    # Formatters, i18n resources, keyboards
├─ handlers/        # Telegram command/callback handlers
├─ jobs/            # Scheduled jobs, backup routines
├─ modules/         # Modular bootstrapping & runtime composition
├─ observability/   # Logging, metrics, health endpoints
├─ utils/           # Shared utilities (time, text, validation)
└─ tests/           # Pytest suites across layers

app.py               # Application bootstrap & module assembly
CONFIG.md            # Environment variable reference
ARCHITECTURE.md      # Detailed architecture documentation
```

> Do not move files across layers without a clear architectural justification and accompanying documentation update.

---

## Operational Constraints & Budgets

* **Weather provider:** Open-Meteo (default, keyless) — shared budget of ~**1000 calls/day**.
* **Geocoding provider:** Nominatim — respect rate limits; cache results where reasonable.
* **Telegram API:** Soft limit **30 msgs/sec** globally; `python-telegram-bot` enforces per-chat throttling.
* **Internal throttle:** Target **≤5 requests/minute per user** for expensive operations.
* **Storage:** JSON files in `data/`; maintain backward compatibility and migrate gracefully.

---

## Security, Privacy, Compliance

* Never commit secrets or `.env` files. Provide `.env.*.example` only.
* Sensitive data (tokens, chat IDs beyond numeric) must not appear in logs or docs.
* Implement data minimization: store only what is required (language, home location, subscriptions, spam counters).
* `/data` export and `/delete_me` flows must remain functional and tested.
* Backups should respect `BACKUP_*` environment variables; redact secrets from backup archives.

---

## Environment & Configuration

* **Mandatory:** `BOT_TOKEN` (Telegram bot token).
* **Common optionals:** `ADMIN_IDS`, `ADMIN_LANGUAGE`, `STORAGE_PATH`, `TIMEZONE`, `BACKUP_*`, `WEATHER_API_*`, `WEATHER_SERVICE_PROVIDER`, `GEOCODE_SERVICE_PROVIDER`.
* Load configuration through DI/container abstractions; never hardcode secrets or rely on implicit globals.
* Local development uses `.env.dev`; production uses `.env.prod`. Provide sample entries in docs if changed.

---

## Coding Standards

* Python 3.12+, async-first, minimal blocking I/O.
* Prefer value objects, immutable conversation state, and pure functions in inner layers.
* Formatting: `make format` (Black + isort). Linting: `make lint` (flake8).
* Tests: `pytest`; keep or improve coverage on touched files.
* Layering rules:
  * `weatherbot/core` and `weatherbot/domain` cannot import from outer layers.
  * `weatherbot/application` orchestrates via interfaces only.
  * `weatherbot/infrastructure` supplies implementations & DI wiring.
  * `weatherbot/presentation` and `weatherbot/handlers` own user-facing strings and Telegram API usage.
* Follow type hints; prefer `Protocol` for ports; avoid `Any` unless justified.

---

## Internationalization & UX

* Supported languages: **ru / en / de**.
* User-facing strings live in `locales/` (GNU gettext) or presentation helpers; never hardcode in handlers.
* Maintain parity across languages when adding keys; ensure command menus refresh after language changes.
* Keep onboarding accessible: provide language choice, home location guidance, and fallback instructions.

---

## Rate Limiting & Quotas

* Respect per-user throttle and global daily quotas for weather/geocoding providers.
* Use caching and quota tracking utilities in `weatherbot.utils` and `weatherbot.infrastructure` where available.
* Notify admins proactively when approaching limits (re-use existing admin notification patterns).

---

## Jobs, Backups & Admin

* Jobs defined in `weatherbot/jobs` and registered via modules must be idempotent and timezone-aware.
* Backups read `BACKUP_*` configuration, store artifacts under `data/backups/`, and must not block the event loop.
* Admin commands gated by `ADMIN_IDS`; test negative paths to prevent privilege escalation.

---

## Error Handling & Observability

* Structured JSON logging (see `weatherbot/observability`). No secrets or PII beyond numeric IDs.
* Prefer explicit exception types and user-facing remediation tips.
* Emit metrics for API calls, quota usage, job execution, and error counts.
* Health endpoints and readiness checks should remain lightweight and authenticated if exposed.

---

## CI/CD & Tooling

* Keep GitHub Actions workflows green (mirrors `make ci-all`).
* Local pre-push checklist:
  * `make format`
  * `make lint`
  * `make test`
  * `make coverage`
* Docker: `make prod-up` / `make prod-down` for local container runs.
* Security scans: `make ci-security` (bandit) — run when touching security-sensitive code.

---

## Makefile Commands Reference

**CRITICAL:** Always use Makefile commands instead of direct CLI invocations for consistency and cross-platform compatibility.

### Development Workflow

* **`make help`** — Show all available commands with descriptions
* **`make venv`** — Create virtual environment (`.venv/`)
* **`make install`** — Install all dependencies (prod + dev)
* **`make run-dev`** — Run bot locally with `.env.dev` configuration
* **`make run-prod`** — Run bot locally with `.env.prod` configuration

### Testing & Quality Assurance

**Use these commands, NOT raw pytest/black/flake8:**

* **`make test`** — Quick test run (pytest -q)
* **`make coverage`** — Run tests with coverage report (generates `coverage.xml`)
* **`make coverage-html`** — Generate HTML coverage report (`htmlcov/index.html`)
* **`make test-all`** — Run coverage + upload to Codecov (requires `CODECOV_TOKEN`)

### Code Quality

**Always run before commits:**

* **`make format`** — Auto-format code (black + isort)
* **`make format-check`** — Check formatting without changes
* **`make lint`** — Run flake8 linter
* **`make pre-commit-install`** — Install pre-commit hooks

### CI/CD Validation

**Run before pushing release candidates:**

* **`make ci-test`** — Run all CI tests (black check + isort check + flake8 + pytest with coverage)
* **`make ci-security`** — Run security scan (bandit)
* **`make ci-all`** — Run complete CI pipeline (ci-test + ci-security)

### Docker Operations (Local)

* **`make prod-up`** — Start containers with `.env.prod` (detached mode, with build)
* **`make prod-logs`** — Show container logs (tail -f, last 100 lines)
* **`make prod-restart`** — Restart running containers
* **`make prod-down`** — Stop and remove containers
* **`make prod-clean`** — Stop, remove, and clean up containers

### Remote Deployment (SSH)

Requires `.env.deploy` with `SSH_HOST`, `PROJECT_DIR`, `BRANCH`:

* **`make deploy`** — Simple deploy (pull + rebuild + up)
* **`make deploy-clean`** — Clean deploy (stop → hard reset → rebuild)
* **`make deploy-logs`** — Show logs from remote containers
* **`make deploy-prune`** — Prune unused Docker images on remote
* **`make ssh-connect`** — Connect to remote server via SSH

### Local Server Deployment

When already on the server:

* **`make local-deploy`** — Deploy on current machine (hard reset to origin)
* **`make local-logs`** — Show local container logs

### Maintenance

* **`make clean`** — Remove `__pycache__` and `*.pyc` files
* **`make clean-venv`** — Remove virtual environment
* **`make freeze`** — Create `requirements.lock` from venv
* **`make backup-now`** — Manually trigger storage backup

### Common Use Cases

**1. Initial setup:**
```bash
make venv
make install
# Create .env.dev with BOT_TOKEN
make run-dev
```

**2. Before committing changes:**
```bash
make format    # Auto-fix formatting
make lint      # Check code quality
make test      # Verify tests pass
```

**3. Pre-release validation (Release Checklist Phase 2):**
```bash
make ci-all    # Must pass before pushing RC branch
```

**4. Code review / coverage check:**
```bash
make coverage-html    # Generate detailed HTML report
# Open htmlcov/index.html in browser
```

**5. Production deployment:**
```bash
# Remote (via SSH):
make deploy-clean

# Local (on server):
make local-deploy
```

**6. Debugging production:**
```bash
make deploy-logs    # Remote logs
make local-logs     # Local logs
```

### Anti-Patterns to Avoid

❌ **Don't:**
- Run `pytest` directly → Use `make test` or `make coverage`
- Run `black .` directly → Use `make format`
- Run `flake8` directly → Use `make lint`
- Run `docker compose up` directly → Use `make prod-up` or `make local-deploy`
- Install dependencies with `pip install -r requirements.txt` → Use `make install`

✅ **Do:**
- Always use Makefile commands for consistency
- Check `make help` when unsure about available commands
- Run `make ci-all` before pushing release branches
- Use `make format` before `make lint` to auto-fix issues

---

## Git Conventions

* Branch naming: `feat/*`, `fix/*`, `chore/*`, `docs/*`, `test/*`, `refactor/*`.
* Commit messages: Conventional Commits (`feat: …`, `fix: …`, etc.).
* Keep PRs focused; describe layer impact, UX changes, and test results.
* Avoid large unrelated refactors; split into separate PRs when needed.

---

## Task Execution Protocol

1. **Clarify** requirements only when absolutely necessary; otherwise infer from docs/tests.
2. **Locate** the relevant layer(s) using the repository map and architecture guidelines.
3. **Design** briefly—note significant choices or trade-offs in the PR body.
4. **Implement** minimal, reversible changes; respect DI patterns and event flows.
5. **Test**: add/adjust unit, handler, and integration tests as appropriate.
6. **Run** `make format && make lint && make test && make coverage` locally before committing.
7. **Document** behavior changes in README/CONFIG/ARCHITECTURE as needed.
8. **Open PR** using the provided template; include coverage deltas and manual QA steps.

Default acceptance: tests pass, coverage not reduced, i18n updated, quotas respected, no schema breaks.

---

## Do / Don’t

**Do**

* Extend functionality via defined interfaces and dependency injection.
* Centralize localization updates and refresh command menus on change.
* Add deterministic tests covering new behavior.
* Cache expensive API calls and respect provider policies.

**Don’t**

* Commit secrets or environment files.
* Introduce heavy dependencies without discussion.
* Bypass Clean Architecture boundaries or couple handlers to infrastructure details.
* Block the event loop with synchronous I/O in async code paths.

---

## Change Placement Decision Tree

* **Pure domain rules / value checks?** → `weatherbot/core`
* **Ports, policies, domain contracts?** → `weatherbot/domain`
* **Use-case orchestration (no IO specifics)?** → `weatherbot/application`
* **External IO, storage, DI wiring?** → `weatherbot/infrastructure`
* **Formatting, keyboards, text?** → `weatherbot/presentation`
* **Telegram update routing, command handlers?** → `weatherbot/handlers`
* **Scheduled tasks, backups?** → `weatherbot/jobs`
* **Cross-cutting logging/metrics?** → `weatherbot/observability`

---

## Machine-Readable Policy

Create or update `.agent-policy.yml` at repo root:

```yaml
language: "python3.12"
test_commands:
  - "make format"
  - "make lint"
  - "make test"
  - "make coverage"
risk_rules:
  forbid:
    - "commit_env_files"
    - "hardcode_tokens"
    - "blocking_io_in_async"
    - "schema_change_without_migration"
  require:
    - "i18n_update_on_command_change"
    - "admin_gating_on_admin_commands"
    - "quota_checks_on_weather_calls"
coverage:
  min_delta: ">=0"
rate_limits:
  per_user_rpm: 5
  telegram_global_mps: 30
  weather_daily_calls: 1000
```

Keep this file synchronized with policy changes; update agents if constraints shift.

---

## Storage Schema Contracts & Migrations

* Primary storage: `data/storage.json`. Changes must remain backward-compatible.
* Maintain a JSON schema similar to the structure documented in `ARCHITECTURE.md`.
* Each breaking change requires:
  1. Version bump in the stored data.
  2. Migration script `data/migrations/mig_<N>_<desc>.py` with `upgrade()`/`downgrade()`.
  3. Tests in `tests/test_migrations.py` covering happy path, corrupted data, and rollback.
* Never write partial data on failure; use atomic file writes.

---

## i18n Style Guide

* Key format: `scope.feature.message` (e.g., `cmd.start.welcome`).
* Placeholder syntax: `{city}`, `{temp}`, `{time}` — handled via `.format` or gettext substitution.
* Keep sentences concise, sentence case, no trailing spaces.
* Update RU/EN/DE simultaneously; add tests ensuring command menus include new translations.

---

## Pre-PR Checklist

* [ ] `make format`
* [ ] `make lint`
* [ ] Tests added/updated (`make test`)
* [ ] Coverage maintained/improved (`make coverage`)
* [ ] i18n strings updated in ru/en/de; command menu refresh verified
* [ ] Quotas and throttling respected (tests or reasoning documented)
* [ ] No secrets or `.env` files committed
* [ ] Storage schema changes include migration + tests (if applicable)
* [ ] Docs updated (README/CONFIG/ARCHITECTURE) when behavior changes

---

## Release Checklist

Follow this step-by-step procedure when preparing a new release:

### Phase 1: Branch and Version Bump

1. **Create Release Candidate Branch**
   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b rc/X.Y.Z
   ```

2. **Update Version Metadata in `weatherbot/__version__.py`**
   - [ ] Update `__version__ = "X.Y.Z"`
   - [ ] Update `__version_info__ = (X, Y, Z)`
   - [ ] Update `__release_date__ = "DD.MM.YYYY"` (current date)
   - [ ] Add new release notes to `RELEASE_NOTES` at the top with `🎉 New in X.Y.Z` header
   - [ ] Include: bug fixes, new features, breaking changes, improvements
   - [ ] Keep previous releases under "Previous Releases" section

3. **Sync Version in `pyproject.toml`**
   - [ ] Update `[project] version = "X.Y.Z"`

4. **Update Localization Files (`locales/{en,ru,de}.json`)**
   - [ ] Change `admin_version_whats_new` to reference new version:
     - EN: `"What's new in version X.Y.Z"`
     - RU: `"Что нового в версии X.Y.Z"`
     - DE: `"Neuerungen in Version X.Y.Z"`

5. **Update README.md**
   - [ ] Change version badge: `[![Version](https://img.shields.io/badge/version-X.Y.Z-blue.svg)]`
   - [ ] Update "What's New — Version X.Y.Z" section with release highlights
   - [ ] Keep highlights concise (3-5 bullet points with emoji)
   - [ ] Focus on user-visible improvements and bug fixes

### Phase 2: Validation

6. **Run Full CI Pipeline**
   ```bash
   make ci-all
   ```
   - [ ] All tests pass (282+ tests expected)
   - [ ] Coverage maintained at ≥73%
   - [ ] No linting errors (black, isort, flake8)
   - [ ] Security scan clean (bandit: no issues)

7. **Verify Release Metadata Consistency**
   - [ ] Run `pytest tests/test_release_metadata.py -v`
   - [ ] Ensure all version checks pass:
     - Version matches across `__version__.py`, `pyproject.toml`, `README.md`
     - Localization keys updated in all languages
     - Release date format valid (DD.MM.YYYY)
     - Release date not in future
     - `RELEASE_NOTES` contains entry for new version

### Phase 3: Commit and Push

8. **Commit Release Candidate**
   ```bash
   git add -A
   git commit -m "chore(release): prepare X.Y.Z RC

   Version bump to X.Y.Z with release notes updates.

   Highlights:
   - [Feature 1]
   - [Feature 2]
   - [Bug fix]

   CI: 282+ tests passing, coverage ≥73%, bandit clean."
   ```

9. **Push Release Branch**
   ```bash
   git push origin rc/X.Y.Z
   ```

### Phase 4: Merge to Dev and Verify CI

10. **Merge RC to Dev**
    ```bash
    git checkout dev
    git pull origin dev
    git merge --no-ff rc/X.Y.Z -m "Merge release candidate X.Y.Z to dev"
    git push origin dev
    ```

11. **Verify GitHub Actions CI Pipeline**
    - [ ] Navigate to GitHub Actions tab
    - [ ] Wait for CI pipeline to complete on `dev` branch
    - [ ] Ensure all checks pass (tests, linting, security)
    - [ ] If CI fails, fix issues in `dev` and cherry-pick to `rc/X.Y.Z`

### Phase 5: Merge to Main and Tag

12. **Merge Dev to Main**
    ```bash
    git checkout main
    git pull origin main
    git merge --no-ff dev -m "Release X.Y.Z"
    git push origin main
    ```

13. **Create Git Tag**
    ```bash
    git tag -a vX.Y.Z -m "Release X.Y.Z

    [Copy release notes highlights here]"
    git push origin vX.Y.Z
    ```

### Phase 6: Post-Release

14. **GitHub Release (Optional)**
    - Create GitHub release from tag `vX.Y.Z`
    - Copy release notes from `RELEASE_NOTES`
    - Attach any binaries or deployment artifacts if needed

15. **Deployment**
    - Deploy to production environment
    - Verify bot responds with new version in `/admin_version` command
    - Monitor logs for errors during initial rollout

16. **Cleanup**
    - Delete release candidate branch (optional):
      ```bash
      git branch -d rc/X.Y.Z
      git push origin --delete rc/X.Y.Z
      ```

### Release Type Guidelines

**Patch Release (X.Y.Z where Z increments):**
- Bug fixes only
- No new features
- Backward compatible

**Minor Release (X.Y.Z where Y increments):**
- New features
- Improvements
- Backward compatible
- May include bug fixes

**Major Release (X.Y.Z where X increments):**
- Breaking changes
- Major architectural changes
- Requires migration guide
- Update `RELEASE_NOTES` with ⚠️ BREAKING CHANGES section

### Common Release Pitfalls

❌ **Don't:**
- Skip CI validation
- Forget to update localization in all languages
- Commit version bumps directly to `main` or `dev`
- Merge RC directly to `main` without testing in `dev` first
- Tag before merging to `main`
- Use generic commit messages

✅ **Do:**
- Always create an RC branch
- Merge RC → dev → verify CI → main
- Run full test suite before pushing
- Keep release notes user-friendly
- Update all version references atomically
- Test metadata consistency with `test_release_metadata.py`
- Wait for GitHub Actions to pass on `dev` before merging to `main`

---

## Incident Playbooks

### Weather Provider Outage

* Return friendly fallback message; log `level=WARNING` with context.
* Retry with capped exponential backoff (≤2 retries).
* Notify admins once; avoid spam.

### Geocoding Throttling

* Apply exponential backoff; encourage user to retry later.
* Cache last successful result per query to reduce load.

### Storage Corruption

* Quarantine corrupted file (`storage.json.broken.<timestamp>`).
* Restore latest backup; notify admins; run in degraded mode if necessary until restored.

---

## Observability Contract

* **Logs:** JSON fields (`ts`, `level`, `event`, `user_id?`, `chat_id?`, `lang`, `cmd`, `duration_ms`, `ok`, `err?`). No secrets.
* **Metrics:** weather calls, geocode calls, quota remaining, subscriptions sent, rate-limit hits, job durations.
* **Tracing/health:** keep lightweight; expose only non-sensitive data.

---

## Common Tasks Cookbook

**Add a new user command (e.g., `/hourly`):**

1. Define request/response models in `weatherbot/domain` (ports + DTOs).
2. Implement use case in `weatherbot/application` and wire through modules.
3. Add presenter/formatter in `weatherbot/presentation`.
4. Implement handler in `weatherbot/handlers` and register via `modules/command.py`.
5. Update localization strings and command menus (ru/en/de).
6. Add tests: use case, presenter snapshot, handler integration.

**Add a background job (e.g., subscription digest):**

1. Implement job in `weatherbot/jobs` using async-friendly patterns.
2. Register in the Jobs module; ensure schedule respects `TIMEZONE`.
3. Cover with tests (scheduling, quota respect, failure handling).

**Swap or extend weather provider:**

1. Implement provider adapter in `weatherbot/infrastructure/weather/` behind existing interface.
2. Update DI to allow selection via `WEATHER_SERVICE_PROVIDER` env.
3. Add tests using provider overrides (`weatherbot.infrastructure.container` helpers) — no monkeypatching globals.

---

## Minimal E2E Smoke (Local)

1. Create `.env.dev` with `BOT_TOKEN=<test token>` and run `make run-dev`.
2. `/start` → language menu renders.
3. `/sethome <city>` → stored; `/home` returns localized weather summary.
4. `/subscribe` → schedules daily job; verify logs for job queue entry.
5. `/language` → switch to `de`; command menu updates accordingly.

---

## PR Template

```markdown
### Summary
(What changed and why—succinct.)

### Layer Impact
- Core:    [ ] none / [ ] touched (details)
- Domain:  [ ] none / [ ] interfaces changed
- App:     [ ] none / [ ] new use case / [ ] modified
- Infra:   [ ] none / [ ] new client / [ ] DI changes
- Present: [ ] none / [ ] i18n/formatting changes
- Handlers:[ ] none / [ ] commands affected
- Jobs:    [ ] none / [ ] scheduler changes

### i18n
- [ ] ru updated
- [ ] en updated
- [ ] de updated
- Per-chat menu refreshed on change: [ ] yes

### Quotas & Rate Limiting
- [ ] respected
- [ ] near-limit messaging for users/admins added (if relevant)

### Testing
- [ ] unit tests
- [ ] handler tests
Coverage: before → after: X% → Y%

### Docs
- [ ] README / ARCHITECTURE / CONFIG updated (if needed)

### Risks & Rollback
(Top risks + quick rollback strategy.)
```

---

## Issue Template

```markdown
### Task
(Concise change request.)

### Acceptance Criteria
- [ ] Expected behavior described
- [ ] i18n updated (ru/en/de)
- [ ] Command menus refreshed (if applicable)
- [ ] Quotas & throttling respected
- [ ] Tests pass; coverage not reduced

### Notes
(Layers involved, related tickets, references.)
```

---

## Release & Versioning

* Update `weatherbot/__version__.py` and accompanying release notes when introducing user-visible changes.
* Keep README “What’s New” aligned with latest tag.
* Follow semantic versioning unless product requirements dictate otherwise.

---

## Glossary

* **Home Location:** `{lat, lon, label}` stored per user for quick forecasts and subscriptions.
* **Subscription:** Scheduled push delivering weather updates at configured local time.
* **Quota:** Shared 24h budget for weather provider calls.
* **Command Menu:** Localized Telegram command list configured via Bot API (`setMyCommands`).
* **Modules:** Bootstrap components in `weatherbot/modules` that register handlers, jobs, and services.

---

## Escalation & Non-Goals

* If a change risks breaking architectural boundaries or quotas, pause and surface a design note before implementation.
* Avoid stateful singletons, cross-layer imports, or schema changes without migration.
* Non-goals: multi-instance scaling, database migrations, paid API integration—unless explicitly requested by stakeholders.

---

**End of AGENTS.md**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frushanto)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/frushanto)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
