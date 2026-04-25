## botlanes

> Kanban mission control for gstack agent tasks. Preact frontend, TypeScript backend.

# AGENTS.md

Kanban mission control for gstack agent tasks. Preact frontend, TypeScript backend.

## Design System
Always read DESIGN.md before making any visual or UI decisions.
All font choices, colors, spacing, and aesthetic direction are defined there.
Do not deviate without explicit user approval.
In QA mode, flag any code that doesn't match DESIGN.md.

## Project Architecture & Maintenance

### Frontend (Preact + No-Build)
- The frontend is located in `src/public/app.js` and `src/public/style.css`.
- **No Build Step:** The UI uses Preact via native ES modules imported from `esm.sh`. 
- **Templating:** Do not write JSX. Use the `htm` library (`html\`...\``) for templating inside `app.js`.
- **Styling:** The app uses Tailwind CSS loaded via CDN for most styling. Utility classes should be added directly to the template strings in `app.js`. Complex or shared custom components (like the board grid or attachment section) use `src/public/style.css`.

### Backend & Agent Execution
- The backend is a Bun server written in TypeScript (`src/server.ts`).
### Backend & Agent Execution
- **Agent Runner:** The application executes agent tasks by directly spawning the local `claude` CLI as a subprocess (via `Bun.spawn`), not an API or a gateway.
- **Agent Conversation:** When a human posts a comment to a card in a stage with an active skill, the backend automatically triggers the agent (or resumes the current run) to process the comment as a direct message. This ensures every comment is a conversation with the agent.
- **Projects & Workspaces:** Cards can be associated with a `Project`. When an agent runs a card, the backend uses the project's `directory` as the `cwd` for the Claude CLI process.
### State Management
- State is managed via SQLite (using `bun:sqlite`) and stored in `.gstack/botlanes.db`.
- `state.ts` handles all CRUD operations for `Projects` and `Cards` using targeted SQL queries for scalability.
- **Cascade Deletions:** When a project is deleted, all associated cards must be cascade-deleted.

### Multimedia
- Attachments (images, videos, documents, audio) are stored in `.gstack/botlanes-uploads/`.
- **Limits:** 10MB per file, 20 files per card, 50MB total per card.
- **Agent Context:** Attachments are passed to the agent as `[media attached: <path> (<mime-type>)]`. Agents can read these files directly if they have file system access (e.g., Claude Code).

### Commands
- Start the server: `bun run src/cli.ts start`
- Run tests: `bun test`
- Typecheck: `bunx tsc --noEmit`

## gstack
Use the `/browse` skill from `gstack` for all web browsing. Never use `mcp__claude-in-chrome__*` tools.

### Available Skills:
- `/office-hours`
- `/autoplan`
- `/plan-ceo-review`
- `/plan-eng-review`
- `/plan-design-review`
- `/design-consultation`
- `/review`
- `/ship`
- `/land-and-deploy`
- `/canary`
- `/benchmark`
- `/browse`
- `/qa`
- `/qa-only`
- `/design-review`
- `/setup-browser-cookies`
- `/setup-deploy`
- `/retro`
- `/investigate`
- `/document-release`
- `/codex`
- `/cso`
- `/careful`
- `/freeze`
- `/guard`
- `/unfreeze`
- `/gstack-upgrade`

If `gstack` skills are not working, run `cd .claude-personal/skills/gstack && ./setup` to build the binary and register the skills.

---
> Source: [developdaly/botlanes](https://github.com/developdaly/botlanes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
