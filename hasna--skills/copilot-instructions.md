## skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
bun install                       # Install dependencies
bun run build                     # Build CLI, MCP server, library, and type declarations
bun run dev                       # Run CLI in development mode (bun run src/cli/index.tsx)
bun test                          # Run all tests (Bun native test runner)
bun test src/lib/registry.test.ts # Run a single test file
bun run typecheck                 # Type-check without emitting (tsc --noEmit)
bun run dashboard:build           # Build Next.js dashboard (cd dashboard && bun install && bun run build)
bun run dashboard:dev             # Next.js dev server with HMR (port 3505, proxies /api to :3579)
bun run server                    # Start HTTP server (port 3579, serves dashboard + API)
bun run server:dev                # Start server with --watch
```

## Architecture

Monorepo containing 202 AI agent skills plus a framework for discovering, installing, and managing them. Four interfaces share a common set of core modules:

```
src/
├── cli/
│   ├── index.tsx                 # Commander.js CLI + Ink TUI entry
│   ├── cli.test.ts
│   └── components/
│       └── App.tsx               # Interactive TUI (React/Ink)
├── mcp/
│   ├── index.ts                  # MCP server (stdio transport)
│   └── mcp.test.ts
├── server/
│   ├── serve.ts                  # Bun.serve HTTP server + REST API
│   └── serve.test.ts
├── lib/
│   ├── registry.ts               # SKILLS array (202 entries) + CATEGORIES (17)
│   ├── installer.ts              # Install/remove to .skills/ and agent dirs
│   ├── skillinfo.ts              # Docs, requirements, env var extraction, run
│   ├── utils.ts                  # normalizeSkillName()
│   ├── registry.test.ts
│   ├── installer.test.ts
│   ├── skillinfo.test.ts
│   ├── skillinfo-run.test.ts
│   ├── utils.test.ts
│   └── validation.test.ts        # Structural validation: all 202 skills
├── index.ts                      # Library re-exports
└── index.test.ts

dashboard/                        # Vite + React 19 + Tailwind v4 + shadcn/ui
├── src/components/
│   ├── skills-table.tsx          # TanStack Table for skills
│   ├── skill-detail-dialog.tsx
│   ├── stats-cards.tsx
│   ├── theme-provider.tsx        # Dark/light/system (oklch tokens)
│   ├── theme-toggle.tsx
│   └── ui/                       # shadcn/ui primitives
├── package.json
├── vite.config.ts
└── tsconfig.json

skills/                           # 202 self-contained skill directories
├── _common/                      # Shared utilities for skills
├── skill-image/
├── skill-deep-research/
├── ...
└── tsconfig.base.json
```

### Interfaces

**CLI (`src/cli/index.tsx`)** -- Commander.js with Ink (React for terminal). Running `skills` with no args launches an interactive TUI. Subcommands: `install`, `list`, `search`, `info`, `docs`, `requires`, `run`, `remove`, `update`, `categories`, `init`, `mcp`, `serve`, `self-update`.

**MCP Server (`src/mcp/index.ts`)** -- Model Context Protocol server over stdio. 9 tools (`list_skills`, `search_skills`, `get_skill_info`, `get_skill_docs`, `install_skill`, `remove_skill`, `list_categories`, `get_requirements`, `run_skill`) and 2 resources (`skills://registry`, `skills://{name}`).

**HTTP Server** — not shipped in OSS. The SaaS dashboard and `/api/v1/*` endpoints live in the private `platform-skills` repo.

**Library (`src/index.ts`)** -- npm package `@hasna/skills` re-exporting registry, installer, and skillinfo modules.

### Core Modules

**`src/lib/registry.ts`** -- The `SKILLS` array (202 entries) with `SkillMeta` interface (name, displayName, description, category, tags) and `CATEGORIES` tuple (17 categories). Functions: `getSkill()`, `getSkillsByCategory()`, `searchSkills()`.

**`src/lib/installer.ts`** -- Copies skill source into `.skills/` in the user's project. Updates `.skills/index.ts` on every install/remove. Also supports agent-specific installs (copies SKILL.md to `~/.claude/skills/`, `~/.codex/skills/`, or `~/.gemini/skills/`). Functions: `installSkill()`, `installSkills()`, `removeSkill()`, `getInstalledSkills()`, `installSkillForAgent()`, `removeSkillForAgent()`, `resolveAgents()`.

**`src/lib/skillinfo.ts`** -- Reads docs (priority: SKILL.md > README.md > CLAUDE.md), extracts env vars and system deps via regex patterns, reads CLI command from package.json bin field, generates SKILL.md from metadata if missing, can execute skills via `runSkill()` (auto-installs deps, spawns via Bun). Functions: `getSkillDocs()`, `getSkillBestDoc()`, `getSkillRequirements()`, `runSkill()`, `generateEnvExample()`, `generateSkillMd()`.

**`src/lib/utils.ts`** -- `normalizeSkillName()` which prefixes `skill-` if missing.

## Key Patterns

### Skill Name Normalization

Registry uses bare names (`image`), filesystem uses prefixed names (`skill-image`). `normalizeSkillName()` in `src/lib/utils.ts` handles conversion. This applies everywhere: installer, skillinfo, server, MCP.

### Installation Modes

