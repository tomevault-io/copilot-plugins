## bataknese-wedding-budget-frontend-automation

> You are a Playwright + Cucumber test generator for this repository.

# FE Automation — Agent Playbook

You are a Playwright + Cucumber test generator for this repository.

This suite runs against a **production / production-like** frontend. Prefer safe, idempotent flows. Never invent secrets. Always explore the real UI before writing or changing automation.

## Credentials and secrets

- **Never** put real emails, passwords, tokens, or API keys in:
  - `AGENTS.md`
  - feature files
  - page objects
  - step definitions
  - commits
- Load accounts only from local files under `config/` (gitignored), referenced by **file name** in Gherkin.
- Expected local credential files (create from examples; do not commit real values):
  - `config/credentials_login_valid.json`
  - `config/credentials_login_invalid.json`
- Shape (see `config/credentials*.example.json`):

```json
{
  "validUser": {
    "email": "your-email@example.com",
    "password": "your-password"
  }
}
```

- In CI (Jenkins), when those files are absent, credentials may come from env:
  - `LOGIN_VALID_EMAIL` / `LOGIN_VALID_PASSWORD` (required for authenticated scenarios)
  - `LOGIN_INVALID_EMAIL` / `LOGIN_INVALID_PASSWORD` (optional; defaults to non-secret placeholders)
- Use env vars from `.env` (from `.env.example`) for `BASE_FE_URL` and related settings. Never hardcode environment-specific URLs into features when a relative path or existing step will do.

## Stack and layout

| Area | Path |
| --- | --- |
| Features (Gherkin) | `features/ui/**/*.feature` |
| Page objects (CSS locators) | `features/pages/` |
| Step definitions | `features/step_definitions/` |
| World, hooks, env, credentials | `features/support/` |
| Credential JSON | `config/` |
| Cucumber profiles | `cucumber.js` |

Do **not** generate standalone `@playwright/test` TypeScript specs unless the user explicitly asks. Default output is **Gherkin + page objects + Cucumber steps (JavaScript)**.

## Mandatory workflow (do not skip)

1. **Explore first** with Playwright MCP (or an equivalent live browser).  
   Do **not** write feature/page/step code from the scenario text alone.
2. **Login** only via the app UI using credentials loaded from the local config files above (or existing login steps that already do that).
3. Walk the scenario **step by step** on the real page. Focus only on elements the scenario needs.
4. After the flow is confirmed, implement in this order:
   - update or add **page object** locators/methods under `features/pages/`
   - **reuse existing steps** when possible (`README.md` step catalog + `features/step_definitions/`)
   - add missing steps under `features/step_definitions/`
   - add or update the **feature** under `features/ui/`
5. Run the relevant tests and **iterate until green**.
6. Prefer tag-scoped runs while developing (see [Commands](#commands)).

## Feature file rules

- Use Gherkin (`Feature` / `Scenario` / `Scenario Outline`).
- Tag every UI feature with `@ui`. Add a domain tag (e.g. `@login`, `@vendor_user`, `@project`).
- Use `@smoke` only for fast, low-risk checks (login, logout, view-only navigation).
- Use `@destructive` for scenarios that create, edit in place, delete, finalize, or otherwise mutate durable production data.
- Prefer **Scenario Outline + Examples** when the flow is the same and only filter/type/export values change.
- Do **not** put project-unique absolute paths, full production URLs, or secrets in feature files.
- Reference credential **file names** only, e.g. `credentials_login_valid.json`.

## Page object rules

- Put selectors in `features/pages/*_page.js` as CSS (or Playwright role/label helpers). **No XPath.**
- Expose an `elements` map and semantic methods (e.g. `open()`, `login()`, `expectDashboardVisible()`).
- Wait until the target is **visible** before click, fill, or assertion.
- Prefer stable selectors: `getByRole`, `getByLabel`, `getByText`, data attributes, then CSS.
- Do not dump every element on the page—only what the scenario uses.

## Step definition rules

- Always wait for visibility before interact or verify (via page methods or `expect(...).toBeVisible()`).
- Reuse steps from existing files before inventing new wording.
- Keep steps business-readable; put technical detail in page objects.
- Register new page objects on the world in `features/support/world.js` when needed.

## Production safety

This suite targets a live deployment (see `BASE_FE_URL` / Jenkins). Follow:

1. Prefer **view-only** checks when they satisfy the requirement.
2. For mutations: **create with a unique name** → assert → **clean up** in the same scenario when possible.
3. Use unique identifiers: timestamps or random suffixes (e.g. `AutoTest-20260719-...`).
4. Avoid deleting or finalizing data you did not create in that scenario unless the step is explicitly designed for disposable fixtures.
5. Do not spam login or heavy filters in tight loops during exploration.
6. Treat screenshots, videos, and traces as sensitive artifacts.

## Locator and assertion preferences

- CSS or role/label based locators; never XPath.
- Web-first assertions from `@playwright/test` (`toBeVisible`, `toHaveURL`, `toContainText`, etc.).
- Avoid brittle fixed `waitForTimeout` unless there is no better signal.
- `networkidle` may fail on long-polling apps—prefer element-level readiness.

## Commands

```powershell
# Install
npm install
npm run install:browsers
Copy-Item .env.example .env
# Create local config/credentials_login_valid.json and credentials_login_invalid.json from examples

# Runs
npm run test:smoke    # @smoke only — fast signal
npm run test:ui       # all @ui scenarios
npm test              # all features
npm run test:headed   # headed UI
npm run test:debug    # Playwright debug
```

Cucumber tag examples:

```powershell
npx cucumber-js --profile ui --tags "@smoke"
npx cucumber-js --profile ui --tags "@login"
npx cucumber-js --profile ui --tags "not @destructive"
```

## Definition of done

A change is done only when:

- [ ] Flow was verified on the live UI (MCP or headed run), not invented offline
- [ ] Page objects use CSS/role locators (no XPath)
- [ ] Steps wait for visibility before actions/asserts
- [ ] Features have correct tags (`@ui`, domain, `@smoke` / `@destructive` when applicable)
- [ ] No secrets or environment-specific absolute paths in committed files
- [ ] Relevant npm script / tag run passes locally
- [ ] Destructive scenarios clean up or use disposable data

## Out of scope unless the user asks

- Installing Playwright CLI skills (this repo uses Playwright MCP + Cucumber)
- Rewriting the suite to pure `@playwright/test` TypeScript
- Committing real credentials or `.env`

---
> Source: [yosilahi96/bataknese-wedding-budget-frontend-automation](https://github.com/yosilahi96/bataknese-wedding-budget-frontend-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
