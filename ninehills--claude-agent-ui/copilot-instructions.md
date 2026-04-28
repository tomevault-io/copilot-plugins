## claude-agent-ui

> Browser-based UI + Bun HTTP/SSE server for agentic chat with the Claude Agent SDK.

# Claude Agent Web UI

Browser-based UI + Bun HTTP/SSE server for agentic chat with the Claude Agent SDK.

## Technology

- **Server:** Bun (HTTP + SSE)
- **UI:** React 19 + TypeScript, Tailwind CSS 4
- **Build:** Vite 7 (web)
- **Runtime:** Bun (package manager, scripts, tests)

## Layout

```
src/
├── server/     # Bun HTTP/SSE server + agent session manager
├── renderer/   # React Web UI
└── shared/     # Types shared between processes
.claude/        # Claude Agent SDK skills
resources/      # Bundled runtime binaries (bun, uv, etc.)
static/         # Icons and static assets
```

## Skills integration

- Skills live in `.claude/skills/<skill-name>/` (each contains `SKILL.md` + `scripts/`).
- Built-in sample: `workspace-tools` with a depth-limited `list-directory` utility.
- Skill TypeScript is checked via the root `tsconfig.json`; there is no package.json or node_modules under `.claude`.
- The server uses the specified agent directory and any `.claude` skills present there.

### Adding a skill

1. Create `.claude/skills/<skill-name>/SKILL.md` with `name` + `description`.
2. Add TypeScript tools under `.claude/skills/<skill-name>/scripts/`.
3. Start the server; the workspace `.claude` folder will be used at runtime.

## Commands

```bash
bun install       # Install dependencies
bun run typecheck # TypeScript checks
bun run lint      # ESLint
bun run test      # Bun tests
bun run format    # Prettier
bun run server    # Start Bun HTTP/SSE server
bun run dev:web   # Start Vite dev server for Web UI
bun run build:web # Build Web UI into dist/
bun run start     # Build Web UI and start server (single port)
bun run dev:single # Build Web UI in watch mode and start server (single port)
```

## Defaults

- Agent directory is required and passed via `--agent-dir` on server start.
- Initial prompt is optional via `--prompt`.
- Claude authentication is handled by your agent directory configuration (e.g. `.claude/settings.json`).
- Server listens on port 3000 by default; override with `--port`.

## Workflow

- After a series of code changes, **always** run: lint, typecheck, test, and format commands to ensure the code is working as expected.
- When drafting commit messages, **always** follow Conventional Commits format.

---
> Source: [ninehills/claude-agent-ui](https://github.com/ninehills/claude-agent-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
