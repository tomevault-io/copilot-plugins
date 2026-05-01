## saasmail

> Self-hosted email server on Cloudflare Workers. See README.md for full documentation.

# saasmail

Self-hosted email server on Cloudflare Workers. See README.md for full documentation.

## Development

- Use `yarn` for all dependency commands (not npm)
- Backend: Hono + Zod OpenAPI routes in `worker/src/routers/`
- Frontend: React + Tailwind in `src/`
- Database: Drizzle ORM with D1 in `worker/src/db/`
- Run `yarn tsc --noEmit` to type-check before committing
- Run `yarn test` for tests

## Skills

- `/saasmail-onboarding` — Interactive setup wizard for deploying a new saasmail instance

---
> Source: [choyiny/saasmail](https://github.com/choyiny/saasmail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
