## wolf

> **W**orkflow of **O**utreaching, **L**inkedIn & **F**illing

# CLAUDE.md — wolf

## What is wolf

**W**orkflow of **O**utreaching, **L**inkedIn & **F**illing

AI-powered job hunting CLI + MCP server. Finds roles, tailors resumes, fills forms, and sends outreach emails. Can be invoked by other agents (e.g. OpenClaw).

## Current milestone

**Milestone 1 — Scaffolding & Skeleton** (in progress)

Goal: wolf is runnable as both a CLI and an MCP server, all subcommands registered (stubs ok).

See [docs/overview/MILESTONES.md](docs/overview/MILESTONES.md) for full milestone plan.

## Project structure

```
wolf/
├── src/
│   ├── cli/              # CLI entry + AppContext (DI container)
│   │   ├── index.ts          # commander.js setup, parses args, calls AppContext
│   │   └── appContext.ts     # manual DI — wires repo + service + application
│   ├── mcp/              # MCP entry — paused for this milestone
│   ├── commands/         # Thin wrappers between CLI and application layer
│   │   ├── hunt/
│   │   ├── tailor/
│   │   ├── fill/
│   │   ├── reach/
│   │   ├── score/
│   │   └── status/
│   ├── application/      # Application services — use-case orchestration
│   │   ├── <name>.ts         # interface
│   │   ├── impl/             # real + mock implementations
│   │   └── model/            # DTOs returned by application services
│   ├── service/          # Domain services — single-responsibility business ops
│   │   ├── <name>.ts         # interface
│   │   ├── impl/             # real + mock implementations
│   │   └── model/            # DTOs returned by domain services
│   ├── repository/       # Data access — SQLite + workspace file I/O
│   │   ├── <name>.ts         # interface
│   │   └── impl/             # concrete backends
│   ├── errors/           # Typed custom error classes
│   ├── types/            # Shared domain types (Job, Company, UserProfile, ...)
│   └── utils/            # Cross-cutting helpers (logger, config, env) — not a layer
├── data/                 # Local DB and runtime data (gitignored)
├── docs/
└── package.json
```

**Layer dependency direction:** `CLI/Commands → Application → Service → Repository → Types`. Each layer may only depend on the layers below it.

**DI entry point:** `src/cli/appContext.ts` constructs all implementations and provides them to commands. Swap real for mock by changing that one file — nothing else.

**MCP is paused.** When reactivated, it will sit next to `src/cli/`, also call `createAppContext()`, and delegate to the same application layer.

## Tech stack

| Layer | Tool |
|---|---|
| Language | TypeScript + Node.js |
| CLI framework | commander.js |
| MCP server | MCP SDK |
| Job data | Pluggable provider system |
| AI | Claude API (anthropic SDK) |
| Resume rendering | LaTeX (xelatex) → PDF |
| Cover letter rendering | md-to-pdf (Markdown → PDF) |
| Browser automation | Playwright |
| Local storage | SQLite |
| Email | Gmail API (OAuth2) |
| Config | `wolf.toml` in workspace root |

## CLI commands

| Command | Description |
|---|---|
| `wolf init` | Interactive setup wizard |
| `wolf hunt` | Find and score jobs |
| `wolf tailor` | Tailor resume to a JD |
| `wolf fill` | Auto-fill job application form |
| `wolf reach` | Find HR contacts and send outreach |
| `wolf status` | List tracked jobs with status/score |
| `wolf env show` | List WOLF_* keys and whether they are set |
| `wolf env clear` | Remove WOLF_* export lines from shell RC files |

## MCP tools

`wolf_hunt`, `wolf_tailor`, `wolf_fill`, `wolf_reach`

## Environment variables

API keys use a `WOLF_` prefix and are stored as shell environment variables (not in `.env` files — workspace may be cloud-synced or shared with resumes). Use `wolf env set` to configure them.

```
WOLF_ANTHROPIC_API_KEY=
WOLF_APIFY_API_TOKEN=
WOLF_GMAIL_CLIENT_ID=
WOLF_GMAIL_CLIENT_SECRET=
```

Add to `~/.zshrc` (Mac/Linux) or User Environment Variables (Windows).
Run `wolf env show` to verify, `wolf env clear` to remove.

For MCP server usage, add these to the `env` section of `claude_desktop_config.json`.

## Testing conventions

- Unit test files go in a `__tests__/` folder adjacent to the source they test
  - e.g. `src/utils/__tests__/config.test.ts` tests `src/utils/config.ts`
  - e.g. `src/commands/env/__tests__/env.test.ts` tests `src/commands/env/index.ts`
- Test files are named `<subject>.test.ts`
- Use Vitest; run with `npm test`

## Workflow rules

- Do NOT commit or push without explicit user approval. Always show changes and wait for confirmation first.
- When a significant design decision is made during conversation, ask the user: "Should we record this in DECISIONS.md?" (This rule itself is recorded in docs/design/DECISIONS.md — 2026-03-20.)

## Implementation plans

- Plans (from `superpowers:writing-plans` or similar) go in `docs/plans/`
- `docs/plans/` is gitignored except for `.gitkeep` — plans are personal working docs, not shared
- Naming convention: `YYYY-MM-DD-<feature-name>.md`

## Documentation rules

- All markdown documents (except `CLAUDE.md` and `README.md`) MUST have a Chinese version with `_zh` suffix (e.g., `ARCHITECTURE.md` → `ARCHITECTURE_zh.md`)
- When creating or updating an English doc, always create or update the corresponding `_zh` version in the same commit
- Chinese versions must stay in sync with their English counterparts

## Common commands

```bash
npm run build       # compile TypeScript
npm run dev         # run in watch mode
npm run test        # run tests
wolf --help         # CLI help
wolf mcp serve      # start MCP server
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guanyu-gerry-tao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
