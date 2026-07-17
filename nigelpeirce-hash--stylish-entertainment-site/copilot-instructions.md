## update-docs-on-major-changes

> Update CURSOR_CONTEXT and DISASTER_RECOVERY when making major changes


# Update Project Docs on Major Changes

When you make **major** changes to this codebase, update these shared docs so all agents (and humans) stay in sync:

## Documents to Update

1. **CURSOR_CONTEXT.md** – Agent familiarisation, tech stack, routing, key features, env vars
   - Add new routes, features, or architectural changes
   - Update "Last updated" date

2. **DISASTER_RECOVERY_GUIDE.md** – Rebuild guide, env vars, DB, services, deployments
   - Add new env vars or external services
   - Document new API keys or integrations
   - Bump version and "Last Updated" date

## What Counts as "Major"

- New pages, routes, or significant features
- New environment variables or external services
- Database schema changes
- New API routes or integrations
- Architecture or build/deploy changes

## Minor Changes (No Doc Update Needed)

- Bug fixes, typo fixes, styling tweaks
- Content/copy changes
- Refactors that don't change behaviour

When in doubt, make a brief note in the relevant doc.

---
> Source: [nigelpeirce-hash/stylish-entertainment-site](https://github.com/nigelpeirce-hash/stylish-entertainment-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
