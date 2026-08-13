## gamepocgen

> *Standard: jg-v1 | type: service*

*Standard: jg-v1 | type: service*
*Inherits global rules from ~/.claude/CLAUDE.md. Project knowledge: brain_query("gamePocGen ...", detail="full")*

# GamePocGen

AI-powered overnight generator for browser-playable 2D incremental games. Node service that runs a 5-phase agentic pipeline per game, then auto-deploys to `gamedemoN.namjo-games.com`.

## Commands

- Run backend: `cd backend && npm start`
- Test backend: `cd backend && npm run test:all`
- Test framework: `cd framework/core && node __tests__/run-tests.js` (also `mechanics/`, `ui/`)
- Deploy: `cd docker && cp .env.example .env && docker compose up -d`

## Map

- `docs/architecture.md` — how the system fits together (entry point)
- `docs/architecture/` — generated per-subsystem detail docs (overview, file-map, API, data models, pipelines, deployment, security)
- `docs/index.html` + `docs/docker-compose.yml` — the public docs website (deployable)
- `backend/` — Express API + PostgreSQL job store + services (queue, container, deployment, game-tester)
- `framework/` — vanilla-JS game framework the pipeline produces games from (core, mechanics, ui, sprites, css, starter)
- `docker/` — worker container, compose stack, idle-shutdown
- `harness/` — shell orchestration for Claude Code CLI sessions in containers
- `prompts/` — agent prompts per pipeline phase
- `gallery/` — generated-game listing page
- `scripts/` — repo automation (game smoke test)
- `verifier-results/` — pipeline verifier output (cleanup/security)
- `plans/` — active plans (`archived/` when done)

## Rules

- Games are vanilla JS + HTML/CSS — no build step. Keep it that way.
- All infra/credentials/subdomain details live in the Second Brain — query before guessing.
- `docs/architecture/` files are generated (marked `<!-- GENERATED -->`) — regenerate, don't hand-edit.
- `todo.md` is the live task list — sync via `brain_sync_todo(project="gamePocGen", path="todo.md")`.

---
> Source: [J-Gierend/gamePocGen](https://github.com/J-Gierend/gamePocGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
