## libg-poc

> Terminal IDE proof-of-concept: Ghostty terminal + Neovim + AI agent, all wired together in a Tauri desktop app. macOS only.

# libg-poc

Terminal IDE proof-of-concept: Ghostty terminal + Neovim + AI agent, all wired together in a Tauri desktop app. macOS only.

## Architecture

Three subsystems communicate through Tauri's IPC and event bus:

```
Ghostty (terminal) ←→ Neovim (editor) ←→ ACP Agent (AI)
        ↕                    ↕                   ↕
    ghostty_embed.rs     nvim_bridge.rs      acp_client.rs
        ↕                    ↕                   ↕
                    Tauri event bus
                         ↕
                   React frontend
```

**Ghostty** renders natively via NSView above the webview — not inside it. The terminal is a real macOS view, not a canvas or xterm.js.

**Neovim** connects via Unix socket RPC (`nvim-rs`). The connection is bidirectional: Tauri calls Neovim APIs (get context, apply edits), and Neovim sends actions back via `vim.rpcnotify()`.

**ACP Agent** (`codex-acp`) runs as a subprocess with stdio pipes. The connection is `!Send`, so it lives on a dedicated OS thread with its own tokio LocalSet. Communication with the main Tauri thread happens through mpsc channels.

## Build

Requires: Zig 0.13.0, Xcode CLI tools, Homebrew freetype, Node.js, Rust.

```bash
just setup-libghostty  # one-time: downloads Zig, clones Ghostty, builds libghostty.dylib
cd tauri-app && npm install
just tauri-dev         # runs the app
```

**Critical**: Every `cargo build`/`cargo check`/`tauri dev` command needs `GHOSTTY_LOCATION=.tools/libghostty`. The Justfile and package.json scripts handle this automatically. If building manually:

```bash
GHOSTTY_LOCATION=$PWD/.tools/libghostty cargo check -p tauri-app
```

The `mcp-debug` feature (`just tauri-dev` with `--features mcp-debug`, or `npm run tauri:debug`) enables the Tauri MCP bridge plugin for external tool automation.

## Key Files

### Rust backend (`tauri-app/src-tauri/src/`)

| File | Role |
|------|------|
| `lib.rs` | Tauri app entry point, registers all commands, manages state. Uses `builder.build()` + `app.run()` to hook `RunEvent::Exit` for socket cleanup. |
| `ghostty_embed.rs` | Embeds Ghostty terminal as NSView above webview. Handles coordinate conversion (JS viewport → AppKit window → view-local), HiDPI scaling, keyboard/mouse forwarding, 60Hz tick loop. Thread-local `GhosttyManager` holds all instances. |
| `nvim_bridge.rs` | Neovim RPC bridge. `NvimHandler` receives `libg_action` notifications and emits Tauri events. `nvim_connect` injects Lua keybindings via `exec_lua`. `build_lua_setup()` generates the Lua code. |
| `acp_client.rs` | ACP agent lifecycle. Spawns subprocess, runs `!Send` connection on dedicated thread, streams events to frontend via Tauri events (`acp-event`). |
| `socket_manager.rs` | Neovim socket lifecycle. PID-scoped paths (`/tmp/libg-nvim-{pid}-{terminalId}.sock`), stale cleanup on startup via `libc::kill`, cleanup on exit via `RunEvent::Exit` and `Drop`. |

### React frontend (`tauri-app/src/`)

