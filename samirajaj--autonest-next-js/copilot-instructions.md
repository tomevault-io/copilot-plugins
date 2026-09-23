## autonest-next-js

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Project conventions

Read `docs/ARCHITECTURE.md` before making changes. It documents:
- Environment variable management (`src/lib/env.ts` as single source of truth)
- Database connection strategy (Mongoose vs native MongoDB driver)
- Auth configuration and session validation
- Domain model, state transitions, and failure handling

---
> Source: [samirajaj/autonest-next.js](https://github.com/samirajaj/autonest-next.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
