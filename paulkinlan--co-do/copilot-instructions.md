## co-do

> Guidance for Claude Code working in this repository.

# CLAUDE.md

Guidance for Claude Code working in this repository.

## Project Overview

Co-do is a Cowork-like browser experience using the File System Access API for collaborative coding with native filesystem integration.

> **IMPORTANT: Always run `npm test` after code changes.** Tests must pass before committing.
> **Exception:** Not required for documentation-only changes (`.md` files, comments).

> **IMPORTANT: Never modify `version` in `package.json`.** Version bumping is automatic via GitHub Action on PR merge.

## Browser Support

**Target: Latest Chrome (Chrome 140+)** — File System Access API is the core dependency.

- Use modern web platform APIs freely; no polyfills or legacy support needed
- Chrome-specific APIs acceptable; focus on Baseline Newly/Widely Available features
- See `.claude/skills/modern-web-dev/SKILL.md` for detailed guidance

## Development Commands

| Command | Description |
|---------|-------------|
| `npm install` | Install dependencies |
| `npm run dev` | Vite dev server on http://localhost:3000 |
| `npm run build` | Full production build (WASM + TS + Vite → `dist/`) |
| `npm run build:web-only` | Build without WASM compilation |
| `npm run type-check` | TypeScript type checking only |
| `npm run preview` | Preview production build |
| `npm test` | All Playwright tests (visual + accessibility) |
| `npm run test:visual` | Visual regression tests only |
| `npm run test:accessibility` | Accessibility tests only (WCAG 2.1 AA) |
| `npm run test:visual:update` | Update baseline screenshots |
| `npm run test:unit` | Vitest unit tests |
| `npm run test:unit:watch` | Unit tests in watch mode |
| `npm run test:ui` | Playwright interactive UI mode |
| `npm run test:debug` | Debug mode with Playwright Inspector |
| `npm run test:report` | Open HTML test report |
| `npm run wasm:build` | Build all WASM tools (requires wasi-sdk) |
| `npm run wasm:build:native` | Build native-only WASM tools |

## Project Structure

```
Co-do/
├── src/
│   ├── main.ts               # Entry point, PWA init, version checking
│   ├── ui.ts                  # UI manager, event handlers, conversations, permissions UI
│   ├── ai.ts                  # AI SDK integration (streaming, multi-provider, dynamic import)
│   ├── tools.ts               # 20 file operation tools + pipe command
│   ├── pipeable.ts            # Self-registering pipeable command registry
│   ├── fileSystem.ts          # File System Access API wrapper with caching
│   ├── preferences.ts         # Tool permissions (ToolName type, permission levels)
│   ├── storage.ts             # IndexedDB manager (configs, conversations, workspaces, WASM tools)
│   ├── router.ts              # Hash-based workspace URL routing
│   ├── diff.ts                # Unified diff generation (LCS algorithm)
│   ├── markdown.ts            # Markdown rendering in sandboxed iframes (XSS protection)
│   ├── toasts.ts              # Toast notification system
│   ├── notifications.ts       # Native browser notifications
│   ├── viewTransitions.ts     # View Transitions API integration
│   ├── provider-registry.ts   # Provider cookie management + CSP coordination
│   ├── network-monitor.ts     # CSP violation monitoring, network request logger, visual firewall
│   ├── tool-response-format.ts # Pure functions for tool response formatting
│   ├── toolResultCache.ts     # Caching large tool outputs (>2KB → summary to AI, full to UI)
│   ├── styles.css             # CSS with custom properties for dark mode
│   └── wasm-tools/            # WebAssembly custom tools system
│       ├── manager.ts         # Central tool orchestrator
│       ├── runtime.ts         # WASI runtime (main-thread fallback)
│       ├── wasm-worker.ts     # Worker-based WASM runtime (default, sandboxed)
│       ├── worker-manager.ts  # Worker lifecycle and pooling
│       ├── vfs.ts             # Virtual file system (WASI syscall interception)
│       ├── loader.ts          # ZIP package loader and manifest validator
│       ├── registry.ts        # Built-in tool config (41 tools by category)
│       ├── types.ts           # TypeScript interfaces and Zod schemas
│       ├── worker-types.ts    # Worker message protocol types
│       ├── index.ts           # Public API exports
│       └── adapters/          # Non-WASI tool adapters (FFmpeg)
├── server/
│   ├── main.ts                # Vite server plugins entry
│   ├── providers.ts           # Provider registry and cookie parsing
│   └── csp.ts                 # Dynamic CSP header generation per provider
├── tests/
│   ├── visual/                # Visual regression tests (screenshots)
│   ├── accessibility/         # WCAG 2.1 Level AA compliance tests
│   ├── unit/                  # Unit tests (Vitest)
│   └── helpers/               # Test utilities (test-utils.ts, dom-inspector.ts)
├── docs/                      # Architecture docs (WASM, CSP, security)
├── public/                    # PWA manifest, service worker, icons
├── wasm-tools/                # C/WASI source, manifests, build scripts
├── index.html                 # HTML entry point (UI structure, modals, permissions)
├── vite.config.ts             # Vite config with dynamic CSP + version plugin
├── playwright.config.ts       # Playwright test configuration
├── tsconfig.json              # TypeScript strict mode config
└── package.json               # Dependencies and scripts
```