| File | Role |
|------|------|
| `App.tsx` | Shell layout: resizable sidebar (explorer/AI) + terminal panel. Auto-switches to AI panel on `nvim-action` events. Sidebar width and active panel persisted to localStorage. |
| `components/Ghostty.tsx` | React wrapper for native Ghostty view. Manages lifecycle (create/update/destroy), ResizeObserver, focus. |
| `components/AiChat/AiChat.tsx` | Chat UI with connection flow, message rendering, auto-apply toggle. Gets socket path from Rust via `invoke("get_socket_path")`. |
| `hooks/useLocalStorage.ts` | Generic hook wrapping `useState` with synchronous localStorage read on init and `useEffect` write on change. Foundation for all UI persistence. |
| `hooks/useAiChat.ts` | Orchestrates Neovim + ACP. Listens for `nvim-action` events, builds prompts from actions, manages streaming messages, auto-apply logic. Messages and autoApply persisted to localStorage (trimmed to 200 on mount). |
| `hooks/useNvimBridge.ts` | Neovim connection hook. Polls context + diagnostics every 2s. |
| `hooks/useAcpAgent.ts` | ACP agent hook. Manages lifecycle, listens to `acp-event` Tauri events. |
| `hooks/useProjectExplorer.ts` | Project/folder state with deterministic path-based IDs (`stableId()`). Projects and active folder persisted to localStorage. Deduplicates on add. |
| `hooks/useTerminalManager.ts` | Terminal state. Lazy creation: terminals are only created when a folder is first selected. Ephemeral (not persisted). |
| `types/nvim.ts` | TypeScript types mirroring Rust structs. Includes `NvimAction` discriminated union for the 4 action types. |

### Config / Build

| File | Role |
|------|------|
| `Justfile` | Build recipes. `setup-libghostty` is the critical one-time setup. |
| `tauri-app/src-tauri/build.rs` | Copies libghostty.dylib, adds rpath, links macOS frameworks. Generates MCP capability file when `mcp-debug` feature is on. |
| `tauri-app/src-tauri/Cargo.toml` | `mcp-debug` feature gates `tauri-plugin-mcp-bridge`. `rmpv` has `with-serde` for msgpack deserialization. `libc` used by `socket_manager.rs` for PID liveness checks. |
| `AGENTS.md` | Commit message conventions (Conventional Commits). |

## Non-Obvious Knowledge

### Ghostty is not in the webview

Ghostty renders as a native NSView *above* the Tauri webview. The React `<Ghostty>` component is just an invisible div that provides positioning — `getBoundingClientRect()` coordinates are sent to Rust, which positions the real NSView. This means:

- You cannot use CSS to style the terminal content
- Z-ordering is native (NSView ordering), not CSS z-index
- Coordinate conversion between web viewport and AppKit is complex and a frequent source of bugs (see `rect_to_frame` in ghostty_embed.rs)

### GhosttyManager is thread-local

`GhosttyManager` lives in a `thread_local! { RefCell }`. All ghostty commands in `lib.rs` use `window.run_on_main_thread()` to access it. This is because AppKit requires main-thread access for views.

### ACP connection is !Send

The `agent-client-protocol` crate's `ClientSideConnection` is `!Send`. It runs on a dedicated OS thread with `tokio::task::LocalSet`. The main Tauri async commands communicate with it via `mpsc::Sender<AcpCommand>` + `oneshot::Sender` for replies. Don't try to hold the connection across `.await` points on the Tauri command thread.

### Neovim Lua injection uses doubled braces

`build_lua_setup()` uses Rust's `format!()` with a raw string. Lua tables use `{}` which conflicts with format string placeholders. Every `{` and `}` in the Lua code is doubled (`{{` / `}}`). Only `{channel_id}` is a real format placeholder. This is easy to break when editing the Lua.

### rmpv two-step deserialization

Neovim sends msgpack `Value` types. To convert to Rust structs, we go through two steps: `rmpv::ext::from_value(Value) → serde_json::Value → serde_json::from_value → NvimAction`. This is because `rmpv::ext::from_value` doesn't handle serde's `tag` attribute well for enums. The `with-serde` feature on `rmpv` is required for this.

### Tauri auto-injects special parameters

`tauri::AppHandle`, `tauri::State<'_, T>`, and `tauri::Window` are auto-injected by Tauri into `#[tauri::command]` functions — they don't appear in the frontend `invoke()` call. `nvim_connect` takes `app_handle: tauri::AppHandle` but the frontend only passes `terminal_id` and `socket_path`.

### Neovim socket path convention

