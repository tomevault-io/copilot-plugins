## dsh-subagent-registry

> Claude-style `.agent.md` subagent registry plugin for DeepSeek Harness.

# dsh-subagent-registry — Module Guide

Claude-style `.agent.md` subagent registry plugin for DeepSeek Harness.

## Quick Start

```bash
# Install dependencies (npm, not pnpm — avoids internal package resolution issues)
npm install

# Build the plugin (bundles to lib/index.js + deploys to DSH profile)
node build.mjs

# Type-check without emitting
npx tsc --noEmit

# Run tests
node --import tsx --test src/parser.test.ts src/scanner.test.ts
```

## Project Structure

```
dsh-subagent-registry/
├── src/
│   ├── index.ts          Plugin entry: apply(), HMR singleton, tool registration
│   ├── config.ts         Schemastery config schema (exported as Config)
│   ├── types.ts          AgentDef, ParseResult, AgentScope type definitions
│   ├── parser.ts         .agent.md parser: YAML frontmatter + Markdown body
│   ├── scanner.ts        AgentScanner: directory discovery, dedup, file watching
│   ├── route-resolver.ts Model resolution chain: per-call > frontmatter > default > inherit
│   ├── dts-shim.d.ts     Type declarations for runtime-only @deepseek-ai/* packages
│   ├── parser.test.ts    Parser tests (40+ cases)
│   └── scanner.test.ts   Scanner tests (13+ cases)
├── examples/              Example .agent.md files
│   ├── code-reviewer.agent.md
│   ├── researcher.agent.md
│   └── translator.agent.md
├── build.mjs              esbuild bundler + pnpm junction fix + deploy script
├── tsconfig.json          TypeScript config (strict, noEmit, bundler resolution)
├── package.json           npm package manifest
├── cordis.patch.yml       DSH bundle patch (plugin config defaults)
└── README.md             User documentation
```

## Key Design Decisions

### 1. HMR Safety via globalThis Singleton

The plugin uses a `globalThis.__dshSubagentRegistry` singleton pattern. On HMR reload, the previous instance is disposed before the new one is created. This prevents duplicate tool registrations and zombie watchers.

### 2. Claude Code Spec Alignment

The parser supports all Claude Code subagent frontmatter fields:
- Required: `name`, `description`
- Tool filtering: `tools` (whitelist), `disallowedTools` (denylist, overrides `tools`)
- Model routing: `model` (alias or full ID)
- Claude spec: `skills`, `mcpServers`, `hooks`, `memory`, `isolation`
- Advisory fields: `maxTurns`, `effort` (parsed and stored, but DSH API does not support injection yet)

### 3. Scan Path Interleaving

Three directory names (`.dsh/subagents`, `.claude/agents`, `.agents/subagents`) are interleaved at each directory level (project walk-up → user → custom). This means proximity to cwd determines priority, not which directory name is used. When two files define the same agent `name`, the higher-priority one wins.

### 4. Model Resolution Chain

```
per-call args.model  →  frontmatter def.model  →  config defaultModel  →  inherit
```

Aliases (`sonnet`, `haiku`, `opus`, `inherit`) are resolved through `config.modelMap`. The default map is empty — deployments must configure it.

### 5. Advisory Fields

`maxTurns` and `effort` are parsed and stored on `AgentDef`, but DSH's `SubagentStartRequest` and `AgentOptions` APIs do not currently support these fields. They are logged for visibility and may be supported in future DSH versions. Same for `hooks` (no hook injection API) and `isolation: worktree` (no worktree API).

### 6. Build System

- **esbuild** bundles `src/index.ts` → `lib/index.js` (single file, ESM, externalizes `@deepseek-ai/*`, `chokidar`, `yaml`)
- **deploy** copies `lib/`, `package.json`, `cordis.patch.yml` to `~/.dsh/profiles/web/node_modules/dsh-subagent-registry/`
- The pnpm junction fix in `build.mjs` handles Windows pnpm's broken symlink behavior (multi-strategy removal)

### 7. Config Validation

Schemastery schema (`src/config.ts`) validates plugin config at load time. All fields are optional with documented defaults. The schema is exported as `Config` from the plugin entry — Cordis calls `Config(rawConfig)` before passing the validated config to `apply()`.

## Testing

Tests use Node.js built-in test runner (`node:test`) with `node:assert/strict`. TypeScript execution via `tsx`.

```bash
# Run all tests
node --import tsx --test src/parser.test.ts src/scanner.test.ts

# Or via npm script
npm test
```

Parser tests cover: frontmatter extraction, required-field validation, YAML edge cases, optional-field normalization, Claude spec fields, and a full integration test with all fields.

Scanner tests cover: basic discovery, recursive scanning, duplicate-name resolution, error collection, helper methods, scan directory configuration, and scope assignment. Scanner tests use temporary directories (`os.tmpdir()`) so no real project/user paths are touched.

## Adding a New Agent

Create a `.agent.md` file in one of the scan directories:

```markdown
---
name: my-agent
description: When to use this agent
tools: Read, Glob, Grep
model: sonnet
---

You are a specialized agent. Your responsibilities are...
```

Place it in `.dsh/subagents/`, `.claude/agents/`, or `.agents/subagents/` at the project or user level. The scanner will discover it on the next scan (or immediately if file watching is enabled).

---
> Source: [john-walks-slow/dsh-subagent-registry](https://github.com/john-walks-slow/dsh-subagent-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
