## ccode

> Persistent context CLI for AI-assisted development. Generates documentation, architecture, rules, and a verifiable task checklist — synced to every major AI tool.

# CCODE

Persistent context CLI for AI-assisted development. Generates documentation, architecture, rules, and a verifiable task checklist — synced to every major AI tool.

## Architecture

```
src/
├── cli/            Session loop, branding, file watcher
│   ├── index.ts    Main CLI entry point, commands, handlers
│   ├── brand.ts    Colors, logo, UI components
│   └── watcher.ts  FileWatcher (fs.watch, debounce)
├── core/           Business logic
│   ├── context.ts  ContextEngine — persistent state in .ccode/
│   ├── tasks.ts    TaskEngine — task CRUD, priorities, stats
│   ├── exports.ts  ContextExporter — universal context sync
│   └── prompt-builder.ts  Meta-prompts for context generation
├── ai/             Provider adapters (Adapter pattern)
│   ├── provider.ts IAIProvider interface
│   ├── manager.ts  AIManager — auto-detection, config, factory
│   ├── claude.ts   Claude (Anthropic) adapter
│   └── gemini.ts   Gemini adapter (API key + OAuth)
└── utils/
    └── files.ts    File system abstraction (fs-extra)
```

**Patterns:** Adapter (AI providers) · Observer (file watcher) · State Machine (workflow) · Builder (prompts)

## Key Principle

CCODE generates context, never code. Tasks describe WHAT to achieve, not HOW to implement it.

## Development Guidelines

- TypeScript with ESM (`"type": "module"`, NodeNext resolution)
- All dependencies are ESM-only (chalk v5, inquirer v13, ora v8)
- Tests with Vitest
- Conventional commits (`feat:`, `fix:`, `docs:`, `chore:`)
- AI providers implement `IAIProvider` interface (generate + getName)

## Context Sync

CCODE generates context files for multiple AI tools from `.ccode/` source:

| File | Tool |
|------|------|
| `AGENTS.md` | Open Standard |
| `CLAUDE.md` | Claude Code |
| `GEMINI.md` | Gemini CLI |
| `.cursorrules` | Cursor |
| `.github/copilot-instructions.md` | GitHub Copilot |

All generated from the same source of truth in `.ccode/`.

## Commands

| Command | Handler |
|---------|---------|
| `ccode init` | `handleInit()` — wizard + context generation + sync |
| `ccode sync` | `handleSync()` — regenerate all AI tool context files |
| `ccode update` | `handleUpdate()` — re-analyze project with AI |
| `ccode export` | `handleExport()` — export universal or per-tool |
| `ccode verify` | `handleVerify()` — AI-powered task verification |
| `ccode doctor` | `handleDoctor()` — health check |
| `ccode status` | `handleStatus()` — dashboard |
| `ccode connect` | `handleConnect()` — AI provider setup |

## Roles

### Architect
Structure, patterns, scalability. Validates `.ccode/architecture.md`.

### Developer
Clean TypeScript, error handling, ESM compliance. Works in `src/`.

### Tester
Edge cases, input validation, integration. Tests in `tests/`.

### Project Manager
Progress tracking, prioritization. Manages `tasks.json`, `state.json`.

---
> Source: [iDevelop25/ccode](https://github.com/iDevelop25/ccode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
