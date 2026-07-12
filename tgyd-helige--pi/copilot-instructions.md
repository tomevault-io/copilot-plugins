## pi

> This monorepo contains Pi extensions (`packages/pi-*`), a shared library (`packages/shared`), and integration tests (`tests/`). Most packages are **Pi extensions** — TypeScript modules that hook into the Pi agent runtime.

# Pi Extensions Monorepo — Agent Guidelines

## Project Overview

This monorepo contains Pi extensions (`packages/pi-*`), a shared library (`packages/shared`), and integration tests (`tests/`). Most packages are **Pi extensions** — TypeScript modules that hook into the Pi agent runtime.

---

## Pi Extension Fundamentals

> Full docs: https://pi.dev/docs/latest/extensions

### Entry Point

Every extension exports a default factory receiving `ExtensionAPI`:

```typescript
import type { ExtensionAPI } from '@earendil-works/pi-coding-agent';

export default function (pi: ExtensionAPI) {
  pi.on('session_start', async (event, ctx) => { /* ... */ });
  pi.registerTool({ /* ... */ });
  pi.registerCommand('name', { /* ... */ });
}
```

### Lifecycle

Events grouped by phase:

**Session-level:**
1. `project_trust`
2. `session_start`
3. `resources_discover`

**Agent-level (per user prompt):**
4. `input`
5. `before_agent_start`
6. `agent_start`

**Per-turn loop** (repeats until agent is done):
- `turn_start`
- `context`
- `before_provider_request`
- tool calls (may be multiple per turn)
- `turn_end`

**Agent completion:**
7. `agent_end`

**Shutdown:**
8. `session_shutdown`

Session transitions (`/new`, `/resume`, `/fork`) emit `session_before_switch`/`session_before_fork`, then `session_shutdown`, then `session_start` on the new session.

Note: `agent_end` fires after each user prompt completes; `session_shutdown` only fires on session exit or transition — not between prompts in the same session.

### Key APIs

| API | Purpose |
|-----|---------|
| `pi.on(event, handler)` | Subscribe to lifecycle events |
| `pi.registerTool({...})` | Register a tool callable by the LLM |
| `pi.registerCommand(name, {handler, ...})` | Register a `/slash` command |
| `pi.registerShortcut(...)` | Keyboard shortcuts |
| `pi.registerProvider(...)` | Model providers |
| `pi.appendEntry(key, data)` | Persist state in session (not sent to LLM) |

### Tool Registration

```typescript
pi.registerTool({
  name: 'my_tool',
  label: 'My Tool',
  description: 'What this tool does',
  parameters: Type.Object({ /* typebox schema */ }),
  // Use StringEnum from @earendil-works/pi-ai for string enums (Google compatibility)
  promptSnippet: 'One-line for Available tools list',
  promptGuidelines: ['Use my_tool when...'],
  async execute(toolCallId, params, signal, onUpdate, ctx) {
    return { content: [{ type: 'text', text: 'result' }], details: {} };
  },
});

// On expected error (config missing, invalid input, etc.):
// return { isError: true, content: [{ type: 'text', text: 'sanitized message' }], details: {} };
```

**Tool output must be bounded** — tool results flow into `pi-coding-agent`, which truncates at `DEFAULT_MAX_BYTES` (50KB) and `DEFAULT_MAX_LINES` (2000). Extensions should proactively keep outputs below these limits rather than rely on runtime truncation.

Truncation priority:
1. If the package already has a local `truncateText` or similar helper, use it (keeps the package internally consistent)
2. Otherwise import `truncateHead`/`truncateTail` from `@earendil-works/pi-coding-agent` (v0.74+)
3. Do not pull in external deps just for truncation

What to truncate:
- Large strings, large arrays, and external response bodies in both `content` and `details`; preserve small structured metadata as-is
- Error messages — a stack trace can easily exceed limits

### ExtensionContext (`ctx`)

Every handler receives `ctx` (type: `ExtensionContext` from `@earendil-works/pi-coding-agent`):

| Field | Purpose |
|-------|---------|
| `ctx.ui` | User interaction: confirm, select, input, notify, setStatus |
| `ctx.cwd` | Current working directory |
| `ctx.mode` | `'tui'` \| `'rpc'` \| `'json'` \| `'print'` |
| `ctx.hasUI` | Boolean — guard dialog methods with this |
| `ctx.signal` | AbortSignal for cancellation — long-running tool calls must propagate this to fetch/child processes; the runtime cancels via this signal |
| `ctx.modelRegistry` | Model access and lookup |
| `ctx.sessionManager` | Read-only session state |
| `ctx.getContextUsage()` | Token usage info |
| `ctx.compact()` | Trigger compaction |
| `ctx.shutdown()` | Graceful exit |

