## qa-workflow

> QA AI Assessment — Cursor rules for consistent code generation


# QA Automation Rules

- All page objects extend BasePage from `./BasePage.js`
- Use `getByTestId()` for selectors (app uses `data-test` attributes)
- All test data from `../../test-data/testData.js` — never hardcode credentials
- API tests use `ApiClient.js` — never create raw `request.newContext()` in spec files
- Tag test.describe blocks with `@smoke` or `@regression` in the description string
- `waitForPageLoad()` after every navigation
- API tests: create fresh ApiClient in beforeEach, dispose in afterEach
- Use timestamp suffix for registration emails: `Date.now()`
- No `page.waitForTimeout()` over 2000ms — use proper waits instead

---
> Source: [tusharguptattn/AssignmentSE](https://github.com/tusharguptattn/AssignmentSE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
