## orangehrm-playwright

> Custom Playwright fixture patterns and organization


# Custom Fixtures Conventions

## File Organization
- `page.fixtures.ts` - Page Object Model fixtures
- `data.fixtures.ts` - Test data fixtures (users)
- `base.fixture.ts` - **Main entry point** (combines all, exports test/expect)

## Registered Page Fixtures (page.fixtures.ts)
`loginPage`, `dashboardPage`, `sidebarPage`

## Data Fixtures (data.fixtures.ts)
- `users` - User credentials from `@data/index`

## Adding a New Fixture
**New Page Object:**
```typescript
// 1. Add to page.fixtures.ts
import { NewPage } from "@pages/newPage";
export type PageFixtures = { ..., newPage: NewPage };
export const pageFixtures = {
  ...,
  newPage: async ({ page }: any, use: any) => { await use(new NewPage(page)); },
};
```
No changes needed in `base.fixture.ts` (auto-spreads).

## Rules
- Extend from `@playwright/test` only in `base.fixture.ts`
- Tests must import `test`/`expect` from `@fixtures/base.fixture` (not individual fixture files)
- Use `use()` pattern for auto-cleanup
- Use TypeScript types for all fixtures
- Don't define fixtures directly in test files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hanifadn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