---

## Pi Packages

> Full docs: https://pi.dev/docs/latest/packages

Packages bundle extensions + skills + prompts + themes. Manifest in `package.json`:

```json
{
  "pi": {
    "extensions": ["./dist/index.js"],
    "skills": ["./skills"]
  }
}
```

Only include `skills` when the package actually ships skill files. See the Adding a New Package checklist below for the full setup.

Core Pi packages (`@earendil-works/pi-ai`, `pi-agent-core`, `pi-coding-agent`, `pi-tui`, `typebox`) must go in `peerDependencies` — never bundle these. Version ranges should follow the existing convention in this repo (e.g. `">=0.74.0"`); check a sibling package for the current pattern before adding.

---

## Preview Images

Pi extension packages should ship a package-root `preview.png` (for example, `packages/pi-foo/preview.png`) and reference it from `package.json` via `pi.image` when the package is intended to appear in extension/package listings.

Preview requirements:
- Use the repo's current preview size: `1672 x 941` PNG
- Keep the overall layout consistent across packages: left-side extension text, right-side UI/icon cards, and a central beaver mascot
- Do not reuse the same beaver image across packages; each extension should have a distinct mascot pose, outfit, expression, or instrument
- Prefer a blue/white visual system; light or dark accents are fine when the result remains polished and consistent
- Include a musical instrument when possible; Chinese and Western instruments are both welcome, and the instrument should help differentiate the mascot
- The left-side title/tagline must be strongly related to the extension's actual purpose
- The right-side UI/icon cards must be strongly related to the extension's domain and capabilities, not generic decoration
- Keep the left text and right card styling visually aligned with existing previews so the package grid feels unified

When changing an extension's name, tagline, or scope of capabilities, update the package `preview.png` together with the root README extension list, the package README, and `package.json` metadata.

---

## Using `@packages/shared`

When you need to read Pi's global config directory, project directory, or settings — use the shared module. Do NOT hardcode paths like `~/.pi/agent/`.

This monorepo uses two npm scopes: `@earendil-works/*` for the Pi runtime core packages and `@amaster.ai/*` for our internal shared utilities. The shared package exposes two entry points: `@amaster.ai/pi-shared` (shared domain types and runtime type definitions) and `@amaster.ai/pi-shared/settings` (config/path utilities). **Import from `/settings` for settings and path utilities** — this is what all existing packages do:

```typescript
import {
  resolveHome,        // ~/.pi/agent (data dir: memories, skills, logs)
  resolveConfigDir,   // config dir (settings.json, policy, auth, models)
  loadPiSettings,     // load extension config (merges global < agentDir < project)
  loadPiPolicyProfiles,
} from '@amaster.ai/pi-shared/settings';
```

### Path Resolution

| Function | Purpose | Env Override |
|----------|---------|--------------|
| `resolveHome()` | User data / runtime home | `PI_AGENT_HOME` |
| `resolveConfigDir()` | System config dir | `PI_CODING_AGENT_DIR` or `PI_AGENT_HOME` |

### Settings Loading

`loadPiSettings<T>(key, options?)` merges config from three layers (low → high priority):
1. Global: `~/.pi/agent/settings.json`
2. Agent dir: `$PI_CODING_AGENT_DIR/settings.json`
3. Project: `<cwd>/.pi/settings.json`

Supports `${ENV_VAR:-fallback}` interpolation in values.

---

## Unit Test Convention

Tests must live in `src/__tests__/`.

### Required Structure

```
packages/pi-foo/
  src/
    __tests__/
      extension.test.ts
      config.test.ts
      ...
    index.ts
    extension.ts
```

### Rules

1. Test files go in `src/__tests__/<module>.test.ts`
2. Use vitest: `import { describe, expect, it, vi } from 'vitest'`
3. Top-level `describe` block named after the module/class under test
4. Use `it(...)` (not `test(...)`) for individual cases
5. Temp directories: `path.join(tmpdir(), 'pi-<pkg>-<suite>')` with `beforeEach`/`afterEach` cleanup. The `pi-` prefix isolates parallel suites and makes leftover dirs easy to grep/clean
6. Mock the ExtensionAPI with a minimal object capturing `registerTool`, `registerCommand`, `on` calls
7. Run: `pnpm test` (root) or `pnpm vitest run` (per-package)

**Unit tests vs integration tests**: `src/__tests__/` holds unit tests scoped to one package. Cross-package and end-to-end tests live in the top-level `tests/` directory.

### Mock Pattern for Extensions

