## harness-visualizer

> > **This is the primary source of truth for working on the Harness Visualizer.**

# AGENTS.md

> **This is the primary source of truth for working on the Harness Visualizer.**
> Any agent — Claude Code, Cursor, Copilot, Aider, Windsurf, Codex, anything — should read this file in full before reading any other file in the repo, and before making any edits. Tool-specific configs (`CLAUDE.md`, `.cursor/rules/`, etc.) point back to this file by design.

---

## What this project is

A tool-agnostic, real-time **harness audit and visualization** demo app. Built as a conference-talk demo for harness engineering and dogfooding.

The app:

1. Watches a user-selected directory tree.
2. Finds harness artifacts (context files, skills, MCP configs, hooks, subagents) using a configurable pattern map.
3. Renders them as an interactive graph in the browser, color-coded by layer.
4. Lets the user click any markdown node to view/edit it in-browser; saves write back to disk; the file watcher closes the loop and the graph re-renders live.

The "wow moment" for the talk is **point the tool at its own repo, edit a skill file in the browser, watch the graph update in real time.** Everything else serves that moment.

Long-form context lives in [`harness-visualizer-plan.md`](./harness-visualizer-plan.md) (the deep-research seed doc) and [`ROADMAP.md`](./ROADMAP.md) (the phase-by-phase build plan). Read both before designing or implementing anything substantial.

---

## Stack

### Backend — Node.js + TypeScript

