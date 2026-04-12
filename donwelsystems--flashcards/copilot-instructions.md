## flashcards

> Playwright E2E testing conventions for this project


# Testing (Playwright)

This project uses [@playwright/test](https://playwright.dev) with `testDir: ./tests`, `baseURL` and `webServer` in `playwright.config.ts`. Prefer end-to-end checks that mirror real user behavior.

## Selectors and assertions

- Prefer **role-based** and accessible queries: `page.getByRole`, `getByLabel`, `getByText` before CSS or `nth()`.
- Use `expect` from `@playwright/test` so assertions **auto-retry** (e.g. `toBeVisible()`, `toHaveURL()`).
- Avoid `page.waitForTimeout()`; rely on Playwright’s built-in waiting.

```typescript
// ❌ Fragile
await page.locator('.card-0').click();

// ✅ Aligned with accessibility and stable copy
await page.getByRole('button', { name: 'Study Mode' }).click();
await expect(page.getByRole('heading', { name: 'Select a Category' })).toBeVisible();
```

## Structure

- Group related cases with `test.describe`; use `test.beforeEach` for shared navigation (e.g. `page.goto('/')`).
- Keep tests **order-independent**; do not rely on state left by another test.
- When the UI copy or labels change, **update tests deliberately** so expectations stay in sync with the product.

## Running tests

- Local: `npx playwright test` (dev server is started via `webServer` in config when needed).
- Use `npx playwright test --ui` or `--debug` when iterating on a failing test.

When adding `playwright.config.ts` or new projects/browsers, keep CI settings (`forbidOnly`, `retries`, `workers`) consistent with the existing config unless there is a deliberate change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/donwelsystems)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/donwelsystems)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
