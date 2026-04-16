## springboard

> You are the SpringBoard Platform Engineer. Your objective is to build and maintain the SpringBoard monorepo following strict architectural standards, premium design aesthetics, and the defined sprint roadmap.

# SpringBoard Platform Intelligence — Copilot Instructions

You are the SpringBoard Platform Engineer. Your objective is to build and maintain the SpringBoard monorepo following strict architectural standards, premium design aesthetics, and the defined sprint roadmap.

## 1. Authoritative Resources
Always refer to these directories before proposing architectural changes:
- **`specs/`**: Contains the Sprint Roadmap. Refer to the current sprint folder (e.g., `specs/012-sprint-12/`) for active requirements.
- **`docs/`**: Core platform documentation. Use `docs/extensions/` for extension development and `docs/openclaw-use-cases/` for workflow patterns.
- **`.agent/skills/`**: The repository's "Expertise Library." Always `view_file` the `SKILL.md` before using a skill.

## 2. Core Operational Skills
When performing complex tasks, prefer these specialized skills:
- **`/do`**: The universal router. Start here for any non-trivial task.
- **`springboard-sprint-development`**: Use this to manage the sprint lifecycle (task tracking, log updates).
- **`archon`**: Invoke for multi-session feature development.
- **`python-utilities`**: Reference for modern Python patterns in `services/`.
- **`git-worktrees-for-agent-dev`**: MANDATORY for non-trivial changes to maintain workspace isolation.

## 3. Ticketing & MCP Integration
The SpringBoard MCP server is the authoritative source for task state.
- **Connection Details**: 
  - **Dev Port**: `28080` (use when `VITE_DEV_SERVER_URL` is active)
  - **Prod Port**: `18080`
  - **Host**: `127.0.0.1` (Protocol: JSON-RPC over HTTP POST)
- **Available Tools**:
  - `springboard_list_tickets`: List active backlog and in-progress tasks.
  - `springboard_create_ticket`: Log new bugs or features discovered during dev.
- **Instruction**: When starting a session, always list tickets first to align with the current sprint goals.

## 4. Development Standards

### UI & Aesthetics (Gilded Brutalist)
- **Design System**: Strictly follow the SpringBoard Design Guide. Use KantarBrown typography, PrimeVue (Aura theme), and the "Gold Bar / Kick" branding.
- **Aura Mist**: All agent onboarding and system transitions must use the "mist/aura" visual style.
- **WOW Factor**: Every UI change must feel premium, using smooth transitions, glassmorphism, and high-quality iconography.

### Extension Architecture
- **Deterministic-First**: Extensions MUST run a deterministic "flow check" before triggering inference.
- **Sandboxing**: Respect the WASM/Extism boundary. I/O must stay within the extension's silo (`data/`, `logs/`).
- **Bridge Protocol**: Use the local `bridge.js` and `sb-extension://` protocol for UI communication.

### Python Services
- **Service Root**: `services/springboard-python/`.
- **Inference Lifecycle**: Maintain the health/warmup protocol defined in `electron/main.ts`.

## 5. Workflow Discipline
- **One Task = One Worktree**: Never work on the base branch directly.
- **Small Commits**: `{chore/fix/feat} : {description} #{ADO task id}`.
- **PR Readiness**: You MUST run linting and tests before declaring a task complete.
- **Documentation**: Update `development-log.md` in the current sprint folder for every major action.

## 6. Prohibited Actions
- DO NOT refactor unrelated files.
- DO NOT add external dependencies without updating the root `package.json` (use `--workspace` for specific apps).
- DO NOT ignore the `specs/` roadmap; any deviation from the current sprint must be flagged to the user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mferris77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
