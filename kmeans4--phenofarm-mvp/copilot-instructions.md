## phenofarm-mvp

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## PhenoFarm Codex Workspace

Read `.codex/project-context.md` before making changes. It contains the repo's current service bindings, validation commands, and deployment targets.

### Operating rules
- Never hardcode secrets. Use env vars from local env files or platform config.
- This repo has both local and production database targets. Confirm which env file is active before running migrations, auth changes, or deploys.
- Prefer existing npm scripts for build, test, lint, and Prisma work instead of ad hoc commands.
- For deployable changes, run the smallest meaningful validation before deploy and verify the production URL after deploy.
- Do not expose Stripe or auth secrets in code, logs, docs, or test fixtures.

---
> Source: [kmeans4/phenofarm-mvp](https://github.com/kmeans4/phenofarm-mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
