## live-classroom

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# Working in this repo

- Gates: `npm run typecheck && npm run lint && npm test` must pass before any change is done; `npm run build && npm run verify` for anything touching the server or config.
- **Lessons cost real money.** Never start a lesson, queue a topic, or call fal to test — one lesson is twelve paid video renders. The free path is `npm run verify` (boots the app with a blank key and proves no spend is admitted).
- All prompts and the teacher live in `src/lib/classroom-config.ts`. The character sheet must stay short numbered lines: fal's prompt rewriter copies lists verbatim but paraphrases prose and silently drops features.
- After any prompt change, validate with `node --experimental-strip-types scripts/probe-h3-expansion.mjs` (renders ONE paid clip, prints which character-sheet lines survived the rewrite) — only with the owner's consent, since it spends their fal credit.
- The lesson runtime is an in-memory singleton; server-code changes need a dev-server restart to take effect.
- Never commit `.env.local` or `recordings/`.

---
> Source: [internetphysics/live-classroom](https://github.com/internetphysics/live-classroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