Socket paths are PID-scoped: `/tmp/libg-nvim-{pid}-{terminalId}.sock`. The `SocketManager` in Rust owns the lifecycle — `get_socket_path` command generates and registers the path, `RunEvent::Exit` cleans up on close, and `cleanup_stale()` removes sockets from dead processes on startup. `AiChat.tsx` calls `invoke("get_socket_path")` then sends `nvim --listen <path> .` to the terminal via `ghostty_write_text`, waits 1.5s, then connects. The terminal ID ties everything together: Ghostty instance, Neovim connection, and ACP events.

### UI state persistence via localStorage

All persistent UI state uses `useLocalStorage` (a generic hook in `hooks/useLocalStorage.ts`). State is read synchronously on mount (no flash of defaults) and written via `useEffect` on change. Keys are prefixed with `libg:`.

| Key | Type | Default | Owner |
|-----|------|---------|-------|
| `libg:sidebarWidth` | `number` | `260` | `App.tsx` |
| `libg:activePanel` | `"explorer" \| "ai"` | `"explorer"` | `App.tsx` |
| `libg:projects` | `Project[]` | `[]` | `useProjectExplorer` |
| `libg:activeFolderId` | `string \| null` | `null` | `useProjectExplorer` |
| `libg:chatMessages` | `ChatMessage[]` | `[]` | `useAiChat` |
| `libg:autoApply` | `boolean` | `false` | `useAiChat` |

Terminals are **not** persisted — they represent live Ghostty/Neovim sessions. On restart the project tree and chat history restore, but terminals are created lazily when the user selects a folder.

### Project and folder IDs are deterministic

`useProjectExplorer` uses `stableId(prefix, path)` — a simple numeric hash of the filesystem path converted to base-36. Same path always produces the same ID. This means persisted IDs survive restarts, and adding the same path twice is a no-op (deduplicated by ID).

### Neovim context polling

`useNvimBridge` polls Neovim for context (cursor, file, diagnostics) every 2000ms. This is separate from the action trigger flow. When a Neovim action is triggered, it sends its own context snapshot in the RPC payload rather than relying on the polled state.

### Visual mode keybindings exit visual mode first

The Lua visual mode mappings (`<leader>me`, `<leader>ma`) read `'<` / `'>` marks after Neovim exits visual mode. This means `get_visual_selection()` uses `vim.fn.getpos("'<")` which contains the last visual selection boundaries. Character-wise and line-wise selection work; block mode falls back to line-level.

### Diagnostic filter uses 0-indexed line

Neovim's `vim.diagnostic.get(bufnr, { lnum = line - 1 })` expects 0-indexed line numbers, but `vim.api.nvim_win_get_cursor(0)` returns 1-indexed. The Lua code in `fix_diagnostic()` subtracts 1 correctly.

### No test infrastructure

There are no tests. When adding features, verify by running the app (`just tauri-dev`) and testing the integration manually.

### Clipboard is stubbed out

Ghostty clipboard callbacks (`runtime_read_clipboard_cb`, `runtime_write_clipboard_cb`) are empty stubs. Copy/paste in the terminal does not work.

### Debug diagnostics in stderr

`ghostty_embed.rs` has `eprintln!` diagnostic logging in `rect_to_frame` and `event_position_px` (first 10 mouse events). These are intentional debug aids, not mistakes.

## Tauri Event Flow

| Event | Direction | Payload |
|-------|-----------|---------|
| `nvim-action` | Rust → Frontend | `NvimActionEvent { terminalId, action }` |
| `acp-event` | Rust → Frontend | `AcpEvent` (ContentChunk, Done, Error, etc.) |

Frontend listens via `@tauri-apps/api/event`'s `listen()`. Both `App.tsx` (panel switch) and `useAiChat.ts` (prompt dispatch) listen for `nvim-action`.

## Commit Conventions

See `AGENTS.md`. Format: `<type>(<scope>): <imperative summary>`. Types: `feat`, `fix`, `chore`, `debug`. Scopes: `ghostty`, `tauri`, `nvim`, `ui`, `acp`, `mcp`, `repo`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fcoury)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fcoury)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
