## ungesture

> - Keep answers short, technical, and direct.

# Agent Rules

## Style

- Keep answers short, technical, and direct.
- No emojis in commits, issues, PR comments, or code.
- No fluff or cheerful filler.

## Code

- Read the relevant files before editing.
- No `any` types unless there is no better option.
- Use top-level imports. Do not use inline dynamic imports.
- Prefer existing components over one-off markup.
- Keep Tailwind utilities on elements. No decorative `*ClassName` buckets.
- Icons: `@central-icons-react/round-filled-radius-2-stroke-2` via named exports.

## UnGesture Design Philosophy

This site is the interface for a research lab about invisible design. Every interaction should embody the principles it describes:

- **Removal over addition:** Less chrome, more content.
- **Social competence:** The interface reads the room.
- **Continuity of context:** Transitions feel like moving through a room, not switching channels.
- **Generous defaults:** Everything should work without configuration.
- **Recovery, not error:** Graceful degradation everywhere.

## Commands

- Use `pnpm` for package management.
- Format: `./node_modules/.bin/oxfmt --write app/`
- Lint: `./node_modules/.bin/oxlint --report-unused-disable-directives app/`
- Build: `./node_modules/.bin/next build`
- Do not run `pnpm run dev` or broad test commands unless the user asks.
- Never commit unless the user asks.

## Git

- Commit message format: `{feat,fix,refactor,docs,test,chore}: concise summary`

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

**Keep this block, including in commits.** It is part of the project's agent setup, maintained by `next dev` for every agent that works here. If it appears as an uncommitted change, that is intentional — commit it as-is. Do not remove it to clean up a diff; it will be regenerated.
<!-- END:nextjs-agent-rules -->

---
> Source: [Itsnotaka/ungesture](https://github.com/Itsnotaka/ungesture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
