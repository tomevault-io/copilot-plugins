## klanker

> | Dev server | `npm run dev` |

# AGENTS.md

## Commands

| Action | Command |
|--------|---------|
| Dev server | `npm run dev` |
| API server (SQLite) | `npm run api` |
| Both together | `npm run dev:full` |
| Build | `npm run build` |
| Run all web tests | `npm test` |
| Watch tests | `npm run test:watch` |
| Run single test file | `npx vitest run src/__tests__/api.test.js` |
| Start SearXNG | `docker start klanker-searxng` |
| Restart SearXNG | `docker restart klanker-searxng` |
| Run voice tests (WSL) | `cd klanker-voice && python3 -m pytest tests/ -v` |
| Voice app (Windows) | `cd klanker-voice && python -m src --debug` |
| Voice app demo mode | `cd klanker-voice && python -m src --demo --debug` |

## Architecture

Two applications sharing a SQLite database:

1. **Klanker Web** — Svelte 5 + Vite chat app with SSE streaming, web search, file/image support, LLM tool framework (shell, read_file, search)
2. **Klanker Voice** — Python voice assistant with wake word, STT, LLM streaming, floating widget

Both talk to the same **LM Studio** instance and **SearXNG** for web search.

### Web App

```
src/
  main.js                  # Mounts App into #app
  App.svelte               # Root layout: sidebar-rail, header, chat, input
  app.css                  # Global styles — Linear design system tokens
  lib/
    api.js                 # streamChat() async generator — SSE streaming client
    store.svelte.js        # createChatStore() — reactive state via Svelte 5 runes
    db.js                  # HTTP client → SQLite API server (was IndexedDB)
    fileParser.js          # Text extraction from file types (PDF, DOCX, XLSX, etc.)
    fileHandler.js         # File processing pipeline (image/document handling)
    search.js              # Web search integration via SearXNG
    tools.js               # Tool directive parser, API client, system prompt builder
  components/
    Chat.svelte            # Scrollable message list with smart auto-scroll + scroll-to-bottom
    Message.svelte         # Message bubble orchestrator (markdown + citation styling)
    MessageAttachments.svelte  # Image and file attachment display
    MessageSources.svelte  # Web search source citation chips
    ThinkingBlock.svelte   # Collapsible reasoning/thought display
    ToolCall.svelte        # Tool call status display with approve/deny buttons
    Input.svelte           # Textarea + send/stop buttons + drag-and-drop
    FileAttachments.svelte # Pending file/image attachment UI
    Sidebar.svelte         # Conversation list layout shell
    ConversationItem.svelte # Single conversation row with context menu + auto-focus rename
    SearchBox.svelte       # Search input with clear button
    ModelSelector.svelte   # Model dropdown with keyboard navigation + auto-focus
  __tests__/
    api.test.js            # streamChat tests with mocked fetch/ReadableStream
    store.test.js          # Store integration tests (28 tests — search, tools, streaming)
    tools.test.js          # Tool parsing + prompt builder tests (23 tests)
server/
  api.js                   # Node.js SQLite API server (better-sqlite3) — 10 REST endpoints
  tools/
    registry.js            # Tool definitions (search, shell, read_file)
    classify.js            # 3-tier command classifier (safe/approval/blocked)
    shell.js               # Shell executor with timeout + truncation
    readFile.js            # File reader with path validation + binary detection
    __tests__/             # 56 server-side unit tests (classify, shell, readFile)
```

### Voice App

```
klanker-voice/
  src/
    __init__.py
    __main__.py            # python -m entry point
    main.py                # App orchestration, system tray, QThread workers, --demo mode
    widget.py              # PyQt6 floating overlay — solid dark panel with animated orb
    db.py                  # Shared SQLite persistence (conversations + messages)
    llm.py                 # LM Studio streaming client (httpx SSE) with search loop
    search.py              # SearXNG web search client (Python port of search.js)
    wake.py                # OpenWakeWord background listener — "Hey Clanker" / Winston / Hey Jarvis
    transcribe.py          # faster-whisper STT with adaptive silence detection
  assets/
    hey_clanker.onnx       # Custom wake word model (trained via Colab)
    winston.onnx           # Fallback wake word — say "Winston"
  train/
    train_on_gpu.bat       # Windows batch script for GPU training on NVIDIA card
    COLAB_GUIDE.md         # Google Colab training instructions
    openwakeword-training/ # CoreWorxLab Docker-based trainer (CPU Dockerfile patched)
  tests/
    test_db.py             # 15 tests — SQLite CRUD
    test_llm.py            # 4 tests — SSE streaming, split chunks, system prompt
    test_main.py           # 5 tests — dismiss phrase detection (EN + RO)
  requirements.txt
  pyproject.toml
  README.md
```

