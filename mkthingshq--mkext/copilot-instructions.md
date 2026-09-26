## mkext

> This file provides guidance to coding agents working in this repository.

# AGENTS.md

This file provides guidance to coding agents working in this repository.

## Project overview

MkExt is a WXT browser-extension template and the browser companion for a
TanStarter SaaS application. It uses React 19, TypeScript, Tailwind CSS
v4, Better Auth, TanStack Query, and WXT's cross-browser APIs.

The extension targets Chrome Manifest V3 and Firefox. It contains popup,
options, side-panel, new-tab, DevTools, background, and unlisted tab
entrypoints. It includes separate Domain Rating demonstrations: Google Search
result badges and a floating panel for ordinary HTTP/HTTPS pages. Narrow the
floating panel's match scope before using it in a production product.

## Commands

```bash
bun install              # Install dependencies and run WXT prepare
bun run dev:chrome       # Start Chrome development mode
bun run dev:firefox      # Start Firefox development mode
bun run build            # Build and zip Chrome and Firefox extensions
bun run typecheck        # Run TypeScript without emitting files
bun run lint             # Run Biome checks without modifying files
bun run lint:fix         # Apply Biome safe fixes and formatting
bun test                 # Run Bun unit tests when relevant
bun run postinstall      # Regenerate WXT types after public/config changes
```

Before completing a code change, run `bun run typecheck` and the narrowest
relevant Biome or test command. Run `bun run build` for changes to WXT config,
entrypoints, public assets, manifest localization, or packaging.

## Architecture

- `src/app/` contains WXT entrypoints. Keep entrypoint files thin.
- `src/components/` contains shared React UI, grouped by feature.
- `src/lib/` owns authentication, storage, messaging, URLs, and utilities.
- `src/types/` owns exported TypeScript types and global declarations.
- `src/assets/` contains bundled styles and the source extension icon.
- `public/` contains files copied into the extension package unchanged.
- `build/` and `.wxt/` are generated; do not edit them manually.

Use `~/` for imports from `src`. Prefer WXT's `browser` and storage APIs over
direct Chrome-only APIs unless a feature is intentionally browser-specific.

## Internationalization

`src/locales/{en,zh_CN}.json` is the source of truth for localized strings. The WXT
i18n module generates the browser `_locales` files used by manifest `__MSG_*__`
placeholders. Application code reads the same source dictionaries through
`src/lib/i18n.ts`.

The UI follows the browser language until the user chooses English or Chinese
in the locale switcher. The override is stored under `local:locale`.
Browser-owned manifest labels still follow the browser language. Keep the
English and Chinese dictionaries key-compatible.

## Authentication and extension identity

- The extension is a Better Auth client for the TanStarter backend (internally
  mkfast) configured by
  `VITE_AUTH_URL`.
- Persist the bearer token through `src/lib/auth.ts` and WXT storage. Do not log
  tokens, passwords, OAuth callbacks, or private user data.
- Extension requests that use bearer authentication must preserve the existing
  credential behavior in `src/lib/auth.ts`.
- `wxt.config.ts` contains a fixed manifest key for a deterministic development
  extension ID. Changing it requires updating the backend's trusted origins.
- Google sign-in uses Chromium's `identity.getAuthToken()` (see
  `src/lib/oauth.ts`), then exchanges the short-lived Google access token for a
  product bearer token at `/api/auth/extension/google`. The Google token is
  never treated as a session. Firefox has no `getAuthToken()`, so the flow
  reports `OAUTH_UNAVAILABLE` there; provider configuration and the end-to-end
  flow still require real manual verification.

## Code style

Biome is the source of truth:

- 2-space indentation and 80-character line width;
- double quotes and semicolons;
- kebab-case file names;
- PascalCase React components;
- camelCase functions and hooks;
- named exports;
- external imports before `~/` imports.

Use React Hook Form with Zod for forms and TanStack Query for server state.
Use `@tabler/icons-react` for icons and `cn()` from `src/lib/utils.ts` for class
composition. Follow the existing Base UI component patterns in
`src/components/ui/`.

## Extension constraints

- Request the minimum manifest permissions and host permissions required.
- Keep background workers event-driven and content scripts isolated with
  explicit cleanup.
- Treat popup width, portal-based overlays, dark mode, keyboard navigation, and
  cross-browser behavior as user-facing requirements.
- Use `public/logo.png` for the product logo and `src/assets/icon.png` for the
  generated extension icons.
- No SVG-to-React transform plugin is configured. Use an image asset or a small
  React component when an SVG-based visual is required.
- Do not publish or submit an extension build unless explicitly requested.

Keep changes narrow and preserve unrelated staged or unstaged work.

---
> Source: [MkThingsHQ/mkext](https://github.com/MkThingsHQ/mkext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