## Key Dependencies

**Runtime:** Vercel AI SDK (`ai` ^6.0.39) with `@ai-sdk/anthropic`, `@ai-sdk/openai`, `@ai-sdk/google`, `@openrouter/ai-sdk-provider`; `zod` for schema validation; `marked` for markdown; `jszip` for WASM package uploads.

**Dev:** Vite ^6.0.7, TypeScript ^5.7.2 (strict, ES2022), Playwright ^1.57.0, Vitest ^4.0.18, @axe-core/playwright ^4.11.0.

**Key tech:** File System Access API, WebAssembly + WASI (41 built-in tools in Workers), IndexedDB (primary storage), localStorage (preferences), dynamic ES Module imports, View Transitions API, Web Speech API. AI uses `stepCountIs(10)` step limits.

## Architecture Guidelines

1. **Modern APIs First**: Prefer modern web platform APIs. See `.claude/skills/modern-web-dev/SKILL.md`.

2. **File System Access API**: Core feature. Uses `window.showDirectoryPicker()`, FileSystemFileHandle/DirectoryHandle. Directory handles persisted in IndexedDB for session restoration.

3. **Security**: Handle permission requests/denials gracefully. All file ops sandboxed to user-selected directory.

4. **Adding New File Operation Tools** (in `src/tools.ts`):
   - Add tool name to `ToolName` type in `src/preferences.ts`
   - Add default permission in `DEFAULT_PERMISSIONS` in `src/preferences.ts`
   - Add permission UI element in `index.html` inside `#tool-permissions`
   - Use `checkPermission()` in the tool's execute function

5. **Adding Pipeable Commands**: Import `registerPipeable` from `src/pipeable.ts`, call at module evaluation time. The pipe tool discovers registered commands automatically.

6. **Adding Built-in WASM Tools**:
   - Add C/WASI source in `wasm-tools/src/`, config in `BUILTIN_TOOLS` in `src/wasm-tools/registry.ts`
   - Assign a functional `category` (existing: `text`, `data`, `crypto`, `file`, `code`, `search`, `compression`, `database`). New categories need entries in `CATEGORY_DISPLAY_NAMES` and `CATEGORY_DISPLAY_ORDER`
   - Build with `npm run wasm:build`; set `pipeable: true` for pipe chain support
   - **Binary data**: Use parameter type `'binary'` (AI sends base64 → decoded to bytes via stdin). Binary stdout auto-detected and preserved as `stdoutBinary`. VFS uses `readFileBinary()`/`Uint8Array` writes. Worker uses `Transferable` ArrayBuffers.

7. **Tool Grouping**: Always group by function, never by implementation. Built-in tools grouped statically in `index.html`; WASM tools grouped dynamically by `category` via `CATEGORY_DISPLAY_NAMES`/`CATEGORY_DISPLAY_ORDER` in `src/wasm-tools/registry.ts`.

8. **IndexedDB Storage**: All persistent data via `src/storage.ts` `storageManager` singleton — never access IndexedDB directly. Separate stores for provider configs, workspaces, conversations, WASM tools. API keys in IndexedDB (not localStorage).

9. **Dynamic CSP**: Provider cookie (`co-do-provider`) → `server/csp.ts` builds per-provider `connect-src`. Vite code-splits provider SDKs. Workers inherit page CSP.

