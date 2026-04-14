## navigatewealth

> **Product**: Navigate Wealth — a React SPA (Vite + TypeScript) for a South African financial advisory platform. Single `package.json`, not a monorepo.

# AGENTS.md

## Cursor Cloud specific instructions

**Product**: Navigate Wealth — a React SPA (Vite + TypeScript) for a South African financial advisory platform. Single `package.json`, not a monorepo.

**Backend**: Fully remote Supabase (Edge Functions via Deno/Hono). No local backend setup is needed. Supabase credentials are hardcoded in `src/utils/supabase/info.tsx`. The frontend connects to the deployed edge functions at `https://vpjmdsltwrnpefzcgdmz.supabase.co/functions/v1/make-server-91ed8379`.

### Dev commands

All standard commands are in `package.json`:

| Task | Command |
|------|---------|
| Install deps | `npm install` |
| Dev server | `npm run dev` (Vite, port 3000) |
| Build | `npm run build` |
| Tests | `npm test` (vitest) |

### Notes

- After changing Edge Function behaviour (for example `GET /admin/stats` fields such as `incomplete` or `draft`), deploy the Supabase Edge Function so production returns the updated JSON; the SPA always calls the deployed function URL above.
- No ESLint config exists in the repo; there is no lint command.
- The test suite has a pre-existing issue: `resolveNestedKey.test.tsx` uses custom assertion logging instead of vitest `test()`/`it()` functions, so vitest reports it as "failed suite" despite all 17 internal assertions passing. 64 tests across 3 other suites pass cleanly.
- The Vite dev server opens a browser by default (`server.open: true` in `vite.config.ts`).
- `tsconfig.json` is at `src/tsconfig.json` (not project root).
- Path alias `@` maps to `./src` (configured in both `vite.config.ts` and `tsconfig.json`).
- For user-visible UI changes, build-only verification is not enough. Run the browser inspection tool before sign-off: `npm run ui:inspect -- --path /your-route --output tmp/ui-inspect/check.png` (or pass `--url` for an already-running environment). Use `--click` and `--wait-for` when the UI state requires interaction, such as opening a modal or dropdown.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShawnDaPrawn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
