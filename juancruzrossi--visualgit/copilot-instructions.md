## visualgit

> - **ALWAYS bump the version in `package.json`** after every change merged to main. Don't wait for the user to ask. Use semver: patch for fixes/refactors, minor for features, major for breaking changes.

# AGENTS.md

## Critical Rules

- **ALWAYS bump the version in `package.json`** after every change merged to main. Don't wait for the user to ask. Use semver: patch for fixes/refactors, minor for features, major for breaking changes.
- **NEVER push directly to `main`** without admin bypass. Use PRs with squash & merge.
- When merging PRs with `gh pr merge`, use `--admin` flag to bypass the protection ruleset.
- Delete branches after merging (`--delete-branch`).
- Responses in Spanish, code comments in English.

## Build & Dev Commands

```bash
# Development (two terminals)
npm run dev              # Frontend: Vite dev server (port 5173, proxies /api → 4321)
npm run dev:server       # Backend: tsx watch (port 4321)

# Build
npm run build            # Both: vite build + tsc -p tsconfig.server.json
npm run build:frontend   # Only frontend → dist/
npm run build:server     # Only backend → dist-server/

# Test
npm test                 # vitest run (all tests)
npm run test:watch       # vitest watch mode

# Run as CLI
node bin/cli.js          # Launch from repo root
npx visualgit            # From any git repo (production)
visualgit update         # Self-update from npm
```

## Architecture

**CLI (`bin/cli.js`)** → spawns **Express server (`dist-server/`)** → serves **React SPA (`dist/`)** → opens browser.

The CLI passes `REPO_PATH`, `PORT`, `IS_GIT_REPO` as env vars to the server process.

### Backend (Express 5 + TypeScript)

```
server/
├── index.ts                    # App factory, static serving, SPA fallback
├── routes/
│   ├── git.ts                  # GET /api/git/{status,info,diff}
│   └── ai.ts                   # POST /api/ai/analyze (SSE streaming)
├── services/
│   ├── git.service.ts          # simple-git wrapper (branch, diff, repo name)
│   └── ai.service.ts           # Spawns claude/openai CLI, pipes stdin/stdout
└── utils/
    └── diff-parser.ts          # Regex-based unified diff parser
```

- AI analysis spawns `claude -p --model sonnet` or `openai` CLI as child process with `cwd: repoPath`
- Prompts are piped via stdin to avoid arg size limits
- Response is streamed as SSE: `data: {text?, done?, error?}`

### Frontend (React 19 + Vite 7 + Tailwind 4)

```
src/
├── App.tsx                     # Split-panel layout (resizable diff + AI panel)
├── hooks/
│   ├── useGitData.ts           # Fetches /api/git/* on mount
│   └── useAiAnalysis.ts        # SSE streaming from /api/ai/analyze
├── components/
│   ├── DiffViewer.tsx          # File tree + diff lines + selection detection
│   ├── AiPanel.tsx             # Markdown rendering (react-markdown) + controls
│   ├── FileHeader.tsx          # Collapse toggle + viewed checkbox
│   ├── DiffLine.tsx            # Single line renderer
│   ├── Header.tsx              # Repo name + branch info
│   └── StatusBar.tsx           # Footer stats
└── lib/
    └── tokens.ts               # Design tokens (colors, fonts)
```

- Inline styles for colors (GitHub dark theme), Tailwind for layout only
- AI analysis is manual (button click), never automatic
- Three analysis modes: `full` (all changes), `file` (single file), `selection` (highlighted text)

### TypeScript Configs

- `tsconfig.json` — Client: ES2022, JSX react-jsx, path alias `@/*` → `src/*`
- `tsconfig.server.json` — Server: NodeNext modules, output `dist-server/`, excludes tests
- `tsconfig.node.json` — Vite config

### Tests (Vitest 4)

- Environment: `node` (no jsdom)
- Location: `server/**/__tests__/*.test.ts`
- Backend only: GitService, AiService, DiffParser
- Mocks: `vi.mock('simple-git')`, `vi.mock('child_process')`
- No frontend component tests

## NPM Distribution

- Package: `@jxtools/visualgit` (public, scoped)
- Published files: `bin/`, `dist/`, `dist-server/`
- `prepublishOnly` runs full build
- GitHub Actions auto-publishes on version bump to main (trusted publisher via OIDC)
- Branch protection: ruleset `protect-main` requires code owner review (@juancruzrossi), admin bypasses

## Design System

- Theme: GitHub dark (`#0D1117` bg, `#161B22` secondary, `#30363D` borders)
- Text: `#E6EDF3` primary, `#9DA5AE` muted
- Accent: `#58A6FF` blue, `#3FB950` green, `#F47067` red
- Font: JetBrains Mono, sizes 11-14px

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juancruzrossi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/juancruzrossi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