### Shared SQLite Database

Both apps read/write the same SQLite file:
- **Windows**: `%APPDATA%\klanker\conversations.db`
- **Linux/WSL**: `~/.local/share/klanker/conversations.db`
- Override: `KLANKER_DB_PATH` env var

The web app talks to SQLite via the API server (`server/api.js` on port 3100, proxied via Vite `/api`). The voice app writes directly via Python `sqlite3`.

**Data flow (web):** `Input` → `App.send()` → `store.send()` → `streamChat()` yields tokens → store mutates `$state` → `Chat`/`Message` re-render reactively.

**Data flow (voice):** Wake word → Record audio → Whisper STT → LLM stream (with search loop) → Widget displays response → Auto-listen for follow-up or dismiss.

**Tool flow (web):** Model outputs `[TOOL: name {params}]` → parsed by `tools.js` → pre-check classification → approval if needed → execute → result injected as conversation turn → model re-prompted. Up to 5 rounds per message. Tool results accumulate as assistant/user turn pairs so the model sees its full history.

**Search flow (both):** Model outputs `[TOOL: search {"query": "..."}]` or `[SEARCH: query]` → detected in stream → SearXNG query → results injected → model re-prompted → streams answer with citations.

## Voice App Details

### Wake Word Priority
1. `assets/hey_clanker.onnx` — custom trained model, say **"Hey Clanker"**
2. `assets/winston.onnx` — community model, say **"Winston"**
3. Built-in `hey_jarvis` — downloaded from OpenWakeWord, say **"Hey Jarvis"**

### Voice Flow
1. Wake word detected → widget appears (animated orb)
2. Wake listener STOPS to release mic → 300ms delay
3. Recording starts → adaptive noise calibration (first 0.5s)
4. Speech detected → records until 1.5s silence
5. Whisper tiny (CPU, int8, beam=1) transcribes in ~0.3s
6. If dismiss phrase ("thanks", "mersi", "mulțumesc") → close widget, resume wake listener
7. Otherwise → send to LLM with search-capable system prompt
8. LLM may emit `[SEARCH: query]` → SearXNG search → re-prompt with results
9. Response streams into widget panel
10. After response → auto-listen for follow-up (no wake word needed)
11. If no speech detected → dismiss and resume wake listener

### Widget States
| State | Orb Color | Animation | Panel |
|-------|-----------|-----------|-------|
| LISTENING | Indigo | Audio bars + breathing glow | Hidden |
| THINKING | Bright indigo | Orbiting arcs + bouncing dots | Hidden |
| RESPONDING | Green | Gentle pulse | Visible, streaming text |
| DONE | Green | Checkmark | Visible, hint shown |
| ERROR | Red | X mark | Visible, error text |

### Key Technical Decisions
- **Runs on Windows, developed in WSL** — all Python source is cross-platform
- **PyQt6** for GUI — solid dark panel, no `WA_TranslucentBackground` (causes ghost windows on Windows)
- **faster-whisper** instead of pywhispercpp — pre-built wheels, no CMake needed
- **Tiny model + int8 + beam_size=1** — optimized for speed on CPU (9800X3D)
- **QThread + Qt signals** for all cross-thread communication — no `QTimer.singleShot` from bg threads
- **httpx** async SSE streaming with proper `break` instead of `return` to avoid dangling coroutines
- **Adaptive silence detection** — calibrates noise floor from first 0.5s, threshold = max(3x noise, 0.005)

## Key Conventions

- **No TypeScript** — plain JS with JSDoc comments for type hints.
- **Svelte 5 runes** (`$state`, `$effect`, `$props`) — no legacy `$:` reactivity or stores.
- Store file uses `.svelte.js` extension so Vite compiles runes outside `.svelte` files.
- **No UI libraries** — all styling is plain CSS with scoped `<style>` blocks per component.
- All colors/spacing use CSS custom properties defined in `app.css`.
- **Linear design system** — muted, semi-transparent surfaces. No loud accent fills on inline elements. Use `--bg-tertiary` + `--border-light` for badges/chips, not `--accent`.
- Components target < 200 lines each.
- All API logic isolated in `src/lib/api.js`; components never call `fetch` directly.
- **Python voice app** — plain Python, no type stubs, Roboto font, Pytest for tests.

## Services

