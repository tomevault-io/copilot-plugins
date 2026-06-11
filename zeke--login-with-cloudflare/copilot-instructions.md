## login-with-cloudflare

> This project is a Cloudflare Workers demo and reference implementation for self-managed OAuth clients. It deploys to `https://login-with-cloudflare.ziki.boo`.

# Agent instructions

## Project

This project is a Cloudflare Workers demo and reference implementation for self-managed OAuth clients. It deploys to `https://login-with-cloudflare.ziki.boo`.

## Stack

- Cloudflare Workers, not Pages.
- TypeScript.
- Hono for routing.
- Vitest for tests.
- ESLint for linting.
- Wrangler config lives in `wrangler.jsonc`.
- KV stores server-side OAuth sessions.

## Commands

- Install dependencies: `npm install`
- Generate Worker types: `npx wrangler types`
- Lint: `npm run lint`
- Typecheck: `npm run typecheck`
- Test: `npm test`
- Run all checks: `npm run ci`
- Local dev: `npm run dev`
- Deploy: `npm run deploy`

## Development rules

- Take a test-driven approach. Add or update tests before feature code when practical.
- Keep OAuth tokens server-side only. Never expose access tokens or client secrets to browser code.
- Use read-only Cloudflare OAuth scopes unless a task explicitly requires write access.
- Use `.env` for local environment variables and keep it ignored by git.
- Do not put secrets in `wrangler.jsonc`, source files, tests, or GitHub Actions workflow files.
- Run `npm run ci` before committing.
- Update this file when project-specific commands, conventions, deployment details, or gotchas change.

## Documentation split

- `README.md` is for humans. Keep it high-level and product-focused.
- Do not put file maps, script lists, or agent instructions in `README.md`.
- Put scripts, file layout, implementation notes, and operational details in this file.

## Design rules

- Use a single narrow centered column.
- Use giant heading text on the homepage.
- Keep the UI minimal and text-forward.
- Do not use cards.
- Product usage should render as lean lists with small status markers.
- Link prominently to the GitHub repo and include an Octocat icon.

## OAuth notes

- Hostname: `login-with-cloudflare.ziki.boo`.
- Cloudflare account ID: `d37edcc2a3a79f5a6df92ad287430b02`.
- OAuth redirect path: `/oauth/callback`.
- OAuth client ID: `e1c3992b54f25cbee54085d5eebb5af4`.
- OAuth client visibility: public.
- OAuth token endpoint auth method: `client_secret_post`. The Worker reads `OAUTH_TOKEN_AUTH_METHOD` (`client_secret_post` or `client_secret_basic`) and it must match the client registration, otherwise the token endpoint returns `invalid_client`.
- KV namespace `SESSIONS`: `037aed917778466f94b22c155c148d77`.
- Session cookies store an opaque KV session id plus an HMAC signature keyed by `SESSION_SECRET`. `getSession` verifies the signature before any KV lookup.
- Public OAuth client setup requires client URL domain verification through DNS.
- Use Cloudflare API MCP first for Cloudflare setup, `npx wrangler` second, and local browser DevTools only as a last resort.

## Deliberately deferred (not yet implemented)

These are out of scope for the current proof of concept. Implement carefully if needed:

- Multi-account selection. The dashboard uses the first authorized account only. A fuller demo would let users pick among authorized accounts.
- Refresh tokens. We do not request `offline_access` or refresh sessions; sessions simply expire after 15 minutes.
- PKCE. We only demonstrate the confidential server-side flow. Public/native clients should use PKCE with `token_endpoint_auth_method: none`.
- Token encryption at rest. Access tokens are stored in KV as JSON. For higher assurance, encrypt them with a server-side key before storing.

## CI and deploy

- Pull requests run lint, typecheck, tests, and Wrangler dry-run deploy.
- Pushes to `main` deploy to production.
- GitHub Actions expects these repo secrets:
  - `CLOUDFLARE_ACCOUNT_ID_PERSONAL_SIKELIANOS`
  - `CLOUDFLARE_API_TOKEN_PERSONAL_SIKELIANOS`
- The production deploy token must include Workers Scripts write access and KV Storage write access because the Worker has a KV binding.

## GitHub visibility

- The GitHub repository starts private.
- Make it public only after the demo works and is ready.
- Pushing to `main` is allowed during the initial project creation session only.

---
> Source: [zeke/login-with-cloudflare](https://github.com/zeke/login-with-cloudflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
