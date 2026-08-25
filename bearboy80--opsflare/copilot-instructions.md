## opsflare

> Opsflare is an Expo / React Native Cloudflare multi-account client.

# Repository Guidelines

## Project Structure & Module Organization

Opsflare is an Expo / React Native Cloudflare multi-account client.

- `app/` — Expo Router layouts and screens (onboarding, unlock, five-tab shell).
- `src/auth/` — local app account, password/biometric unlock, route guards.
- `src/cloudflare/` — REST/GraphQL clients, OAuth, account-scoped resources.
- `src/components/` — shared UI. Reusable primitives live in `src/components/ui/`.
- `src/theme/` — color tokens; do not scatter hex values through components.
- `src/i18n/` — English source copy plus translations in `locales/` (`en.json`, `zh-Hans.json`). Render text via `t()` keys.
- `oauth-relay/` — Cloudflare Worker that 302s the OAuth callback into `opsflare://`.

## Build, Test, and Development Commands

```sh
npm install
npm start
npm run ios
npm run android
npm test -- --runInBand
npx tsc --noEmit
```

`npm start` launches Metro. Tests use Jest via `jest-expo` with React Native Testing Library.

## Coding Style & Naming Conventions

Use TypeScript with two-space indentation. Name React components and contexts in PascalCase, hooks with a `use` prefix, and ordinary modules in camelCase (for example, `localAccount.ts`). Prefer the `@/` alias for app imports.

## Testing Guidelines

Add tests as `*.test.ts` or `*.test.tsx` in a nearby `__tests__/` directory. Cover authentication state changes, storage behavior, and account-scoped data logic. UI changes require manual checks on both iOS and Android.

## Commit & Pull Request Guidelines

History follows Conventional Commits, often with concise Chinese descriptions: `feat: ...`, `docs: ...`, and `chore: ...`. Keep each commit focused. Pull requests should summarize scope, list commands and devices tested, and include before/after screenshots for visual changes.

## Security & Configuration

Never commit API tokens, OAuth secrets, or local credentials. OAuth needs a
deployed `oauth-relay/` Worker URL plus a Cloudflare OAuth client that uses
that `/callback` address; put both in `.env` (see `.env.example`) and do not
hard-code them.
Production secrets belong in `expo-secure-store`, not AsyncStorage, SQLite,
fixtures, or logs. Do not commit generated output, `node_modules/`, `.env`,
or `.worktrees/`.

---
> Source: [bearBoy80/opsflare](https://github.com/bearBoy80/opsflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