| Package | Role |
|---|---|
| `express` | HTTP server, REST endpoints (`/api/browse`, `/api/file`, `/api/patterns`) |
| `socket.io` | Real-time push to frontend (`harness:update`, `configure-watch`, `patterns:update`) |
| `chokidar` | File watching, supports `.add()`/`.unwatch()` for live reconfiguration |
| `fast-glob` | Initial directory scan (faster than chokidar's initial pass on large trees) |
| `gray-matter` | YAML frontmatter parser for skill files, rules, AGENTS.md, etc. |
| `zod` | Schema validation for artifacts, IPC payloads, and request bodies (re-exported via `@harness-visualizer/shared`) |
| `vitest` | Test runner |

### Frontend — Vue 3 + Vite + TypeScript

| Package | Role |
|---|---|
| `pinia` | State management — Pinia setup-stores hold `harness:update` payload state |
| `socket.io-client` | WebSocket client (paired with reactive Pinia state); typed `Socket<S2C, C2S>` (note flipped order vs server) |
| `v-network-graph` | SVG graph rendering, drag/pan/zoom, `node:click` events. Consumes `Record<string, T>` (id-keyed), NOT arrays. Phase 3 ships a `computed()` adapter from `Node[]/Edge[]` |
| `d3-force` | Peer-dep of `v-network-graph` (force-directed layout via `v-network-graph/lib/force-layout`); install explicitly |
| `@vueuse/core` | Composables — `useDark` for theme toggle |
| `tailwindcss` v4 | Utility-first styling, CSS-first config (`@import "tailwindcss"` + `@theme` + `@custom-variant dark`); paired with `@tailwindcss/vite` plugin |
| `md-editor-v3` | Markdown viewer + editor (`MdEditor`, `MdPreview`, `MdCatalog`) — Phase 5 |
| `vitest` + `@vue/test-utils` | Component + unit tests; mock `socket.io-client` and stub `v-network-graph` in `frontend/src/test/setup.ts` |

### Tooling

- npm **workspaces** (root `package.json` lists `backend`, `frontend`, `shared`).
- TypeScript shared base config at `tsconfig.base.json`, extended per-package.
- ESLint flat config + Prettier at the repo root.
- `concurrently` for the root `dev` script that boots both servers.
- **pnpm migration trigger**: npm workspaces is sufficient for ≤3 packages without hoisting conflicts. Revisit pnpm if a fourth workspace lands or hoisting conflicts emerge.

---

## Repo layout

```
harness-visualizer/
├── AGENTS.md                            ← you are here (source of truth)
├── CLAUDE.md                            ← thin pointer to AGENTS.md
├── ROADMAP.md                           ← 8-phase build plan; one spec per phase
├── harness-visualizer-plan.md           ← deep-research seed doc
├── package.json                         ← workspaces, root scripts
├── tsconfig.base.json
├── eslint.config.js                     ← flat config
├── .prettierrc
│
├── backend/
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── server.ts                    ← Express + Socket.IO entrypoint
│       ├── patterns/                    ← default pattern map + loader
│       ├── scanner/                     ← fast-glob + gray-matter + zod
│       ├── watcher/                     ← chokidar wrapper + hand-rolled debounce
│       ├── scoring/                     ← per-layer presence/validity scores
│       ├── security/                    ← validatePath helper (Vite-style isParentDirectory)
│       └── __test-helpers/              ← withTmpRoot for integration tests
│
├── shared/                              ← @harness-visualizer/shared workspace
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── schemas.ts                   ← zod runtime schemas (PatternMap, Diagnostic, HarnessUpdatePayload, etc.)
│       ├── types.ts                     ← z.infer-derived TS types
│       ├── events.ts                    ← Socket.IO ServerToClientEvents / ClientToServerEvents
│       └── index.ts                     ← named re-exports
│
├── frontend/
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── src/
│       ├── main.ts
│       ├── App.vue
│       ├── socket.ts                    ← typed socket.io-client singleton
│       ├── stores/harness.ts            ← Pinia store
│       └── components/
│           ├── HarnessGraph.vue
│           ├── DirectoryPicker.vue
│           ├── MarkdownEditor.vue
│           ├── PatternEditor.vue
│           ├── ScoreCard.vue
│           └── LayerLegend.vue
│
├── specs/                               ← generated by /spec:* pipeline, one dir per phase
│
└── .claude/
    ├── settings.json
    ├── implementation-team.config.json
    ├── plugins/                         ← spec-gen, claude-build (vendored)
    └── skills/                          ← playwright-skill, implementation-skill
```

---

## Harness layer taxonomy (the data model)

The whole app exists to surface these five layers. Treat them as the canonical vocabulary (singular keys, v2 — Phase 8 dropped the v1 `spec` layer):

| Layer | Artifact examples | Engineering concern |
|---|---|---|
| **context** | `CLAUDE.md`, `AGENTS.md`, `AGENTS.override.md`, `GEMINI.md`, `SYSTEM.md`, `.cursor/rules/`, `.github/copilot-instructions.md`, `.continue/rules/`, `.clinerules/` | What the agent always knows |
| **skill** | `.claude/skills/<name>/SKILL.md`, `.pi/agent/skills/<name>/SKILL.md`, `.pi/skills/<name>/SKILL.md`, `.agents/skills/` | On-demand capability injection |
| **mcp** | `.mcp.json`, `.cursor/mcp.json`, `.gemini/settings.json`, `.continue/config.yaml` | Action space extension |
| **hook** | `.claude/settings.json` hook entries, `.claude/settings.local.json` | Harness execution clamps |
| **subagent** | `.claude/agents/*.md` | Delegated reasoning scope |

The default pattern map (`backend/src/patterns/default-template.json`) uses the v2 shape: `Record<Layer, PatternEntry[]>` where each `PatternEntry` carries `{ tool, scope, globs }`. Users can layer overrides via `.harness-visualizer/patterns.json` in the watched project — v1-keyed/v1-shape override files (`{skills: {globs}}`, etc.) are auto-migrated in-memory on load with a `pattern-migrated` info diagnostic; the patterns API also auto-migrates v1-shape POST bodies and persists the v2 form to disk.

**Layer iteration order**: `skill > subagent > hook > mcp > context`. As of Phase 9 (v2), this is **iteration** order for stable test output — NOT first-match-wins. v2's scanner emits one `HarnessEntry` per (file, layer, pattern-entry) match, so a single file matching multiple layer globs produces multiple entries (e.g., `.claude/settings.json` surfaces as both a `hook` entry and an `mcp` entry, with distinct `id`s derived from `sha256(scope::path::layer)[:8]`). The `multi-layer-match` diagnostic code remains in `DIAGNOSTIC_CODES` but is no longer emitted.

---

## Diagnostic codes

The scanner and pattern loader emit `Diagnostic` records (defined in `@harness-visualizer/shared`). Phase 7's UI maps codes to icons, so this set is stable:

| `code` | Severity | Emitted when |
|---|---|---|
| `invalid-frontmatter` | error | A markdown file's frontmatter (or a JSON artifact) failed zod validation against its layer schema. |
| `malformed-yaml` | error | gray-matter threw while parsing frontmatter. |
| `multi-layer-match` | warning | A file matched multiple layer globs; assigned to the highest-priority one, others discarded. |
| `symlink-skipped` | info | Encountered a symlink during the scan; not followed (per `followSymbolicLinks: false`). |
| `invalid-pattern-file` | error | `<watchRoot>/.harness-visualizer/patterns.json` was unreadable or unparseable; defaults retained. |
| `unknown-event` | warning | A Socket.IO client emitted an event name not in `ClientToServerEvents`. |
| `pattern-migrated` | info | A v1-keyed user pattern override was migrated in-memory to v2 keys on load. |
| `migration-lossy` | warning | A v1 layer was dropped or renamed during migration (e.g., `specs` dropped, `agents` → `subagent`), or an unknown key was stripped. |
| `invalid-id-override` | warning | A frontmatter `id:` field failed length/charset validation; derived id used. (Phase 9.) |
| `id-collision` | warning | Two harness entries derived the same `id`; second falls back to a `:N` suffix. (Phase 9.) |

---

## Security rules (non-negotiable)

This is a **local-only dev tool**. The threat model is "local user, possibly malicious project on disk." All of the following must hold; missing any one is a release blocker:

1. **Bind to `127.0.0.1` only** — never `0.0.0.0`. Enforced from Phase 1.
2. **Path safety**: every read/write goes through `path.resolve` → `fs.realpath` → watch-root allowlist check. Symlinks that resolve outside a watch root are rejected. User-supplied paths are NFC-normalized (`p.normalize('NFC')`) before resolution and before any extension-allowlist comparison — defends against macOS APFS Unicode-form allowlist bypass.
3. **Extension allowlist** for editable files: `.md`, `.markdown`, `.json`, `.yml`, `.yaml`, `.txt`. Anything else returns 403. Compare against the NFC-normalized lowercased extension.
4. **Size cap**: 1 MB read/write hard cap. `express.json({ limit: '1mb' })` at the app level so a missing per-handler check can't bypass.
5. **Schema validation**: every REST request body and Socket.IO payload is zod-validated on both ends. All zod object schemas use `.strict()` so unknown keys (including `__proto__`, `constructor`, `prototype`) are rejected.
6. **No shell-outs** to user-controlled paths. Filesystem APIs only.
7. **Host-header allowlist (DNS rebinding defense)**: every HTTP handler and every Socket.IO `engine.use` middleware rejects requests whose `Host` header isn't in `{ '127.0.0.1:<port>', 'localhost:<port>' }`. Reject with 400. Binding alone doesn't stop a browser-resident page that rebinds DNS to `127.0.0.1` — this rule does.
8. **Socket.IO connection gating**: `io.engine.use` enforces the same Host/Origin allowlist as HTTP. Zod validates payload shape; it doesn't gate the upgrade. A failing-on-purpose test asserts: WebSocket upgrade with a foreign `Origin` is rejected.
9. **Content Security Policy**: served `index.html` carries a strict CSP header — `default-src 'none'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self' ws://127.0.0.1:* http://127.0.0.1:*; object-src 'none'; frame-ancestors 'none'; base-uri 'self'`. No `unsafe-eval`, no inline `<script>`. The Vue + Vite build is CSP-compatible by default; preserving this is a non-goal nothing should regress.
10. **Untrusted markdown rendering**: all rendered markdown is parsed with raw HTML disabled (`md-editor-v3` / `markdown-it` `html: false`) AND passed through DOMPurify with `FORBID_TAGS: ['style','iframe','object','embed','script','link','meta']`, `FORBID_ATTR: ['style','on*','srcdoc']`, no SVG profile, and a `ALLOWED_URI_REGEXP` excluding `javascript:` and `data:` (except for images). `v-html` is allowed ONLY when bound to a DOMPurify-sanitized string; never to raw markdown or to API response fields.
11. **Error / log hygiene**: HTTP and Socket.IO error responses MUST NOT include `err.message`, `err.stack`, or absolute paths. Responses use the diagnostic-code vocabulary in `DIAGNOSTIC_CODES`; long-form details stay server-side. Logs MUST: never include file contents; strip absolute paths to project-relative form (`path.relative(watchRoot, abs)`); never log full payloads (zod errors → field names + types, not values).

Every security branch must have a failing-on-purpose unit test (rejected traversal, rejected symlink, rejected oversize, rejected extension, rejected foreign Host, rejected foreign Origin). See Phase 5 in `ROADMAP.md`.

### Hardening rules

These complement the non-negotiable list. They're not classified as release blockers individually, but each closes a real attack surface the threat model permits.

- **Prototype-pollution discipline**: code MUST NOT `Object.assign(target, untrusted)`, deep-merge untrusted into config, or spread `{ ...payload }` into reactive state — copy named fields explicitly. Use `Object.create(null)` for any map keyed by user-controlled strings. (Rule 5's `.strict()` is the schema-side companion.)
- **YAML safe loading**: parse `.yaml`/`.yml` with `js-yaml`'s `FAILSAFE_SCHEMA` (no custom types, no `!!js/function`). Default `yaml.load` is forbidden on user content.
- **Resource caps**: explicit constants — `MAX_WATCH_ROOTS=8`, `MAX_FILES_PER_SCAN=20_000`, `MAX_SCAN_DEPTH=12`. Hitting a cap emits a structured diagnostic (e.g., `scan-capped`), never a 500. New cap codes go in `DIAGNOSTIC_CODES`.
- **Glob safety**: user-supplied glob patterns in the pattern editor are validated against `safe-regex` (or restricted to a known-safe grammar) to prevent ReDoS via `picomatch`/`micromatch` regex compilation.

---

## Conventions

### Generated convention docs (read these for per-workspace specifics)

Auto-generated by the `progressive-context` plugin from real evidence in the code. Each subtree pairs `patterns/` (HOW — reusable idioms) with `features/` (WHAT — domain capabilities).

**These docs are observational, not normative.** They describe what the code does today, with `path:line` citations — they do not establish rules and do not override the rules in this file. If you observe a conflict between the code and a rule in `AGENTS.md`, surface it for human review (raise it in chat, or add a `> ⚠️ Conflict with AGENTS.md` callout in the generated doc). Do not silently document an observation as a new convention, and do not edit `AGENTS.md` without explicit user direction — changes to curated rules require human approval.

- **[`backend/docs/conventions/`](./backend/docs/conventions/)** — Express+Socket.IO+chokidar idioms; route-module features (`browse`, `configure-watch`). Start at `backend/docs/conventions/README.md`.
- **[`frontend/docs/conventions/`](./frontend/docs/conventions/)** — Vue 3 + Pinia + `@vueuse/core` idioms; UI features (`agents`, `inspector`, `inventory`, `issues`). Start at `frontend/docs/conventions/README.md`.
- **[`shared/docs/conventions/`](./shared/docs/conventions/)** — Public API surfaces of the zod-schema-and-event-map package. Start at `shared/docs/conventions/README.md`.

To regenerate after substantive code changes: `/progressive-context:discover` → review `.discovery.md` per workspace → `/progressive-context:scaffold` (no-op if subtree exists) → `/progressive-context:author`. Idempotent — already-written docs are never overwritten without an explicit `--update`.

### TypeScript / code style

- **Strict mode** in every `tsconfig.json` (`strict: true`, `noUncheckedIndexedAccess: true`).
- **Prettier**: 2-space indent, single quotes, semicolons **on**, trailing commas where valid.
- **ESLint flat config** at the repo root; both workspaces extend it.
- **Naming**: `camelCase` for variables/functions, `PascalCase` for types and Vue components, `kebab-case.vue` for component filenames, lowercase `.ts` for everything else.
- **Imports**: relative within a workspace; `@harness-visualizer/shared` for shared zod schemas, types, and Socket.IO event-map interfaces.
- **No barrel files** (`index.ts` re-exports) unless they remove genuine duplication. (`shared/src/index.ts` is a deliberate exception — it gives consumers one entry point across schemas/types/events.)
- **ESM throughout**. `__dirname` is undefined in ESM modules; use `fileURLToPath(import.meta.url)`. TS imports of local files include explicit `.js` extensions (e.g., `import { x } from './foo.js'` even from a `.ts` source file) — required by `module: NodeNext`.
- **Single root `vite.config.ts`** is the source of truth for both Vite dev/build and Vitest project config; no per-workspace `vitest.config.ts` or deprecated `vitest.workspace.ts`. The root `vitest.config.ts` orchestrates all three workspaces via `test.projects`.

### Vue conventions

> Per-idiom and per-feature docs grounded in this codebase live at [`frontend/docs/conventions/`](./frontend/docs/conventions/).

- All components use `<script setup lang="ts">`. No Options API.
- State management: **Pinia stores only**. No global event buses.
- Styling: Tailwind v4 utility classes. Prefer utilities over scoped CSS; reach for `<style scoped>` only when a utility class would be unreadable.
- Component tests via `@vue/test-utils` + Vitest's `jsdom` environment.
- **Vite dev server**: `vite.config.ts` MUST set `server.host: '127.0.0.1'`, `server.strictPort: true`, `server.cors: false`. Mirrors the backend bind rule — a stray `vite --host 0.0.0.0` would expose dev sources to the LAN.

### Backend conventions

> Per-idiom and per-feature docs grounded in this codebase live at [`backend/docs/conventions/`](./backend/docs/conventions/).

- Express handlers stay thin — parse + validate + delegate. Business logic lives in `scanner/`, `scoring/`, etc.
- Socket.IO uses a **typed event map** shared with the frontend via `shared/`.
- Debounce file events at ~250 ms before emitting `harness:update`.
- Tests use **real filesystem** in `tmp` dirs. Do not mock `fs` — the scanner code paths are small enough that integration coverage is more valuable than mock fidelity.
- **chokidar v5 footgun**: `unwatch(parent)` adds a *recursive ignore matcher* that silently filters events from `parent/sub` even if `sub` was added separately. Always dedup parent/child overlap server-side BEFORE the diff, and call `unwatch(removed)` BEFORE `add(added)` when reconfiguring watch roots. Verified by source-reading at `node_modules/chokidar/index.js:398-403`; see `specs/directory-picker/research/requirements-analyst.md` Q3.4.
- **All file IO MUST go through `safeOpen`** (`backend/src/api/safe-open.ts`) — TOCTOU-safe primitive that does `realpath` → `lstat` → `open(O_NOFOLLOW)` → `fstat` → `(ino, dev)` compare. Symlinks are followed exactly once via realpath; subsequent symlink swaps are rejected. Atomic writes use sibling-tmp + `O_CREAT|O_EXCL|O_NOFOLLOW` + `fsync` + `rename` (cleanup-on-error). See `specs/markdown-editor/research/synthesis.md` for the full threat model + empirical verification on macOS APFS.
- **`STANDARD_IGNORES`** (`backend/src/watcher/watcher.ts`) MUST include `'**/.harness-visualizer/**'`. App-managed config files (`patterns.json`, future `.harness-visualizer/cache/**`) are written by the app and would otherwise trigger redundant chokidar events on top of the explicit `watcher.rescan()` calls. Triggering re-scans is the responsibility of the handler that wrote the config; chokidar handles user-edited project content only.

### Branch + commit naming

- Branches: `feature/<spec-name>` for spec work, `fix/<short-description>` for bug fixes.
- Commits: imperative mood, present tense ("add scanner pattern loader", not "added"). Reference the spec name when relevant.

---

## Quality gates

Before declaring any task complete, all of these must pass clean from the repo root:

```bash
npm run typecheck      # tsc --noEmit across both workspaces
npm run lint           # eslint, exits non-zero on warnings
npm run test           # vitest run, both workspaces
```

For UI changes: run `npm run dev`, exercise the feature in a browser (Playwright via the `playwright-skill` is fine), and verify the live-update loop still works. **Type-checking and tests verify code correctness, not feature correctness.**

---

## Spec workflow

This project uses the [`spec`](./.claude/plugins/) plugin pipeline for every phase. The flow:

1. `/spec:init <phase-name>` — kicks off a new spec; phase names come from `ROADMAP.md`.
2. Feed the relevant `ROADMAP.md` section + `harness-visualizer-plan.md` as discovery sources.
3. `/spec:discover` → `/spec:research` → `/spec:requirements` → `/spec:design` → `/spec:tasks`.
4. **Implementation**:
   - Multi-agent (`/team:implement` via the `team` plugin) for phases marked multi-agent in the roadmap.
   - Single-agent (`implementation-skill`) for small, well-bounded phases.
5. Update the phase status in `ROADMAP.md` (`✅`, `🔄`, `⏸️`) on completion.

**One spec per phase** unless a phase is explicitly split. Don't merge phases — the boundaries are deliberate.

---

## Roadmap (summary)

Full detail in [`ROADMAP.md`](./ROADMAP.md). Headline view:

| Phase | Spec name | Workflow |
|---|---|---|
| 0 | Bootstrap | (complete — this file, `CLAUDE.md`, `ROADMAP.md`, plugins, settings) |
| 1 | `workspace-skeleton` | single-agent |
| 2 | `scanner-watcher-backend` | multi-agent |
| 3 | `graph-frontend` | multi-agent |
| 4 | `directory-picker` | single-agent |
| 5 | `markdown-editor` | multi-agent (security surface) |
| 6 | `pattern-editor` | single-agent |
| 7 | `score-panel-polish` | single-agent |

---

## Out of scope for v1

These are deliberate non-goals — don't let them creep into early phases:

- Multi-user / network access
- Authentication / authorization
- Hosted or SaaS deployment
- Cross-tool harness conversion (Claude → Cursor, etc.)
- Rich scoring rules beyond presence + structural validity
- Pattern-template sharing/import
- Remote project scanning (SSH, git URL)

If a phase pulls one of these in, push back and defer to a future spec.

---

## Working agreement for agents

- **Read this file first.** Then `ROADMAP.md`. Then the relevant spec under `specs/`. Only then start touching code.
- **Don't add features the spec doesn't ask for.** Bug fixes don't need cleanup. One-shot operations don't need helpers. No premature abstractions.
- **No backwards-compat shims.** This is a greenfield demo — change code directly rather than layering compatibility hacks.
- **No comments unless the *why* is non-obvious.** Well-named code documents itself.
- **No new top-level docs without being asked.** This file, `CLAUDE.md`, and `ROADMAP.md` are the only repo-root docs the project needs.
- **Run quality gates before declaring done.** `typecheck`, `lint`, `test` — all green.
- **Verify UI changes in a browser**, not just via tests.

---
> Source: [willmarple/harness-visualizer](https://github.com/willmarple/harness-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
