## e2e-playwright

> Playwright E2E — structure, selectors, seeder


# Playwright E2E

## Run
```bash
make up && make e2e-seed && make playwright
```
Port **8081**. User: `e2e@finanzamente.test` / `password` (`E2ESeeder`). CI: `.github/workflows/playwright.yml`.

## Projects
| Project | Role |
|---------|------|
| `setup` | `auth.setup.ts` → `e2e/.auth/user.json` |
| `pubblico` | No session (public + auth flows) |
| `autenticato-desktop` | Session, Chrome |
| `autenticato-mobile` | Session, Pixel 5 |

## Rules (must)
1. New visible page → ≥1 E2E smoke test.
2. UI text/nav/field changes → update matching specs.
3. New area → `e2e/<area>/<area>.spec.ts`.
4. Selectors: `getByRole` / `getByLabel` / `getByText` > CSS; `#id` only if explicit in markup.
5. Unique data: `` `Name E2E ${Date.now()}` ``.
6. Authenticated specs: use `storageState` from setup — no manual login (except `e2e/auth/`, `e2e/public/`).
7. Specs independent — no order assumptions.
8. Assertions match **Italian** UI copy.
9. New seed data → extend `database/seeders/E2ESeeder.php`.
10. E2E env: `FEATURE_GUIDED_CREATE_FORMS=false` on `app_e2e`; after env change run `make e2e-seed`.

Full tree and conventions: `docs/agent/e2e-conventions.md`.

---
> Source: [mnossa/finanzamente](https://github.com/mnossa/finanzamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
