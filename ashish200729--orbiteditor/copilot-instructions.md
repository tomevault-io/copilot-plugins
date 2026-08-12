## orbiteditor

> Guidelines for contributors and AI coding agents working in this repository.

# Repository Guidelines

Guidelines for contributors and AI coding agents working in this repository.

## Project Structure

```
orbiteditor/
├── src/vs/workbench/contrib/orbit/   # All Orbit-specific code
│   ├── common/                        # Types, settings, prompts, registries
│   ├── browser/                       # Workbench services, tools, React host
│   │   └── react/                     # React UI (separate build pipeline)
│   ├── electron-main/                 # LLM transport, OAuth, MCP, updates
│   └── test/                          # Orbit unit tests
├── extensions/                        # Built-in VS Code extensions
├── build/                             # Gulp tasks and CI configs
├── scripts/                           # Dev launcher, release, install
├── docs/                              # Public documentation (see docs/README.md)
├── product.json                       # App identity, version, API URLs
└── test/                              # Upstream VS Code test runners
```

Generated outputs — never edit: `out/`, `.build/`.

## Build Commands

| Command | When to use |
|---------|-------------|
| `npm install` | First-time setup |
| `npm run watch` | Day-to-day development |
| `npm run compile-client` | Fastest compile check |
| `npm run compile` | Full compile (slow) |
| `npm run buildreact` | **Required** after any React source change |
| `./scripts/code.sh` | Launch dev Orbit window (macOS/Linux) |
| `npm run test-node` | Run Orbit + VS Code unit tests |
| `npm run eslint` / `npm run stylelint` | Lint touched files |

**Build order after React changes:** `npm run buildreact` → `compile` or `watch`.

Node version: **20.18.2** (`.nvmrc`). Run `nvm use`.

## React UI Pipeline

Location: `src/vs/workbench/contrib/orbit/browser/react/`

```
src/  →  src2/  (scope-tailwind)  →  out/  (tsup)
```

Run `node build.js` or `npm run buildreact` from repo root.

**Critical rules:**
- All external imports must end with `.js` (e.g. `../../../../../file.js`)
- `src/` must stay **one folder deep** for tsup externals detection

See [browser/react/README.md](./src/vs/workbench/contrib/orbit/browser/react/README.md).

## Where Orbit Code Lives

| Concern | Path |
|---------|------|
| Chat modes & tool policies | `common/prompt/prompts.ts` |
| Provider settings | `common/orbitSettingsTypes.ts`, `common/modelCapabilities.ts` |
| Chat threads & checkpoints | `browser/chatThreadService.ts` |
| Tool execution | `browser/toolsService.ts` |
| LLM provider implementations | `electron-main/llmMessage/sendLLMMessage.impl.ts` |
| Sidebar React UI | `browser/react/src/sidebar-tsx/` |
| Subagents | `common/subAgentRegistry.ts`, `browser/subAgentOrchestratorService.ts` |
| Skills | `common/skillRegistry.ts`, `browser/skillLoader.ts` |
| MCP | `common/mcpService.ts`, `electron-main/mcpChannel.ts` |

Most Orbit-specific code lives in `src/vs/workbench/contrib/orbit/`. See [ORBIT_CODEBASE_GUIDE.md](./ORBIT_CODEBASE_GUIDE.md) for architecture details.

## Styling

- Use theme tokens from `browser/react/src/styles.css` (`--void-*`, `--vscode-*`)
- Avoid hardcoded colors
- Match existing Tailwind patterns (`void-` prefix)
- Preserve responsive behavior and reduced-motion support

## Testing

- Orbit tests: `src/vs/workbench/contrib/orbit/test/`
- Run: `npm run test-node`
- Match existing test naming and structure

## Documentation

- [readme.md](./readme.md) — project overview
- [ORBIT_CODEBASE_GUIDE.md](./ORBIT_CODEBASE_GUIDE.md) — architecture and codebase walkthrough
- [HOW_TO_CONTRIBUTE.md](./HOW_TO_CONTRIBUTE.md) — developer setup and build
- [CONTRIBUTING.md](./CONTRIBUTING.md) — contribution index
- [docs/](./docs/) — feature and release docs

## Commit & PR Guidelines

- Short, imperative commit messages
- Do not use AI to write PR descriptions
- Protected branches: `main`, `distro`, `release/*`
- Run targeted tests/lints for touched areas

## Gotchas

- `cli/` is a Rust CLI — separate from the TypeScript build
- `product.json` defines app identity (`orbitVersion`, API URLs)
- Watch daemon: `deemon npm run watch` / `deemon --kill npm run watch`
- Data folder: `.orbit-editor` (VS Code heritage); user config uses `~/.orbit/` for skills/agents
- Windows builds need VS 2022 build tools (see HOW_TO_CONTRIBUTE.md)

---
> Source: [ashish200729/orbiteditor](https://github.com/ashish200729/orbiteditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
