## naming-and-routes

> Naming, imports, typed routes, AppProviders


# Naming, imports, routes

- Files: **kebab-case**. Feature pages: `sth-page.tsx` exporting `SthPage`.
- Shared imports: `@/components`, `@/lib`, `@/messages`, ….
- Same-feature imports: **relative**.
- Navigation/redirects/Links: only via generated `routes.*()` from `src/lib/routes.ts`.
- **Do not edit `src/lib/routes.ts`** — it is gitignored; regenerate via `npm run routes:generate` / watch on `npm run dev`.
- Config aliases/extras: `scripts/routes.config.mjs`. See `docs/routes.md`.
- Post-login `next`: panel allowlist only; else dashboard.
- Client providers: one `AppProviders` module; document order in-file.
- `cn`: from shadcn CLI.

---
> Source: [soheilhasanjani/contently](https://github.com/soheilhasanjani/contently) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
