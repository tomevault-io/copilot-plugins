## crm

> You should always check and see if there are any relevant skill files you should review before starting a task e.g. if you're working on better-auth, always review the better auth best practice skill - if you're working on prisma, review your prisma-database-setup skill.

# Strict rules - Always review before starting any work

You should always check and see if there are any relevant skill files you should review before starting a task e.g. if you're working on better-auth, always review the better auth best practice skill - if you're working on prisma, review your prisma-database-setup skill.

Please check below, if you're working on anything related review the rules and let the user know you've read them:

## Design
Read @docs/design.md

## API:
Read @docs/api.md

## The research agent (`apps/agent`):
Read @docs/agent.md

Every piece of intelligence in this repo lives there, not in the API. The
complete eve documentation ships in `apps/agent/node_modules/eve/docs` and
matches the installed version — read the relevant guide before writing eve code
rather than working from memory of the API.

## Environment / configuration:
Read @docs/environment.md

There is **one `.env`, at the root of the repo**, and `.env.example` is its
documentation. If you add a variable, add it to `.env.example` with a note on
what it does — and if the API reads it, declare it in
`apps/api/src/config/env.validation.ts` too. Never add a per-package `.env`.

Anything a self-hoster might not have is optional, and the code must work
without it: a missing key removes a capability, it never throws. See
`apps/agent/agent/lib/capabilities.ts` for the pattern.

## Contributing / licence

This repository is public and MIT-licensed. Before writing anything that ships:
no real customer names, addresses or company data in fixtures, tests,
screenshots or docs — the seed in `packages/db/prisma/seed.ts` is the source of
demo data. See @CONTRIBUTING.md and @SECURITY.md.

---
> Source: [trycompai/crm](https://github.com/trycompai/crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
