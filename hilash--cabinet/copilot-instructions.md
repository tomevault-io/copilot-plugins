## cabinet

> Cabinet is an AI-first self-hosted knowledge base and startup OS. All content lives as markdown files on disk. The web UI provides WYSIWYG editing, a collapsible tree sidebar, drag-and-drop page organization, structured AI runs for tasks/jobs/heartbeats, and interactive `WebTerminal` surfaces for direct CLI sessions.

# CLAUDE.md — Cabinet

## What is this project?

Cabinet is an AI-first self-hosted knowledge base and startup OS. All content lives as markdown files on disk. The web UI provides WYSIWYG editing, a collapsible tree sidebar, drag-and-drop page organization, structured AI runs for tasks/jobs/heartbeats, and interactive `WebTerminal` surfaces for direct CLI sessions.

**Core philosophy:** Humans define intent. Agents do the work. The knowledge base is the shared memory between both.

## Tech Stack

- **Framework:** Next.js 16 (App Router), TypeScript
- **UI:** Tailwind CSS + shadcn/ui (base-ui based, NOT Radix — no `asChild` prop)
- **Editor:** Tiptap (ProseMirror-based) with markdown roundtrip via HTML intermediate
- **State:** Zustand (tree-store, editor-store, ai-panel-store, app-store)
- **Fonts:** Inter (sans) + JetBrains Mono (code)
- **Icons:** Lucide (no emoji in system chrome)
- **Markdown:** gray-matter (frontmatter), unified/remark (MD→HTML), turndown (HTML→MD)
- **AI:** Claude Code and Codex CLI via the adapter runtime; `WebTerminal` stays in the product for interactive sessions

## Architecture

```
src/
  app/api/tree/              → GET tree structure from /data
  app/api/pages/[...path]/   → GET/PUT/POST/DELETE/PATCH pages
  app/api/upload/[...path]/  → POST file upload to page directory
  app/api/assets/[...path]/  → GET/PUT static file serving + raw file writes
  app/api/search/            → GET full-text search
  app/api/agents/conversations/ → Manual task/conversation creation + listing
  app/api/agents/providers/  → Provider, model, adapter metadata
  app/api/agents/tasks/      → Task board data
  app/api/agents/scheduler/  → Scheduler control/status
  app/api/git/               → Git log, diff, commit endpoints
  stores/                    → Zustand (tree, editor, ai-panel, task, app)
  components/sidebar/        → Tree navigation, drag-and-drop, context menu
  components/editor/         → Tiptap WYSIWYG + toolbar, website/PDF/CSV viewers
  components/ai-panel/       → Right-side AI chat panel
  components/tasks/          → Task board + task detail panel
  components/agents/         → Agents workspace + live/result conversation views
  components/jobs/           → Jobs manager UI
  components/terminal/       → xterm.js web terminal
  components/composer/       → Shared composer + task runtime picker
  components/search/         → Cmd+K search dialog
  components/layout/         → App shell, header
  lib/storage/               → Filesystem ops (path-utils, page-io, tree-builder, task-io)
  lib/markdown/              → MD↔HTML conversion
  lib/git/                   → Git service (auto-commit, history, diff)
  lib/agents/                → Adapter runtime, conversation runner, personas, providers
  lib/jobs/                  → Job scheduler (node-cron)
server/
  cabinet-daemon.ts          → Unified daemon for structured runs, PTY sessions, scheduler, events
  terminal-server.ts         → Standalone PTY WebSocket server kept for focused terminal debugging/legacy use
data/                        → Content directory (KB pages, tasks, jobs)
```

## Key Rules

1. **No database** — everything is files on disk under `/data`
2. **Pages** are directories with `index.md` + assets, or standalone `.md` files. PDFs and CSVs are also first-class content types.
3. **Frontmatter** (YAML) stores metadata: title, created, modified, tags, icon, order
4. **Path traversal prevention** — all resolved paths must start with DATA_DIR
5. **shadcn/ui uses base-ui** (not Radix) — DialogTrigger, ContextMenuTrigger etc. do NOT have `asChild`
6. **Dark mode default** — theme toggle available, use `next-themes` with `attribute="class"`
7. **Auto-save** — debounced 500ms after last keystroke in editor-store
8. **AI runs use a mixed runtime model** — tasks/jobs/heartbeats default to structured adapters; `WebTerminal` remains for interactive sessions and experimental legacy PTY flows.
9. **Do not assume the terminal is being removed** — the product direction is away from terminal-first task execution, while keeping terminal functionality for direct sessions and future features such as Cabinet-managed tmux-like workflows.
10. **Version restore** — users can restore any page to a previous git commit via the Version History panel
11. **Embedded apps** — dirs with `index.html` + no `index.md` render as iframes. Add `.app` marker for full-screen mode (sidebar + AI panel auto-collapse)
12. **Linked repos** — `.repo.yaml` in a data dir links it to a Git repo (local path + remote URL). Agents use this to read/search source code in context. See `data/CLAUDE.md` for full spec.

## AI Editing Behavior (CRITICAL)

When Cabinet starts an AI edit or task run:

1. **The request becomes a conversation** with `providerId`, `adapterType`, and optional adapter config such as model or effort.
2. **Detached runs** go through `/api/agents/conversations` → `conversation-runner` → `cabinet-daemon`.
3. **Structured adapters are the default** for detached Claude/Codex runs; legacy PTY adapters remain available as experimental escape hatches.
4. **Interactive editor/live surfaces may still mount `WebTerminal`** when terminal feedback matters.
5. **Models should edit targeted files directly when useful** and reflect durable value in KB files, not only transcript text.
6. **If content gets corrupted** — users can restore from Version History (clock icon → select commit → Restore)

The AI panel supports `@` mentions — users type `@PageName` to attach other pages as context. The mentioned pages' content is fetched and appended to the prompt so Claude has full context.


## Commands

```bash
npm run dev          # Start Next.js dev server on localhost:3000
npm run dev:daemon   # Start unified daemon on localhost:3001
npm run dev:terminal # Start standalone terminal WebSocket server on localhost:3001 (focused PTY/legacy debugging)
npm run dev:all      # Start both servers
npm run debug:chrome # Launch Chrome with CDP on localhost:9222 for frontend debugging
npm run build        # Production build
npm run lint         # ESLint
```

## Frontend Debugging

Use `npm run debug:chrome` when you need a debuggable browser session. It launches Chrome or Chromium with `--remote-debugging-port=9222`, opens Cabinet at `http://localhost:3000` by default, and prints the DevTools endpoints:

- `http://127.0.0.1:9222/json/version`
- `http://127.0.0.1:9222/json/list`

This makes it possible to attach over CDP and inspect real DOM, network, and screenshots instead of guessing at frontend state.

## Progress Tracking

After every change you make to this project, append an entry to `PROGRESS.md` using this format:

```
[YYYY-MM-DD] Brief description of what changed in 1-3 sentences.
```

This is mandatory. Do not skip it. The PROGRESS.md file is the changelog for this project.

---
> Source: [hilash/cabinet](https://github.com/hilash/cabinet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