1. **Full source install** (default): copies entire `skills/skill-{name}/` to `.skills/skill-{name}/` in the project, auto-generates `.skills/index.ts` with re-exports.
2. **Agent install** (`--for claude|codex|gemini|all`): copies only SKILL.md to `~/.{agent}/skills/skill-{name}/SKILL.md`. If SKILL.md does not exist, one is generated from registry metadata + README.md/CLAUDE.md content via `generateSkillMd()`.

### Path Resolution

The `findSkillsDir()` function in `installer.ts` walks up to 5 parent directories from `__dirname` looking for a `skills/` directory. This makes it work from both `src/lib/` (dev) and `bin/` or `dist/` (built).

### TTY Detection

CLI checks `process.stdout.isTTY && process.stdin.isTTY`. If true, renders Ink TUI. If false (piped, CI), prints help text and exits. This is the `isTTY` variable at the top of `src/cli/index.tsx`.

### Skill Name Validation

The HTTP server validates skill names with `/^[a-z0-9-]+$/` to prevent path traversal attacks.

### ESM with .js Extensions

All imports use `.js` extensions even for `.ts` source files (e.g., `from "../lib/registry.js"`). JSON imports use `with { type: "json" }` syntax.

### Build Outputs

Three separate `bun build` invocations in the build script:
- CLI: `src/cli/index.tsx` -> `bin/index.js` (externals: ink, react, chalk)
- MCP: `src/mcp/index.ts` -> `bin/mcp.js`
- Library: `src/index.ts` -> `dist/index.js`
- Types: `tsc --emitDeclarationOnly --declaration --outDir dist`

### Environment Variable Extraction

`skillinfo.ts` scans docs and `.env.example` files using two regex patterns:
- `ENV_VAR_PATTERN` -- matches `*_API_KEY`, `*_TOKEN`, `*_SECRET`, etc.
- `GENERIC_ENV_PATTERN` -- matches known provider prefixes (OPENAI_, ANTHROPIC_, AWS_, etc.)

### System Dependency Detection

`skillinfo.ts` scans doc files for known tool names (ffmpeg, playwright, docker, pandoc, etc.) using regex patterns.

## Skill Structure Template

Each skill under `skills/skill-{name}/` follows this structure:

```
skills/skill-{name}/
├── src/
│   ├── index.ts          # Main entry / programmatic API
│   ├── commands/          # CLI command handlers (optional)
│   ├── lib/               # Core logic
│   ├── types/             # TypeScript types
│   └── utils/             # Utility functions
├── SKILL.md               # Skill definition (frontmatter: name, description)
├── README.md              # Usage documentation
├── CLAUDE.md              # Development guide (optional)
├── package.json           # Must have "bin" entry for runnable skills
└── tsconfig.json          # Extends ../tsconfig.base.json
```

## MCP Tools Reference

| Tool | Input Schema | Description |
|------|-------------|-------------|
| `list_skills` | `{ category?: string }` | List all skills, optionally filtered by category |
| `search_skills` | `{ query: string }` | Search by name, description, and tags |
| `get_skill_info` | `{ name: string }` | Metadata + requirements |
| `get_skill_docs` | `{ name: string }` | Best available doc (SKILL.md > README.md > CLAUDE.md) |
| `install_skill` | `{ name: string, for?: string, scope?: string }` | Install full source or agent SKILL.md |
| `remove_skill` | `{ name: string, for?: string, scope?: string }` | Remove installed skill |
| `list_categories` | `{}` | Categories with skill counts |
| `get_requirements` | `{ name: string }` | Env vars, system deps, npm deps |
| `run_skill` | `{ name: string, args?: string[] }` | Execute a skill |

Resources: `skills://registry` (full JSON), `skills://{name}` (individual skill + docs + requirements).

## Testing

Tests use `bun:test` with `describe`/`test`/`expect`. Set `NO_COLOR=1` in test environments for deterministic output.

### Test Files

| File | What It Tests |
|------|---------------|
| `src/lib/registry.test.ts` | Registry lookups, search, category filtering |
| `src/lib/installer.test.ts` | Install, remove, agent install, index generation |
| `src/lib/skillinfo.test.ts` | Docs reading, requirements extraction, SKILL.md generation |
| `src/lib/skillinfo-run.test.ts` | Skill execution via `runSkill()` |
| `src/lib/utils.test.ts` | `normalizeSkillName()` |
| `src/lib/validation.test.ts` | Structural validation: every registry entry has a directory, every directory has a registry entry, all have package.json and at least one doc file |
| `src/index.test.ts` | Library re-exports are present |
| `src/cli/cli.test.ts` | CLI integration tests (spawns subprocesses, checks stdout) |
| `src/mcp/mcp.test.ts` | MCP integration tests (in-memory transport via SDK) |
| `src/server/serve.test.ts` | HTTP server and REST API tests |

### Running Tests

```bash
bun test                              # All tests
bun test src/lib/registry.test.ts     # Single file
bun test --timeout 30000              # Increase timeout for slow tests
```

## Adding a New Skill

1. Create `skills/skill-{name}/` with `src/index.ts`, `package.json`, `tsconfig.json`, `SKILL.md`
2. Add entry to the `SKILLS` array in `src/lib/registry.ts` (name, displayName, description, category from `CATEGORIES`, tags)
3. Run `bun test` to verify the registry and structural validation tests pass

## TypeScript

Strict mode. JSX uses `react-jsx` transform for Ink components. Target: ES2022, module: ESNext, moduleResolution: bundler.

---
> Source: [hasna/skills](https://github.com/hasna/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
