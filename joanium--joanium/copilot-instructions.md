## joanium

> Joanium is a **local-first AI desktop assistant** built on Electron (Node.js ≥ 24).

# Joanium — Agent Context

Joanium is a **local-first AI desktop assistant** built on Electron (Node.js ≥ 24).
It supports multi-model chat, automations, scheduled agents, MCP, personas, skills,
and real integrations with Anthropic, OpenAI, Gemini, Ollama, Gmail, GitHub, and more.

---

## Project Layout

```
App.js                  Electron entry point
Core/Electron/          Main process (IPC, window management, lifecycle)
Packages/
  Capabilities/         Low-level capability modules (tools, integrations)
  Features/             High-level feature modules (chat, agents, automations)
  Main/                 Main-process packages
  Modals/               Modal UI components
  Pages/                Renderer pages
  Renderer/             Renderer-side shared utilities
  System/               System-level helpers
Config/                 User-facing configuration files
Data/                   Persistent local data (SQLite, JSON stores)
Skills/                 Packaged skill definitions
Personas/               Persona definitions and prompts
SystemInstructions/     System prompt templates
Instructions/           Runtime instruction sets
Memories/               Memory store for agents
Assets/                 Icons, images, static assets
Scripts/                Build, version, audit scripts
Docs/                   Developer documentation
dist/                   Build output (not committed)
```

## Path Aliases (package.json `imports`)

| Alias             | Resolves to                 |
| ----------------- | --------------------------- |
| `#capabilities/*` | `./Packages/Capabilities/*` |
| `#core/*`         | `./Core/*`                  |
| `#features/*`     | `./Packages/Features/*`     |
| `#main/*`         | `./Packages/Main/*`         |
| `#modals/*`       | `./Packages/Modals/*`       |
| `#pages/*`        | `./Packages/Pages/*`        |
| `#renderer/*`     | `./Packages/Renderer/*`     |
| `#system/*`       | `./Packages/System/*`       |

---

## Key Conventions

- **Pure ESM** (`"type": "module"` in package.json). Always use `import`/`export`.
- **No TypeScript** — vanilla JS throughout.
- **Node.js ≥ 24** — modern APIs (native `fetch`, top-level await, etc.) are available.
- **npm workspaces** — run `npm install` from the root; never `cd` into packages to install.
- Electron main process lives in `Core/Electron/`; renderer code lives under `Packages/`.
- IPC channels are defined in the capability/feature that owns them.
- All user data is stored locally (local-first); no telemetry or remote sync by default.

## Common Commands

```bash
npm start              # Launch in production mode
npm run dev            # Launch with --dev flag (DevTools enabled)
npm run build          # Full production build via electron-builder
npm run format         # Prettier format all files
npm run lint           # ESLint check
npm run packages:audit # Audit workspace package dependencies
```

## Coding Guidelines for Agents

1. **Respect the alias structure** — import from `#capabilities/…` not relative paths across package boundaries.
2. **IPC is the bridge** — main ↔ renderer communication must go through IPC; no direct `require` of main-process modules in the renderer.
3. **Minimal surface** — each Capability/Feature should export only what it declares in its own `package.json` `exports` field.
4. **No bundler** — the project uses raw Electron loading, not webpack/vite. Keep files loadable as-is.
5. **Formatting** — run `npm run format` before committing. Husky + lint-staged enforces this on commit.
6. **Tests** — Jest is the test runner. Test files live alongside the code they test (`*.test.js`).

---
> Source: [Joanium/Joanium](https://github.com/Joanium/Joanium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
