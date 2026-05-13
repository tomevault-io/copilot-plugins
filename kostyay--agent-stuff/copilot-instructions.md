## agent-stuff

> This repo contains pi coding agent extensions, skills, and themes.

# Agent Notes

## Overview

This repo contains pi coding agent extensions, skills, and themes.

- **Extensions** live in `./pi-extensions/` (single `.ts` files, one per extension).
- **Skills** live in `./skills/`.
- **Themes** live in `./pi-themes/`.

### Canonical Extension Documentation

Always fetch the latest pi extension docs before creating or modifying extensions:

- **Extensions API**: https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent/docs/extensions.md
- **TUI Components**: https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent/docs/tui.md
- **Session Management**: https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent/docs/session.md
- **Examples**: https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent/examples/extensions/

Use the `summarize` skill to fetch and convert these docs when you need API details. Do **not** rely on memorized patterns — the API evolves.

---

## Code Quality

Apply the `@skills/js-code-simplifier/` skill to **every** code change in this repo. This means:

1. Read every file in scope before editing.
2. Follow the refinement principles: clarity over brevity, explicit types, early returns, sorted imports, no dead code.
3. After changes, list what was simplified and why.

### Additional Rules

- **JSDoc on every export.** All exported functions, types, and constants must have a JSDoc comment explaining purpose and usage.
- **Explicit TypeScript types.** No implicit `any`. Use precise types for function parameters, return values, and tool parameter schemas.
- **`function` declarations** for top-level exports. Arrow functions for callbacks and inline handlers only.
- **Imports sorted** alphabetically by source, grouped: pi packages first, then node built-ins, then relative.

---

## Extension Design Rules

### One Responsibility Per Extension

Each `.ts` file in `pi-extensions/` must own exactly **one** concern:

- A command (e.g., `clear.ts` registers `/clear`)
- A tool (e.g., a standalone `ask_question` tool)
- An event handler (e.g., `notify.ts` handles `agent_end`)
- A UI component (e.g., `status-bar.ts` manages footer status)

If an extension grows beyond ~400 lines, split it. If it does two unrelated things, split it.

### Strict Isolation — No Cross-Extension Imports

Extensions must **not** import from or depend on other extensions in this repo. Each extension is self-contained at the module level.

**Wrong:**
```typescript
// plan-ask.ts
import { askQuestion } from "./kbrainstorm"; // ❌ Cross-extension import
```

**Right:**
Each extension registers its own tools/commands. If two extensions need the same capability, extract it into a shared utility in a `lib/` directory, or better yet, make each extension independently register what it needs.

### Cross-Extension Communication via `pi.events`

Extensions communicate at runtime through the shared event bus (`pi.events`), never through imports. Data-producing extensions emit typed events; consuming extensions listen.

```typescript
// producer (ticket extension)
pi.events.emit("ticket:stats", { total: 14, epics: 1, open: 12, ... });

// consumer (status-bar extension)
pi.events.on("ticket:stats", (data) => { ticketStats = data; });
```

### Status Bar Rendering Architecture

`status-bar.ts` is the **sole owner of footer rendering**. Other extensions must **not** call `ctx.ui.setStatus()` to display information in the footer. Instead:

1. **Data extensions** (e.g., `ticket/`) compute stats and emit them via `pi.events` using a namespaced event (e.g., `"ticket:stats"`).
2. **`status-bar.ts`** listens for these events, stores the data, and renders it in the footer alongside model info, context meter, git stats, and tool tallies.

The status bar **may be aware of specific data shapes** (e.g., ticket stats interface) to render them with appropriate formatting, icons, and color coding. This is intentional — the status bar is a UI orchestrator, not a generic passthrough.

**Wrong:**
```typescript
// ticket extension
ctx.ui.setStatus("ticket", "🎫 14 tickets"); // ❌ Extension rendering its own footer status
```

**Right:**
```typescript
// ticket extension — emit raw data
pi.events.emit("ticket:stats", { total: 14, epics: 1, open: 12, inProgress: 1, closed: 1 });

// status-bar.ts — owns the rendering
pi.events.on("ticket:stats", (data) => { ticketStats = data; });
// ... renders ticketStats in footer with proper theme colors
```

### Naming Conventions

- **File names**: lowercase kebab-case matching the primary command or tool name (e.g., `status-bar.ts`, `notify.ts`).
- **Export function**: Named descriptively — `export default function clearExtension(pi)`, not `export default function (pi)`.
- **Tool names**: snake_case (e.g., `ask_question`, `my_tool`).
- **Command names**: kebab-case (e.g., `/clear`, `/plan-mode`).

### File Header

Every extension file starts with a JSDoc block explaining:

```typescript
/**
 * <Extension Name>
 *
 * <One-line description of what it does.>
 * <Any important behavioral notes.>
 */
```

See `clear.ts` and `notify.ts` for examples of this pattern.

---

## File Structure

```
agent-stuff/
├── AGENTS.md                  # This file — repo conventions
├── package.json               # Pi package manifest
├── pi-extensions/             # All extensions (auto-discovered by pi)
│   ├── clear.ts               # /clear command
│   ├── notify.ts              # Desktop notifications on agent_end
│   ├── kbrainstorm.ts         # ask_question tool (interactive TUI)
│   ├── plan-ask.ts             # /plan and /ask commands (three-way mode rotation)
│   ├── status-bar.ts          # Footer status display
│   └── ...
├── skills/                    # Agent skills (SKILL.md + support files)
│   ├── js-code-simplifier/
│   ├── kbrainstorm/
│   └── ...
└── pi-themes/                 # Custom themes
```

### Single-File Extensions

Most extensions should be single `.ts` files. This is the preferred style for this repo. Only create a directory extension (with `index.ts`) when you genuinely need multiple source files or npm dependencies.

---

## Known Refactoring Needed

### Extract `ask_question` from `kbrainstorm.ts`

The `ask_question` tool in `kbrainstorm.ts` is used by the `plan-ask.ts` extension (via LLM tool calls, not direct import). This creates a runtime coupling: `plan-ask.ts` assumes `ask_question` is available.

**Target state:** Extract the `ask_question` tool into its own `ask-question.ts` extension. The `kbrainstorm` skill's `SKILL.md` should reference the tool by name, and `plan-ask.ts` should document that it expects the `ask_question` tool to be registered by a separate extension.

---

## Verification

Before committing any extension change:

1. **Type-check** — Run `npx tsc --noEmit` if a `tsconfig.json` is present, or manually verify types are correct against the pi package type definitions.
2. **Manual test** — Load the extension in isolation:
   ```bash
   pi -e ./pi-extensions/<extension>.ts
   ```
   Exercise the registered commands/tools/events to confirm behavior.
3. **Check isolation** — Ensure the extension has no imports from other files in `pi-extensions/`. Only import from `@mariozechner/pi-*` packages, `@sinclair/typebox`, and Node built-ins.

---

## Releases

1. Run `npm version <patch|minor|major>` and verify `package.json` updates.
2. Update `CHANGELOG.md` for the release.
3. Commit the release changes and tag with the same version.
4. Push commits and tags, then publish with `npm publish` if needed.

---
> Source: [kostyay/agent-stuff](https://github.com/kostyay/agent-stuff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