10. **Tool Result Caching**: Outputs >2KB cached in `toolResultCache` — AI gets summary + first 5 lines; UI gets full content via `resultId`.

11. **Collaboration Features**: Consider real-time sync, conflict resolution, presence indicators, efficient file content transfer.

## Documentation Guidelines

**Keep docs up-to-date with every feature.** Update:
- **README.md** — new features, tools, security, browser support, config, architecture
- **CHANGELOG.md** — new versions, significant features, breaking changes, bug fixes
- **PWA-SETUP.md** — PWA config, service worker, manifest changes
- **CLAUDE.md** — new commands, structure changes, new guidelines, PR review lessons

**Before marking a feature complete:** Verify README reflects it, features are listed, example prompts updated, security documented, new tools listed, architecture changes reflected.

**Document:** What it does, how to use it, configuration, security considerations.
**Skip:** Internal refactoring, non-behavior-changing bug fixes, comment improvements, test-only changes.

## Asking Clarifying Questions

Ask before proceeding when: multiple valid approaches exist, design trade-offs are involved, scope is ambiguous, new features requested, breaking changes possible, or UI/UX decisions needed.

**How to ask:** Be specific, offer 2-3 options with trade-offs, suggest a default with reasoning.

**Skip questions when:** task is straightforward, following established patterns, user specified exactly what they want, clear bug fix, or docs-only changes.

Proactively suggest related improvements while keeping user in control of scope.

## Code Review Skills (PR Review Toolkit)

Review agents in `.claude/skills/` **must be used** for all code changes.

| Skill | Purpose |
|-------|---------|
| **code-reviewer** | Reviews against CLAUDE.md guidelines (confidence >= 80) |
| **silent-failure-hunter** | Finds silent failures, empty catch blocks |
| **pr-test-analyzer** | Evaluates test coverage, identifies gaps |
| **type-design-analyzer** | Rates type design quality |
| **comment-analyzer** | Verifies comment accuracy, finds comment rot |
| **code-simplifier** | Simplifies for clarity and maintainability |

**`/review-pr`** — runs all applicable agents. Target specific: `code`, `tests`, `errors`, `types`, `comments`, `simplify`.

### Required Pre-Commit Workflow

1. Write code
2. Run **code-reviewer**
3. Run **silent-failure-hunter** (if error handling touched)
4. Run **pr-test-analyzer** (if tests changed)
5. Run **type-design-analyzer** (if types changed)
6. Fix all critical (90-100) and important (80-89) issues
7. Run **code-simplifier** (final pass)
8. Run `npm test`
9. Commit

**Skip reviews for:** docs-only changes, non-runtime config changes, test baseline updates.

## Testing Guidelines

**All UI changes must include tests.**

Uses Playwright (visual regression + accessibility) and Vitest (unit tests).

### When Tests Are Required

Adding/modifying UI components, adding features, changing styles, adding modals/overlays, implementing responsive design → write visual regression + accessibility tests.

### When Tests Are NOT Required

`.md` files, comment-only changes, README updates, license changes, config file comments.

### Test Types

- **Visual regression** (`tests/visual/`): Screenshot comparison. Use `animations: 'disabled'`. Catches layout breaks, misalignment, style regressions.
- **Accessibility** (`tests/accessibility/`): WCAG 2.1 AA via axe-core. Contrast (4.5:1 normal, 3:1 large), ARIA, keyboard nav, focus, semantic HTML.
- **Unit** (`tests/unit/`): Vitest for pure functions (tool response formatting, pipe logic).

### Best Practices

- Disable animations in screenshots; wait for `networkidle`
- Test all states: normal, hover, focus, active, disabled
- Use IDs/data-testid over CSS classes
- One test = one thing; reuse helpers from `tests/helpers/test-utils.ts`
- Test mobile viewports with `page.setViewportSize()`
- All text must meet WCAG AA contrast (4.5:1)

### Updating Baselines

When intentionally changing UI: make changes → `npm run test:visual:update` → review snapshots → commit code + screenshots together.

### CI/CD

Tests run on every PR, push to main, and feature branch commit. Failing tests = failing build.

## Pre-Push Rebase Workflow

