## agent-control-cli

> CLI tool (`agent-control` / `agentctl`) for managing AI agent repositories and installing agents/skills into Kiro.

# AGENTS.md — agent-control-cli

CLI tool (`agent-control` / `agentctl`) for managing AI agent repositories and installing agents/skills into Kiro.

## Build & Test

```bash
npm run build          # tsc → dist/
npm run dev            # tsc --watch
npm test               # vitest run (all tests in tests/**)
npm run test:watch     # vitest interactive
npm run test:coverage  # v8 coverage report
```

All imports within `src/` use the `.js` extension (ESM `"type": "module"`) even for `.ts` source files — keep this pattern when adding imports.

## Architecture

Three logical layers, each with a clear boundary:

| Layer | Directory | Purpose |
|---|---|---|
| CLI entry | `src/cli.ts` | Registers `commander` commands → delegates to commands layer |
| Commands | `src/commands/` | Single-responsibility async functions (`installAgent`, `addRepo`, …) |
| Tools (interactive) | `src/tools/` | Wraps commands with `@inquirer/prompts` UI for interactive mode |

`src/commands/index.ts` is the single re-export barrel for commands; `src/utils/index.ts` is the barrel for utilities.

## Key Data Flows

**Install flow:** `installAgent(repo, agentId)` → `loadRepoMeta(repo)` → `installAgentFiles(agent, repo)` in `symlinks.ts` → copies files to `~/.agent-control/agents/<id>/`, rewrites `file://` and `skill://` paths in the agent's JSON, then creates two symlinks in `~/.kiro/agents/`: `agent-control_<id>.json` and `agent-control_<id>/` (files dir). Symlink paths are tracked in `~/.agent-control/config.json` under `symlinks`.

**Skill auto-install:** When an agent JSON references `skill://skills/<name>/SKILL.md`, `symlinks.ts::resolveSkillReference` auto-installs the skill from the same repo before writing the agent JSON. Installed skill symlinks land in `~/.kiro/skills/agent-control_<skillId>/`.

**Repo analysis:** `addRepo` / `update` → `analyzeRepository(name, path)` in `analyzer.ts` → scans all `.json` files for agent manifests (must have `name`, `description`, `prompt`) and `skills/<dir>/SKILL.md` for skills (YAML front-matter `name` + `description` required) → writes `~/.agent-control/repos/<name>.meta.json`.

## Runtime State

All state is under `~/.agent-control/`:

- `config.json` — installed agents, skills, and symlink registry (`{ agents, skills, symlinks }`)
- `repos/<name>/` — cloned or symlinked repo (symlink = local path)
- `repos/<name>.meta.json` — cached repo analysis output (`RepoMeta`)
- `agents/<id>/` — copied agent files + rewritten JSON
- `skills/<id>/` — copied skill directory

Agent/skill IDs are 16-char hex SHA-256 hashes of `"<repoName>:<agentIdField|jsonPath>"` (see `utils/hash.ts`).

## Conventions

**Testing pattern — `vi.doMock` + `vi.resetModules`:** All tests use dynamic `vi.doMock` (not static `vi.mock`) combined with `vi.resetModules()` in `beforeEach`, then re-import the module under test. This is required because modules are ESM and share a module cache across tests.

```ts
beforeEach(() => { vi.resetModules(); vi.restoreAllMocks(); });
it('…', async () => {
  vi.doMock('../../src/utils/index.js', () => ({ readConfig: vi.fn(…) }));
  const { installAgent } = await import('../../src/commands/install.js');
});
```

**`@inquirer/prompts` is always lazy-imported** inside functions (`await import('@inquirer/prompts')`) to keep modules testable without a TTY.

**`DiagnosticResult`** (`{ issues: number; fixes: Array<() => void> }`) is the shared return type for all `utils/doctor/check-*.ts` functions. Fixes are collected and applied together in `commands/doctor.ts`.

**Symlinks use `agent-control_` prefix** on every file/dir placed in `~/.kiro/` to avoid name collisions with manually-installed agents.

## Key Files

- `src/types.ts` — all shared interfaces (`Agent`, `Skill`, `Config`, `RepoMeta`, `AgentJson`)
- `src/constants.ts` — all file-system paths derived from `~`
- `src/symlinks.ts` — install/uninstall logic, path rewriting, rollback
- `src/analyzer.ts` — repo scanning and meta persistence
- `src/utils/doctor/` — modular diagnostic checks, each returning `DiagnosticResult`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michael-braun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/michael-braun)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
