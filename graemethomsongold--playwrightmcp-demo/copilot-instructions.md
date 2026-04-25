## playwrightmcp-demo

> Playwright test automation framework using TypeScript and the Page Object Model (POM) pattern.

# Test Automation Framework — Claude Context

## Project Overview
Playwright test automation framework using TypeScript and the Page Object Model (POM) pattern.
This framework operates in an FCA-regulated banking environment. Security and code quality
standards are non-negotiable.

---

## Architecture

### Directory Structure
project-root/
├── .auth/                  # Playwright session state (gitignored)
├── .claude/                # Claude slash commands
│   └── commands/
├── pages/                  # Page Object classes
│   └── BasePage.ts         # Abstract base class — all Page Objects extend this
├── tests/                  # Test specs
│   └── auth.setup.ts       # Authentication setup project
├── utils/                  # Helpers (secrets, common functions)
├── CLAUDE.md
├── playwright.config.ts
└── .env                    # Local secrets only (gitignored)

---

## Page Object Model Rules

- All Page Object classes must extend `BasePage` (`pages/BasePage.ts`)
- `BasePage` provides: cookie consent handling and `goto(path)` navigation — do not duplicate these
- All locators must be defined as `private readonly` properties on the Page Object class
- Page Object constructors accept a Playwright `Page` object and must call `super(page)`
- All page interactions are encapsulated as methods on the Page Object class
- Test files must only call Page Object methods — no direct `page.click()`, `page.fill()` etc. in tests
- If an interaction is needed in a test, create a method for it first

---

## Naming Conventions

| Artefact          | Convention          | Example          |
|-------------------|---------------------|------------------|
| Page Object file  | `<Name>Page.ts`     | `LoginPage.ts`   |
| Page Object class | `<Name>Page`        | `LoginPage`      |
| Test file         | `<feature>.spec.ts` | `login.spec.ts`  |
| Auth setup        | `auth.setup.ts`     | —                |
| Utility files     | `camelCase.ts`      | `secrets.ts`     |

---

## Language & Framework

- **Language:** TypeScript (strict mode)
- **Test runner:** `@playwright/test`
- **Secrets:** Environment variables or Azure Key Vault via `utils/secrets.ts`
- **Session management:** Playwright `storageState` cached to `.auth/user.json`

---

## Authentication Pattern

- Credentials are **never** hard-coded anywhere in the codebase
- Credentials are **never** included in prompts or context files
- All credentials come from environment variables (`TEST_USERNAME`, `TEST_PASSWORD`)
  or Azure Key Vault at runtime
- Login is performed **once** in `auth.setup.ts` and saved to `.auth/user.json`
- All authenticated test projects depend on the setup project and load `storageState`
- Only dedicated automation accounts are used — never real user or shared credentials

---

## Security Rules

- `.env` and `.auth/` are always in `.gitignore`
- No credentials, tokens, or secrets appear in any committed file
- No production environment testing
- Test accounts have minimum necessary permissions
- If in doubt, ask before generating anything credential-related

---

## Test Execution

- Tests must **never** run in parallel — always run serially, one at a time
- This is critical: the target is a live site that will block or rate-limit concurrent requests
- Ensure `playwright.config.ts` has `workers: 1` and `fullyParallel: false`
- Do not suggest or generate config changes that would enable parallel execution

### Browser strategy

Running the full suite across all three browsers (141+ tests × 3 = 400+ navigations) will
trigger Akamai CDN rate limiting mid-run and cause spurious failures unrelated to the tests.

| Context | Command | Reason |
|---------|---------|--------|
| Full suite run (local) | `npx playwright test --project=chromium` | Avoids rate limiting — Chromium is the primary validation browser |
| New Page Object / new tests | `npx playwright test --project=chromium --project=firefox --project=webkit` | Cross-browser locator validation is essential when first writing selectors |
| CI | All browsers (configured in pipeline) | IP rotates per run, so rate limiting is not a concern |

- When running `/run` with no arguments locally, always use `--project=chromium` by default
- When running `/run` after writing or modifying a Page Object or test file, run all three browsers to validate cross-browser compatibility before considering the work done

---

## CI/CD

- Secrets are injected as environment variables — never stored in the repo
- The `.auth/` session file is generated fresh per pipeline run
- Tests run against non-production environments only

---

## Code Style

- Use `async/await` throughout — no promise chains
- Locators use Playwright best-practice selectors (prefer `getByRole`, `getByLabel`,
  `getByTestId` over CSS or XPath where possible)
- All Page Object methods should be `async` and return `Promise<void>` or a typed value
- Use TypeScript strict mode — no `any` types without justification

---

## Playwright MCP Server

When building or exploring tests, always use the Playwright MCP server to interact with the
browser directly. This allows real-time inspection of the live page to discover accurate
locators, validate behaviour, and confirm page structure before writing code.

**Workflow when creating a new Page Object or test:**

1. Use `mcp__playwright__browser_navigate` to open the target URL
2. Use `mcp__playwright__browser_snapshot` to inspect the accessibility tree and identify locators
3. Use `mcp__playwright__browser_take_screenshot` to visually confirm the page state if needed
4. Interact with elements via the MCP tools to verify behaviour before writing Page Object methods
5. Only write the Page Object and test once the live page structure has been confirmed

Do not guess or assume locators — always verify against the live page via the MCP server.

---

## What Claude Should Always Do

- Use the Playwright MCP server to explore the live page before writing any Page Object or test
- Follow POM structure without being asked
- Suggest adding `.env` and `.auth/` to `.gitignore` if creating those artefacts
- Reference environment variables for any credential-related code
- Flag any pattern that would expose secrets or violate the above rules
- Ask clarifying questions before generating a Page Object if the MCP server is unavailable

## What Claude Should Never Do

- Hard-code credentials, tokens, or environment-specific URLs
- Place locators or direct Playwright calls in test files
- Generate code that targets production environments
- Assume a page's structure — ask if uncertain

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/graemethomsongold) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