```typescript
const tools: Map<string, unknown> = new Map();
const commands: Map<string, unknown> = new Map();

const mockPi = {
  on: vi.fn(),
  registerTool: vi.fn((t) => tools.set(t.name, t)),
  registerCommand: vi.fn((name, opts) => commands.set(name, opts)),
  appendEntry: vi.fn(),
  // Extend as needed: registerShortcut, events, etc.
};
```

---

## Tool + Command Registration

**When registering a tool, also register a corresponding `/command`** so users can invoke it directly via slash command.

- **New tools**: must have a matching command (hard requirement)
- **Existing tools**: commands should be added over time (soft requirement, tracked as tech debt)

Exceptions — do NOT add a command when:
- The tool is an internal bridge or low-level MCP wrapper (not user-facing)
- The tool performs dangerous/destructive operations that shouldn't be casually invokable
- The tool has no stable human-friendly parameter form (e.g. takes complex nested JSON)

```typescript
// Tool (LLM-callable)
pi.registerTool({
  name: 'web_search',
  // ...
  async execute(toolCallId, params, signal, onUpdate, ctx) { /* ... */ },
});

// Matching command (user-callable via /web-search <query>)
pi.registerCommand('web-search', {
  description: 'Search the web',
  args: [{ name: 'query', description: 'Search query', required: true }],
  async handler(args, ctx) { /* reuse tool logic */ },
});
```

Packages that currently lack commands for their user-facing tools should be backfilled when touching the package — no formal tracker; surface in PR review.

---

## Logging

Use `console.error` for all log output in extension code — stdout may be consumed by the runtime (e.g. stdio transport), only stderr is a safe side-channel.

| Scenario | Log? | Example |
|----------|------|---------|
| Error / unexpected exception | Always | `console.error(`[pi-foo] failed to fetch: ${err.message}`)` |
| Config loaded / connection established | Once at startup | `console.error(`[pi-foo] connected to ${host}`)` |
| Normal tool execution | No | Rely on the runtime's tool event store |
| Debug / verbose tracing | Gated only | Use `DEBUG=pi-foo*` env-based gate; never leave unconditional debug logs in PRs |

Rules:
- Prefix with `[package-name]` for easy grep
- Never log sensitive data (tokens, keys, user content)
- No `console.log` in extension code — it pollutes stdout
- Exception: CLI scripts / migration tools where stdout IS the expected machine-readable output may use `console.log` for that output; diagnostic messages still go to stderr

---

## Implementation Discipline

- Before modifying code, look at the same package's existing patterns first — match style, naming, structure
- Do not make cross-package refactors unrelated to the current task
- New config keys must have corresponding test coverage
- Modifying a public tool or command's parameters requires updating its tests and README; update `promptSnippet` and `promptGuidelines` when the user-facing behavior or tool selection guidance changes
- Do not swallow exceptions — expected errors (config missing, user input invalid) should return sanitized `isError: true` results; unexpected exceptions should log to stderr but not expose full stack traces or raw request/response bodies to the user
- Keep changes minimal and scoped; don't "improve" surrounding code unless asked

---

## Security / Secrets

- Never log tokens, API keys, cookies, or full user message content
- Never embed secrets in source — use env vars and `loadPiSettings` with `${ENV_VAR}` interpolation
- Error messages exposed to users must not leak internal secrets or full stack traces
- When reading config that may contain credentials, validate early and fail clearly rather than passing bad values downstream
- Enforcement: no automated pre-commit hook currently — relies on code review. Reviewers should check for: raw error objects passed to `console.error`, response bodies logged verbatim, secrets in string interpolation, and stack traces reaching tool results

---

## Adding a New Package

Minimal checklist:

1. Copy a sibling `package.json` — update `name`, `description`, `peerDependencies` version range
2. Create `src/index.ts` (extension entry) + `src/__tests__/`
3. Add `"pi": { "extensions": ["./dist/index.js"] }` to `package.json` (see Pi Packages section for the full `pi` block when adding skills/prompts/themes)
4. Add `README.md`; add `preview.png` and `pi.image` when the package appears in listings (see Preview Images section)
5. Register in `pnpm-workspace.yaml` if not auto-discovered by glob
6. Run `pnpm build && pnpm test` from root to verify integration

---

## Build & Test

```bash
pnpm build        # Build all packages
pnpm test         # Run all tests (vitest)
pnpm typecheck    # Type-check all packages
pnpm lint         # Biome check
pnpm lint:fix     # Biome auto-fix
pnpm pr-check     # build + test + typecheck (CI gate)
```

---
> Source: [TGYD-helige/pi](https://github.com/TGYD-helige/pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
