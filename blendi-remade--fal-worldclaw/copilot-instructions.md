## fal-worldclaw

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# WorldClaw on fal

Browser reimplementation of the WorldClaw paper (arXiv 2608.05248) on fal.ai endpoints.

- `src/lib/` is the pipeline: `director.ts` orchestrates (browser-side), `terrain.ts`/`placement.ts`/`procedural.ts` are deterministic builders (seeded — never introduce Math.random), `fal.ts` wraps the fal endpoints, `validate.ts` repairs LLM spec JSON, `prompts.ts` holds the agent system prompts (the WorldSpec schema contract lives there — keep it in sync with `types.ts`).
- Every fal endpoint the client calls MUST be listed in `src/app/api/fal/proxy/route.ts` allowlist.
- Remote images/GLBs must load through `mediaUrl()` (same-origin relay) or canvases taint and the refinement screenshots break.
- Verify changes: `npx tsx scripts/smoke.ts` (core invariants), `node scripts/snap.mjs` (Playwright visual pass against `npx next dev -p 3777`), `npm run build`.

---
> Source: [blendi-remade/fal-worldclaw](https://github.com/blendi-remade/fal-worldclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
