## repohub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RepoHub is a native macOS desktop app (Electron) that serves as a project management dashboard. It scans a local repos directory, auto-detects project types, provides inline terminals for running projects, monitors open ports, tracks Git status, checks dependency health, and integrates with GitHub PRs.

## IMPORTANT

- You may add any shadcn components you need, prefer to use shadcn over creating custom components or workarounds where shadcn versions exist
- UPDATE docs after making changes to keep a strong knowledgebase. This helps both developers and ai agents.
- UPDATE this file when you encounter difficulties or codebase peculiarities so that your future self can avoid them.
- UPDATE this file when changes occur that render anything in here to be outdated.
- DESIGN: Think macOS meets Linear

## Documentation

Detailed docs live in `docs/`:

- `docs/features.md` — All features: dashboard, repo detail view, monorepo, dependency health, GitHub integration, branch cleanup, semantic code search, package intelligence, skills browser, port monitoring
- `docs/configuration.md` — Config options, settings UI, file location, command overrides
- `docs/architecture.md` — Three-process model, IPC flow, async design, PATH fix, security
- `docs/development.md` — Commands, prerequisites, project structure, how to add components/IPC channels

## Commands

```bash
pnpm dev          # Development mode with hot reload
pnpm build        # Compile TypeScript to out/
pnpm dist         # Build + package macOS .app to release/
pnpm install-app  # dist + copy to /Applications
pnpm typecheck    # Type-check only (tsc --noEmit)
```

No test framework or linter is configured. `pnpm run typecheck` is the primary validation command.

## Architecture

### Three-Process Electron Model

```
Main Process (Node.js)          Preload (context bridge)          Renderer (Chromium/React)
  src/main/                       src/preload/index.ts              src/renderer/
  ├── index.ts (entry)                                              ├── views/
  ├── services/                                                     ├── components/
  ├── ipc/                                                          ├── hooks/
  └── types/                                                        └── store/
```

**IPC communication flow:**
React hooks → `window.electron.*` (preload bridge) → IPC handlers (`src/main/ipc/`) → Service layer (`src/main/services/`) → events back via `webContents.send` → preload listeners → Zustand stores → React re-renders.

### Main Process Services (`src/main/services/`)

- **RepositoryService** — Async filesystem scan, project type detection, git info extraction, file watching (chokidar)
- **ProcessService** — PTY process lifecycle via node-pty, output buffering (50ms debounce), emits events
- **PortService** — Polls `lsof` every 5 seconds, links ports to managed processes
- **ConfigService** — Persists to electron-store (`~/Library/Application Support/repohub/config.json`)
- **LogService** — Terminal output to disk per repo (100KB max per file)
- **DependencyHealthService** — Runs npm/pnpm audit + outdated, caches results
- **GitHubService** — Async gh CLI integration for PR status, CI checks, PR creation
- **GitBranchService** — Branch listing (`git branch -vv` + `--merged`) and safe deletion (`git branch -d`)
- **ScaffoldService** — Recipe-based project scaffolding via PTY, template file copying, recipe CRUD
- **CodeSearchService** — Semantic code search: tree-sitter parsing, ONNX embeddings, vectra vector store, file watching
- **CodeParser** — Tree-sitter WASM-based code parsing into semantic chunks (functions, classes, methods)
- **ProjectDetector** — Heuristic detection: Node.js, Python, Rust, Go, Java, Swift, Monorepo
- **PackageIntelligenceService** — npm registry search and package details (HTTP fetch, 1hr cache, no shell commands)
- **WorkspaceDetector** — pnpm workspace parsing for monorepo packages
- **AgentWebSocketServer** — Local WebSocket server for Claude Code CLI SDK communication (random localhost port)
- **AgentService** — Agent lifecycle: spawn `claude` CLI with `--sdk-url`, route NDJSON messages, manage sessions/permissions/cost
- **SkillsService** — Browse and install agent skills from curated GitHub repos via `gh api`, YAML frontmatter parsing with gray-matter, 1hr cache

### Important: All Shell Commands Are Async

All `git`, `gh`, `npm`/`pnpm`, and `lsof` calls use async `exec` (promisified `child_process.exec`), never `execSync`. This prevents blocking Electron's main thread. Git info and GitHub PR fetches run in parallel via `Promise.all`.

### PATH Augmentation

macOS packaged apps get a minimal PATH. `src/main/index.ts` prepends `/opt/homebrew/bin`, `/opt/homebrew/sbin`, `/usr/local/bin`, `/usr/local/sbin` at startup so external tools are found.

### Renderer Architecture (`src/renderer/src/`)

- **State**: Zustand stores in `store/` (repositoryStore, processStore, portStore)
- **IPC wrappers**: Custom hooks in `hooks/` (useRepositories, useProcesses, usePorts, useConfig, useHealth, useGitHub); git branch ops via `window.electron.git.*` directly
- **Routing**: react-router-dom with hash-based routing (9 routes: /, /repo/:id, /github, /search, /agents, /skills, /packages, /ports, /settings)
- **UI**: shadcn/ui components in `components/ui/`, Tailwind CSS v4, Lucide icons
- **Terminal**: xterm.js with fit addon and clickable URL detection
- **Tooltips**: All interactive badges and buttons have tooltips via shadcn/ui Tooltip

### Path Alias

`@/*` maps to `src/renderer/src/*` (configured in tsconfig.json and electron.vite.config.ts).

### Key Identifiers

- **Repository IDs**: MD5 hashes (first 12 chars) of the full filesystem path
- **Process keys**: Can be composite (`repoId:packageName`) for workspace packages

## Build System

electron-vite (Vite-based) with three separate build targets in `electron.vite.config.ts`:

- **main**: Node.js target, externalizes deps except electron-store
- **preload**: Node.js target, externalizes all deps
- **renderer**: Browser target with React and Tailwind plugins

Output goes to `out/main/`, `out/preload/`, `out/renderer/`.

## Key Conventions

- **Package manager**: Always use `pnpm`, never npm/yarn
- **Radix UI**: Import from `"radix-ui"` (unified package), not `@radix-ui/react-*`
- **shadcn/ui components**: Located in `src/renderer/src/components/ui/`
- **No hardcoded colors**: Never use hex color values (e.g., `#82aaff`, `bg-[#292d3e]`) in components. Use Tailwind theme classes (`text-foreground`, `bg-card`, `border-border`) for UI chrome and Tailwind palette colors (`text-blue-400`, `text-green-400`) for accents. Exception: external brand colors (e.g., GitHub language colors) may use inline hex.
- **Prefer theme tokens over palette colors for interactive states**: For selection, hover, focus, and active states use semantic theme tokens (`bg-accent`, `border-primary/40`, `text-primary`) rather than raw palette colors (`bg-blue-950/20`, `border-blue-700/60`, `text-blue-400`). Palette colors are for static category badges and decorative accents only.
- **No layout shift**: Avoid conditionally rendered UI elements (action bars, toolbars, banners) that push page content around. Instead, integrate contextual actions into existing fixed-position areas (e.g., header bars) or use overlays/absolute positioning, or popovers/dialogs. The page layout should be stable regardless of state.

## Security Model

- `contextIsolation: true` — Renderer cannot access Node APIs directly
- `nodeIntegration: false` — All IPC goes through the preload bridge
- `sandbox: false` — Disabled because node-pty requires it; acceptable since no untrusted content is loaded

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/InfinityBowman)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/InfinityBowman)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