**Always rebase onto latest main before pushing.** Commit first — never rebase with uncommitted changes.

1. `git add <files> && git commit -m "message"`
2. `git fetch origin main`
3. `git rebase origin/main`
4. Resolve conflicts (for `package-lock.json`: `git checkout --ours package-lock.json && npm install && git add package-lock.json && git rebase --continue`)
5. If lockfile regenerated: `git add package-lock.json && git commit --amend --no-edit`
6. Run `npm test` (for code changes)
7. `git push -u origin <branch> --force-with-lease`

## GitHub PR Review Feedback

1. Fix all reviewer comments
2. Run `npm test` after fixes
3. If feedback reveals a recurring pattern, add it to Lessons Learned below and update relevant CLAUDE.md sections

## Lessons Learned from PR Reviews

<!-- Add lessons learned from PR reviews below this line -->

#### 1. Empty String vs Falsy — Use Explicit Undefined Checks

When a value can be a valid empty string, use `value !== undefined` or `value != null` — not `if (value)` which silently drops `""`. Applies to stdin, search queries, file contents, user text.

```typescript
// BAD: if (stdin) { ... }
// GOOD: if (stdin !== undefined) { ... }
```

#### 2. One-Time Init Must Handle Failure Gracefully

Only set initialization flags on **successful** completion. Use three-state (`unloaded | loading | loaded`) for lazy-loading. Never mark loaded before knowing the operation succeeded.

#### 3. Concurrent Access — Deduplicate Lazy Initialization

Use a shared Promise so the first caller creates it and subsequent callers await the same one, preventing redundant fetches and race conditions.

```typescript
let loadPromise: Promise<void> | null = null;
function ensureLoaded() {
  if (!loadPromise) loadPromise = fetch(url).then(r => { data = r; });
  return loadPromise;
}
```

#### 4. Update All Related State When Syncing Cached Entities

Treat manifest + binary (or config + data) as atomic units. Partial syncs create mismatches.

#### 5. Dynamic Registries Must Stay in Sync

Update registries when source data changes (install, enable, disable, uninstall). Rebuild derived descriptions after modifications. Prefer getters/callbacks over cached-once values.

#### 6. URL and Path Handling — Strip Query Strings, Handle Dev vs Prod

Strip query strings/hash fragments before path matching. Handle both `.ts` (dev) and `.js` (prod) extensions.

```typescript
const pathname = url.split(/[?#]/, 1)[0];
const segment = pathname.split('/').pop() ?? '';
return segment.startsWith('wasm-worker') && (segment.endsWith('.js') || segment.endsWith('.ts'));
```

#### 7. JSON.stringify Is Unreliable for Deep Equality

Don't use `JSON.stringify` for comparison — property ordering varies. Use deep-equal utilities or normalize objects first.

#### 8. MessagePort and Worker Cleanup

`MessagePort` doesn't fire `close` on tab unload. Wrap `postMessage` broadcasts in try-catch, remove failed ports. Use explicit disconnect protocols (`beforeunload` → disconnect message).

#### 9. Permission Checks Must Be Granular

Pipeable commands must respect their own tool-specific permission, not just the global pipe permission. Pre-validate every command in a chain against its specific permission.

#### 10. GitHub Actions — Validate Workflow Syntax and Safety

- Valid top-level keys only: `name`, `on`, `permissions`, `env`, `defaults`, `concurrency`, `jobs`
- Least privilege permissions; add `concurrency` groups; make operations idempotent
- Create tags **after** rebase/push; pin action versions to SHAs/exact tags

#### 11. Comments Must Match Actual Behavior

Verify every factual claim in comments/docstrings against actual code: numeric thresholds, function signatures, parameter names, git semantics, referenced files. PR reviews repeatedly caught wrong terminology, incorrect thresholds, and references to nonexistent code.

#### 12. Wrap Throwing Calls in Try-Catch at System Boundaries

`atob`, `JSON.parse`, `new URL`, `decodeURIComponent` can throw on invalid input. Wrap at first use point, converting exceptions to meaningful error responses.

#### 13. Avoid Redundant File I/O — Cache Parsed Configuration

Read and parse shared config files (e.g., `package.json`) once; share the result across functions.

---
> Source: [PaulKinlan/Co-do](https://github.com/PaulKinlan/Co-do) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
