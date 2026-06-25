## opencodelikeclaude

> | Command | Description |

# OpenCode Claude Parity Plugin - Agent Instructions

## Commands
| Command | Description |
|---------|-------------|
| `npm run build` | Compile TypeScript (`tsc`) |
| `npm run dev` | Watch mode (`tsc --watch`) |
| `npm run lint` | Lint (`eslint src --ext .ts`) |
| `npm run test` | Test once (`vitest run`) |
| `npm run test:watch` | Watch tests (`vitest`) |
| `npm run migrate` | Run CLI migration (`node dist/cli.js migrate`) |

CI: `lint -> build -> test` (`npm ci && npm run lint && npm run build && npm test`).  
Publish: `npm ci && npm run lint && npm test && npm run build && npm publish --provenance --access public` (tag `v*`).

## Architecture
- **Entry point**: `src/index.ts` → `createPlugin()`, `createOpenCodePlugin()`, `getPlugin()`, `reloadPlugin()`
- **CLI**: `src/cli.ts` → binary `opencode-claude-parity` with `migrate`/`check`/`config`/`help`
- **Module**: ESM (ES2022), extensionless imports (`"./builtin"` not `"./builtin.js"`)
- **Builtins auto-register** via side-effect imports in `src/index.ts`: `import "./builtin"`, `"./agents/builtin"`, `"./skills/builtin"`, `"./hooks/builtin"`, `"./mcp/loader"`
- **Registries**: `hookRegistry`, `commandRegistry`, `agentRegistry`, `mcpRegistry`, `skillRegistry` — all singletons
- **Dist**: `dist/` committed and in npm package; rebuild after changes
- **Node ≥18** (ESM, `import.meta`)

## Critical: Stubs
Agents, MCP, skills, review, research, verify, insights, cost, and command script execution all **return mock/stub data**. `REAL_IMPLEMENTATION.md` documents how to wire them to OpenCode's real subsystems (gated by `experimental.useRealImplementations`).  
**Files that work fully**: config, hooks, session, permissions, compat, CLI, builtin command handlers, utils.

## Config
`loadConfig(cwd)` searches (first found wins):
- `.opencode/opencode-claude-parity.jsonc`
- `.opencode/opencode-claude-parity.json`
- `~/.config/opencode/opencode-claude-parity.jsonc`
- `~/.config/opencode/opencode-claude-parity.json`

Validated with Zod, deep merged with defaults. TTL cache (5s).

## Conventions
- **Lint**: `@typescript-eslint/no-explicit-any` = warn; `@typescript-eslint/no-unused-vars` = warn (underscore prefix ignored)
- **No tests exist** — vitest configured (`include: ["src/**/*.test.ts"]`) but empty
- **Plan/Progress**: `PLAN.md` / `PROGRESS.md` — update PROGRESS.md after each session
- **Modules**: All under `src/` with 1:1 mapping to `dist/`; no barrel-index pattern — imports are per-directory

## Environment Variables
Copy `.env.example` to `.env` (`.env` is gitignored).
- `NVIDIA_NIM_API_KEY` — required for `useRealImplementations` (agents, skills, review, research)
- `OPENCODE_LOG_LEVEL` — debug, info, warn, error, silent
- `EXA_API_KEY` — websearch MCP
- `GITHUB_TOKEN` — grepApp MCP

---
> Source: [shaurya202/opencodelikeclaude](https://github.com/shaurya202/opencodelikeclaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