| Service | URL | Config |
|---------|-----|--------|
| LM Studio API | `http://10.3.58.20:1234/v1` | Proxied via Vite `/v1` |
| SearXNG | `http://localhost:8888` | Docker `klanker-searxng`, proxied via Vite `/search` |
| SQLite API | `http://localhost:3100` | `server/api.js`, proxied via Vite `/api` |

SearXNG config is volume-mounted from `./searxng/settings.yml`. The `formats` list MUST include `json` — without it, the `/search?format=json` endpoint returns 403.

## Environment Variables

### Web App
Configured in `.env`, prefixed with `VITE_` for client-side access:

| Variable | Default | Purpose |
|----------|---------|---------|
| `VITE_API_BASE` | `http://10.3.58.20:1234/v1` | LM Studio API base URL |

### Voice App

| Variable | Default | Purpose |
|----------|---------|---------|
| `KLANKER_API_BASE` | `http://10.3.58.20:1234/v1` | LM Studio API URL |
| `KLANKER_DB_PATH` | Platform-specific (see above) | SQLite database path |
| `KLANKER_SEARCH_URL` | `http://localhost:8888/search` | SearXNG endpoint |

## Testing

### Web (Vitest)
- `api.test.js` mocks `fetch` with `vi.stubGlobal` and constructs `ReadableStream` to simulate SSE chunks.
- `store.test.js` mocks `../lib/api.js` module to isolate store logic from network. Includes search flow, tool execution, blocked tools, thinking-as-content, multi-round search, source dedup, and buffered streaming.
- `tools.test.js` — 23 tests: parseTool directive extraction, incomplete directives, buildToolPrompt generation.
- Svelte plugin processes `.svelte.js` files during test runs, so runes work in tests.

### Voice (Pytest)
- `test_db.py` — 15 tests: SQLite CRUD for conversations and messages, JSON fields, cascading deletes.
- `test_llm.py` — 4 tests: SSE streaming with mocked httpx, split chunks, system prompt injection.
- `test_main.py` — 5 tests: dismiss phrase detection in English and Romanian.
- No PyQt6 needed for tests — all testable logic is isolated from GUI.

## Gotchas

- The store file **must** be `.svelte.js` (not `.js`) — `$state` and other runes are compile-time transforms that only activate for `.svelte` and `.svelte.js` files.
- SSE parsing in `api.js` handles chunks split across `ReadableStream` reads by buffering incomplete lines — don't assume one read = one SSE event.
- `streamChat` is an **async generator** — consumers must use `for await...of` and handle `AbortError` for cancellation.
- The store sends the full conversation history (including system prompt) on every request — there is no server-side session.
- **Tool buffering**: `streamResponse` buffers content to detect `[TOOL:]` and `[SEARCH:]` directives before writing to reactive state. Some models (Qwen/Gemma) dump reasoning as content tokens — the tool loop extracts text before directives and moves it to `msg.reasoning`.
- **SearXNG JSON format**: Must be enabled in `searxng/settings.yml` under `search.formats`. Without it, all JSON API calls return 403.
- **System prompt is in `store.svelte.js`** (web) and `llm.py` (voice): The web prompt is `BASE_SYSTEM_PROMPT` + dynamic tool section from `/api/tools`. The voice prompt has search instructions only. Keep behavior sections in sync when tuning.
- **Voice: wake listener must stop before recording** — both use the same mic. The 300ms delay after stopping ensures the device is released.
- **Voice: `WA_TranslucentBackground` breaks on Windows** — causes ghost window artifacts. Use solid dark background with `border-radius` instead.
- **Voice: `QTimer.singleShot` from background threads is unreliable on Windows** — always use QThread + Qt signals for cross-thread communication.
- **Voice: httpx async generators must use `break` not `return`** — `return` inside `async for` abandons the stream without cleanup, causing "Task was destroyed" warnings.
- **Voice: Python 3.14 is too new** — `faster-whisper` and other deps don't have wheels. Use Python 3.12 on Windows.
- **Web app db.js was migrated from IndexedDB to HTTP API** — now calls `server/api.js` REST endpoints. The `idb` package has been removed.
- **Tool framework classifier order matters** — the `||` chain check must run before the `|` pipe check in `classify.js`, otherwise `||` gets split on single `|` and produces empty segments.

## Pending / TODO

- **Wake word training** — `hey_clanker.onnx` was trained via Google Colab. For re-training or improvements, use `train/train_on_gpu.bat` on an NVIDIA GPU machine, or the Docker trainer in `train/openwakeword-training/`.

## Dev Server

Always keep the dev server running with hot reload during development:

