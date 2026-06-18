## nestbrain

> LLM-powered personal knowledge base packaged as a native desktop workspace. Raw sources go in, a structured Markdown wiki comes out — compiled, linked, and maintained entirely by AI. Ships as an Electron app with an embedded Next.js UI, a real PTY terminal, and deep Claude Code integration.

# NestBrain

LLM-powered personal knowledge base packaged as a native desktop workspace. Raw sources go in, a structured Markdown wiki comes out — compiled, linked, and maintained entirely by AI. Ships as an Electron app with an embedded Next.js UI, a real PTY terminal, and deep Claude Code integration.

Published binaries are sold at [nestbrain.app](https://nestbrain.app) ($29 one-time, signed + notarized). The source in this repo is **GPL-3.0** and fully buildable.

## Project Overview

NestBrain ingests raw documents (URLs, PDFs, GitHub repos, arXiv papers, YouTube transcripts, RSS feeds) and uses an LLM to compile them into an interconnected wiki of Markdown files. The wiki is Obsidian-compatible and queryable through hybrid search + LLM-grounded Q&A. The desktop app wraps all of this in a workspace UI: VS Code-style file tree, CodeMirror editor, integrated xterm.js terminal, session-aware Claude Code skills.

## Repo Layout (pnpm monorepo)

```
nestbrain/
├── apps/
│   ├── desktop/                # Electron 33 shell (main + preload + builder config)
│   │   ├── src/main.ts         # Electron main: PATH fix, IPC, embedded server
│   │   ├── src/dev-module.ts   # Open-core seam: guarded require of src/dev-impl/ (gitignored,
│   │   │                       #   lives in the PRIVATE nestbrain-modules repo; CI overlays it,
│   │   │                       #   local dev uses scripts/sync-modules.sh)
│   │   ├── src/preload.ts      # Renderer-safe IPC bridge
│   │   └── build/              # electron-builder hooks, icons, NSIS installer
│   └── web/                    # Next.js 16 + React 19 UI (runs as standalone inside Electron)
│       └── src/{app,components,lib,types}
├── packages/
│   ├── cli/                    # `nestbrain` CLI (commander)
│   ├── core/                   # Domain logic
│   │   └── src/{compiler,ingest,llm,qa,search,lint,vectorstore}
│   ├── db/                     # Chroma client + embeddings wrapper
│   ├── shared/                 # Types and constants (auth/sync types live here so main + renderer share them)
│   └── sync/                   # Drive-backed multi-device sync engine
│       └── src/{engine,drive-adapter,watcher,manifest,excludes,walker,hash}
├── skeleton/                   # Workspace template copied to NestBrain/ on first run
│   ├── CLAUDE.md
│   └── Skills/{start_session,end_session}/SKILL.md
├── docker/                     # Dockerfile + compose (includes a chromadb service)
├── data/                       # Local-dev workspace (raw/, wiki/, chromadb/)
├── docs/screenshots/
├── nestbrain.yaml              # Default workspace config (provider, embeddings, search, server)
├── pnpm-workspace.yaml
├── turbo.json
└── package.json                # version: 1.14.3
```

## Tech Stack

- **Language**: TypeScript 5.9 (Node 20+, ESM where possible, CJS where Electron forces it)
- **Package manager**: pnpm 10 + Turborepo
- **Desktop shell**: Electron 33, `node-pty` (lazy-loaded), `electron-builder` (mac DMG signed/notarized, Windows NSIS, Linux placeholder)
- **UI**: Next.js 16.2 + React 19, CodeMirror 6, xterm.js + addon-fit, mermaid, react-force-graph-2d, lucide-react, Tailwind v4
- **CLI**: `commander`
- **LLM providers** (`packages/core/src/llm/`): `claude-cli` (default — spawns the user's `claude` CLI) and `openai`
- **Embeddings**: `@huggingface/transformers` running ONNX locally (`Xenova/all-MiniLM-L6-v2`)
- **Vector store**: `packages/core/src/vectorstore` is the primary path used by the compiler; `packages/db` wraps `chromadb` for the dockerized setup
- **Ingest**: `@mozilla/readability` + `linkedom` + `turndown` for URLs, `pdf-parse` for PDFs, `rss-parser`, `youtube-transcript`, custom GitHub + arXiv adapters
- **Auth**: Google OAuth 2.0 Desktop flow (PKCE, loopback redirect). Refresh token in OS keychain via Electron `safeStorage`. Code in `apps/desktop/src/auth/`.
- **Sync**: `@nestbrain/sync` package — chokidar watcher + Drive REST adapter + manifest + engine. Wired into the main process by `apps/desktop/src/sync/`. Scope: `drive.file` (no security audit, app only sees what it created).
- **Lint/QA**: LLM-driven against the compiled wiki
- **Testing**: Vitest (configured at root, very thin coverage today)
- **Lint/format**: ESLint 9 + Prettier 3

## Key Commands

### Repo-level (pnpm + turbo)

```bash
pnpm install                       # bootstrap workspace
pnpm dev                           # turbo dev across all packages
pnpm build                         # turbo build
pnpm lint                          # turbo lint
pnpm test                          # turbo test (vitest)
pnpm format                        # prettier write
```

### Desktop app

```bash
pnpm desktop:dev                   # build TS + launch Electron with NESTBRAIN_DEV=1
pnpm desktop:build                 # build web standalone + copy assets + build desktop TS
pnpm desktop:package:mac           # DMG into apps/desktop/release/
pnpm desktop:package:win           # NSIS .exe into apps/desktop/release/
```

`desktop:build` runs `apps/desktop/build/copy-assets.mjs` and `prepare-standalone.mjs` — these dereference pnpm symlinks and promote hoisted deps so the packaged app works on Windows.

### CLI (after `pnpm build`)

```bash
nestbrain ingest <source>          # URL / file / GitHub / arXiv / YouTube / RSS
nestbrain compile [--force]        # Incremental wiki compile from data/raw → data/wiki
nestbrain ask "question"           # Hybrid-search-grounded Q&A
nestbrain search "query"           # Hybrid semantic + keyword
nestbrain lint                     # LLM health check
nestbrain serve                    # Start the web UI standalone
```

## Coding Conventions

- TypeScript everywhere. Type all exported functions; rely on inference inside function bodies.
- ESM in `packages/*` and `apps/web`; the Electron main bundle ends up CJS (so dynamic `require` is fine when needed, e.g. lazy `node-pty`).
- Use `node:fs/promises` + `node:path` for filesystem work. Use `node:path.join` with the platform separator — don't hand-build paths with `/`.
- One responsibility per file. The `packages/core/src/<area>/index.ts` files are the public surface; siblings are internals.
- Errors propagate inside `packages/core`. CLI commands, Next.js route handlers, and the Electron IPC layer are the boundaries that turn errors into user-facing messages.
- All wiki output is valid Markdown with YAML frontmatter and `[[wikilinks]]` for internal references (Obsidian compatibility).
- Prefer direct implementations over abstractions. No premature interfaces.
- Default to no comments. Add one only when the *why* is non-obvious — a real example is the `node-pty` lazy-load block in `apps/desktop/src/main.ts`.
- Do not log to stdout from `packages/core` — pass a `ProgressCallback` (see `CompileOptions`) so the caller decides how to surface progress.

## Wiki File Format

Every generated wiki article must follow this structure:

```markdown
---
title: "Article Title"
created: 2026-01-15
updated: 2026-01-15
source: "raw/filename.md"        # if derived from a source
tags: [concept, topic]
backlinks: ["[[Other Article]]"]
---

# Article Title

Body…

## See Also

- [[Related Article]]
- [[Another Article]]
```

## LLM Agent Guidelines

- The wiki is the LLM's domain. Generate and maintain wiki content programmatically — never hand-edit files in a user's `Library/Knowledge/`.
- Always be incremental: the compiler's `tracker.ts` records source hashes; honour it. Don't reprocess unchanged sources.
- Maintain `_index.md` and `_concepts.md` as the navigational backbone.
- When answering questions, cite wiki articles by `[[wikilink]]` to the actual file the answer used.
- When linting, produce actionable suggestions with concrete file references.
- Minimize token usage: read indexes and summaries first; dive into full articles only when needed.
- Never delete user-provided raw data. Wiki content is regenerable.

## Important Notes

- `data/raw/` (or, in user workspaces, `<NestBrain>/.nestbrain/raw/`) is user data — never modify or delete it.
- `data/wiki/` (and the user's `Library/Knowledge/`) is fully regenerable from raw — treat it as a build artifact, but warn before wiping if the user has hand-edited anything.
- LLM credentials come from the user's `claude` CLI auth (default), the OpenAI key in Settings, or `nestbrain.yaml`. Never hardcode keys, never log them.
- All paths inside the wiki must be relative for portability between machines.
- The Electron main on macOS does **not** inherit the user's shell PATH — `apps/desktop/src/main.ts` runs an inline `fix-path` equivalent so that spawning `claude` works regardless of where it's installed. Don't remove it.
- `node-pty` is loaded with a `try/catch require` because a native-binding load failure must not crash the app — the terminal is optional.
- **Modules (open-core)**: Enterprise add-ons (Dev = terminal/git/Projects; Anatomize planned) are gated twice — build-time (implementation only in the private `nestbrain-modules` repo, overlaid into `apps/desktop/src/dev-impl/` by CI or `scripts/sync-modules.sh`) and license-time (`module:<id>` in the org license features, served by the Team Server, decoded in `apps/desktop/src/modules.ts`). The renderer reads entitlement via `useModules()` (`apps/web/src/lib/modules-context.tsx`). Public source builds compile green without the overlay and run as the pure knowledge core.
- `packages/sync` uses **chokidar 3** (CJS) on purpose — chokidar 4/5 are ESM-only and won't load from our CJS main bundle.
- The Google OAuth Client ID + non-confidential Desktop client secret live in `packages/shared/src/constants.ts`. For OAuth client type "Desktop app" Google considers the secret non-confidential (PKCE is what actually secures the flow), so it's safe to embed in the distributed binary.
- Sync details (architecture, exclude rules, conflict + delete semantics, the `drive.file` trade-off, known limits) are documented in [`docs/SYNC.md`](docs/SYNC.md). Read it before touching `packages/sync` or `apps/desktop/src/sync`.

---
> Source: [mikegazzaruso/nestbrain](https://github.com/mikegazzaruso/nestbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