```bash
npm run dev
```

For full stack with SQLite API:
```bash
npm run dev:full
```

Vite HMR auto-updates the browser on file changes. If the port is occupied, kill the old process first:

```bash
kill $(lsof -ti :5173) 2>/dev/null; npm run dev
```

## MemPalace — Persistent Memory

This project has a MemPalace wing named **klanker** (275+ drawers across rooms: src, general, searxng, voice).

### On Session Start

- Run `mempalace_search` or `mempalace_kg_query` before answering questions about past decisions, architecture choices, or project history.
- Use `mempalace_diary_read({agent_name: "crush"})` to recall what happened in previous sessions.

### During Work

- When you discover something important (a decision, a gotcha, a pattern), file it with `mempalace_add_drawer({wing: "klanker", room: "<appropriate_room>", content: "<verbatim content>"})`.
- When facts change (e.g., a dependency is swapped, an API endpoint moves), update the knowledge graph with `mempalace_kg_invalidate` + `mempalace_kg_add`.

### On Session End

- Write a diary entry with `mempalace_diary_write({agent_name: "crush", entry: "<AAAK compressed summary>"})` summarizing what you worked on and what matters.

### Rooms

| Room | Contents |
|------|----------|
| `src` | Source code fragments, store logic, API layer, components |
| `general` | Config, docs, specs, project-level files |
| `searxng` | SearXNG search engine configuration |
| `voice` | Voice assistant decisions, wake word, STT, widget |

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **klanker** (113 symbols, 200 relationships, 10 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## When Debugging

1. `gitnexus_query({query: "<error or symptom>"})` — find execution flows related to the issue
2. `gitnexus_context({name: "<suspect function>"})` — see all callers, callees, and process participation
3. `READ gitnexus://repo/klanker/process/{processName}` — trace the full execution flow step by step
4. For regressions: `gitnexus_detect_changes({scope: "compare", base_ref: "main"})` — see what your branch changed

## When Refactoring

- **Renaming**: MUST use `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` first. Review the preview — graph edits are safe, text_search edits need manual review. Then run with `dry_run: false`.
- **Extracting/Splitting**: MUST run `gitnexus_context({name: "target"})` to see all incoming/outgoing refs, then `gitnexus_impact({target: "target", direction: "upstream"})` to find all external callers before moving code.
- After any refactor: run `gitnexus_detect_changes({scope: "all"})` to verify only expected files changed.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Tools Quick Reference

| Tool | When to use | Command |
|------|-------------|---------|
| `query` | Find code by concept | `gitnexus_query({query: "auth validation"})` |
| `context` | 360-degree view of one symbol | `gitnexus_context({name: "validateUser"})` |
| `impact` | Blast radius before editing | `gitnexus_impact({target: "X", direction: "upstream"})` |
| `detect_changes` | Pre-commit scope check | `gitnexus_detect_changes({scope: "staged"})` |
| `rename` | Safe multi-file rename | `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` |
| `cypher` | Custom graph queries | `gitnexus_cypher({query: "MATCH ..."})` |

## Impact Risk Levels

| Depth | Meaning | Action |
|-------|---------|--------|
| d=1 | WILL BREAK — direct callers/importers | MUST update these |
| d=2 | LIKELY AFFECTED — indirect deps | Should test |
| d=3 | MAY NEED TESTING — transitive | Test if critical path |

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/klanker/context` | Codebase overview, check index freshness |
| `gitnexus://repo/klanker/clusters` | All functional areas |
| `gitnexus://repo/klanker/processes` | All execution flows |
| `gitnexus://repo/klanker/process/{name}` | Step-by-step execution trace |

## Self-Check Before Finishing

Before completing any code modification task, verify:
1. `gitnexus_impact` was run for all modified symbols
2. No HIGH/CRITICAL risk warnings were ignored
3. `gitnexus_detect_changes()` confirms changes match expected scope
4. All d=1 (WILL BREAK) dependents were updated

## Keeping the Index Fresh

After committing code changes, the GitNexus index becomes stale. Re-run analyze to update it:

```bash
npx gitnexus analyze
```

If the index previously included embeddings, preserve them by adding `--embeddings`:

```bash
npx gitnexus analyze --embeddings
```

To check whether embeddings exist, inspect `.gitnexus/meta.json` — the `stats.embeddings` field shows the count (0 means no embeddings). **Running analyze without `--embeddings` will delete any previously generated embeddings.**

> Claude Code users: A PostToolUse hook handles this automatically after `git commit` and `git merge`.

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StefanAlexandru-V) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
