## imhottep

> ImHoTTeP is an HTTP(S) TUI client — send HTTP(S) requests, inspect responses, edit headers/body interactively.

# imhottep: agent instructions

ImHoTTeP is an HTTP(S) TUI client — send HTTP(S) requests, inspect responses, edit headers/body interactively.

These are the agent instructions for the ImHoTTeP project. If your changes make them obsolete or need to be reflected here, edit them at AGENTS.md as part of your session.


## Build / Run / Test

```sh
cargo build
cargo run
cargo test
```

## Coverage

```sh
# Generate (requires cargo-tarpaulin)
# tarpaulin.toml excludes main.rs and ui.rs (TUI entry & rendering — not unit-testable)
cargo tarpaulin
```

## Git usage

You should never commit anything or change the git state in any other way unless the user explicitly asked you to. Otherwise, you may view Git history in any way you need to.

## Architecture

| File | Purpose |
|------|---------|
| `src/main.rs` | Entry point, event loop, `execute_action()` dispatcher |
| `src/app.rs` | `App` struct — all state (buffers, cursors, scroll, requests, config, networking accumulators) |
| `src/http_client.rs` | `HttpClient`, `HttpRuntime`, `HttpRequest`, `HttpResponse`, `RequestEvent` — networking layer (hyper 1.x + hyper-tls) |
| `src/ui.rs` | Ratatui rendering — layout, cursor positioning, scrollable paragraphs |
| `src/keymap.rs` | `Keymap` — context-aware keybinding resolution (`KeyTrigger`), action enum |

## Layout (Request screen)

```
┌─ Method URL ──────────────────────────────────────┐
│ GET https://api.example.com                       │
├─ Headers ─────────────────────────────────────────┤
│ Content-Type: application/json                    │
│ Authorization: Bearer ...                         │
├─ Body ────────────────────────────────────────────┤
│ {"key": "value"}                                  │
├─ Request Events ───────┬─ Response ───────────────┤
│ 2025-01-01 12:00:00    │ 200 OK                   │
│ Sent request           │ {"status": "ok"}         │
└────────────────────────┴──────────────────────────┘
```

## Editing

Six editable fields: `Url`, `Headers`, `Body`, `JsonFilter`, `StreamPrefixRegex`, `StreamSuffixRegex`.

Each has its own cursor position and scroll state:
- `input_buffer` / `cursor_pos` — shared by Url, Body, JsonFilter, prefix, suffix
- `header_key_buffer` / `header_key_cursor` — Headers key field
- `header_value_buffer` / `header_value_cursor` — Headers value field

Scroll state:
- `url_h_scroll` — horizontal scroll for Url
- `filter_h_scroll` — horizontal scroll for JsonFilter, prefix, suffix
- `body_scroll` — vertical scroll for Body
- `headers_scroll` — vertical scroll for Headers

### Cursor column (wrapped text)

Use `wrapped_cursor_column(text, max_width)` for the visual column within the last wrapped line of `text`. It tokenizes by whitespace and simulates ratatui word-wrap to find the cursor's position. Handles `\n` boundaries by resetting column per logical line.

Ratatui strips trailing newlines via `str::lines()`. When `body_prefix` ends with `\n`, add 1 to `line_count` and set `col = 0`.

## Word navigation (CursorWordLeft / CursorWordRight / DeleteWordBackward / DeleteWordForward)

Words are defined by **alphanumeric category boundaries** — the cursor jumps/stops between blocks of alphanumeric characters and blocks of non-alphanumeric characters.

Examples:
- `https://api.example.com/v1/users` traverses as: `https` → `://` → `api` → `.` → `example` → `.` → `com` → `/` → `v1` → `/` → `users`
- `hello, world!` traverses as: `hello` → `,` → ` ` → `world` → `!`

Implementation: detect `chars[i].is_alphanumeric()` on the first non-whitespace char, then skip while `is_alphanumeric() == is_alnum && !is_whitespace()`.

## Keymap

`KeyTrigger` variants:
- `Char(c)` — matches `KeyCode::Char(c)` with `modifiers == NONE`
- `Code(code)` — matches `KeyCode` with `modifiers == NONE`
- `Modified(mods, code)` — matches `KeyCode` with `modifiers.contains(mods)`

Resolution: collect all matching `ContextRule`s sorted by specificity descending, iterate bindings then triggers, return first match.

**Important**: `KeyTrigger::Code` **must** require `modifiers == NONE` — otherwise it intercepts modified events (e.g. `Code(Left)` would catch Ctrl+Left).

### Modifier fallbacks

- Ctrl+Arrow may not work on all terminals → add Alt+Arrow as fallback
- Ctrl+Backspace may decode as `Char('h')` with CONTROL (0x08 = Ctrl+H) → add `Modified(CONTROL, Char('h'))` as trigger
- Alt+Backspace for word-delete fallback
- Ctrl+Delete / Alt+Delete for word-delete-forward

### Key routing (event loop)

In editing mode: chars with NONE/SHIFT modifiers are routed directly to the input buffer (insert at cursor). All other events (Ctrl+key, arrows, function keys, etc.) fall through to `keymap.resolve()`.

## Networking / HTTP

### HttpClient (`src/http_client.rs`)

- Wraps `hyper::Client` with TLS via `hyper-tls` (native-tls — OpenSSL on Linux).
- `HttpClient::execute(request, event_tx)` sends a single request and streams the response:
  1. Emits lifecycle `RequestEvent`s through an `mpsc::UnboundedSender`: `Started`, `ResolvingHost`, `HostResolved`, `Connecting`, `TlsHandshakeStarted`/`TlsHandshakeComplete` (if `https`), `SendingRequest`, `RequestSent`, `WaitingForResponse`, `ReceivingHeaders`, `HeadersReceived(status)`, `BodyChunk(data)`, `Completed(total_bytes)`, or `Failed(error)`.
  2. Response frames are read via `response.frame().await` — each data frame is lossily converted to `String` and emitted as `BodyChunk`.
  3. The full body is accumulated in `HttpResponse::body` for use by Text/Json view modes.

### HttpRuntime (`src/http_client.rs`)

- Owns a long-lived `tokio::runtime::Runtime` and a cloned `HttpClient`.
- `execute_request(request)` spawns an async task to run `HttpClient::execute`, returns `(oneshot::Receiver<Result<HttpResponse, HttpError>>, mpsc::UnboundedReceiver<RequestEvent>)`.
- The oneshot carries the final response; the mpsc carries streaming events.

### HttpRequest / HttpResponse (`src/http_client.rs`)

- `HttpRequest` fields: `method: HttpMethod`, `url: String`, `headers: HashMap<String, String>`, `body: Option<String>`, `jq_filter: String`, `stream_prefix_regex: String`, `stream_suffix_regex: String`.
- `HttpResponse` fields: `status_code: u16`, `status_text: String`, `headers: HashMap<String, String>`, `body: String`.
- Both derive `Serialize`/`Deserialize` for persistence.

### Request flow (data path)

```
User presses "s" -> execute_action(SendRequest)
  -> App::send_current_request()
    -> HttpRuntime::execute_request(request)
      -> spawns async task on Tokio runtime
        -> HttpClient::execute(request, event_tx)
          -> builds hyper::Request from HttpRequest fields
          -> sends via hyper (TLS via hyper-tls)
          -> streams response body frame-by-frame
          -> sends BodyChunk events through mpsc channel
          -> returns final HttpResponse through oneshot channel
    -> stores (result_rx, event_rx) in App fields
```

## Async event loop (`main.rs`)

Every iteration:
1. `app.check_pending_response()` — tries `oneshot::try_recv()`; on success stores in `last_response_per`, on `Closed` stores error.
2. `app.check_for_events()` — drains `event_receiver` (HTTP events) and `streamed_jq_output_rx` (jq output). BodyChunks accumulate in `streamed_body_per` and flow through `process_chunk()` (line reassembly -> prefix/suffix stripping -> jq feeding).
3. `terminal.draw(ui)` — renders response based on view mode.
4. If async work is in-flight (`pending_response`, `event_receiver`, or `streamed_jq_output_rx` are `Some`), uses non-blocking event poll (50ms timeout); otherwise blocks on key events.

## Request event lifecycle

```
Started
  -> ResolvingHost(host) -> HostResolved
    -> Connecting
      -> TlsHandshakeStarted (if https) -> TlsHandshakeComplete
        -> SendingRequest -> RequestSent
          -> WaitingForResponse -> ReceivingHeaders -> HeadersReceived(status)
            -> BodyChunk(data)* -> Completed(total_bytes)
              OR TemporaryConnectionProblem(msg) [non-terminal]
              OR Failed(error) [terminal]
```

## Response view modes

| Mode | Description |
|------|-------------|
| `Text` | Raw response body, scrollable |
| `Json` | Full body piped through external `jq --color-output <filter>` (one-shot), rendered with ANSI colour support |
| `StreamedJson` | Live-streamed SSE/chunked response: lines reassembled, prefix/suffix stripped via regex, fed line-by-line to persistent `jq` subprocess for real-time filtering |

### StreamedJson details

1. **Line reassembly** in `process_chunk()`: Each `BodyChunk` is appended to a per-request `streamed_line_buffer`, split on `\n`, complete lines processed, incomplete tail preserved.
2. **Prefix/suffix stripping** in `strip_line()`: `stream_prefix_regex` (default `r"^\w+:\s*"`) strips SSE prefixes (e.g. `data: `); `stream_suffix_regex` (default `r"\s*$"`) strips trailing whitespace.
3. **Persistent jq subprocess** in `ensure_jq_running()`: `jq --color-output --unbuffered <filter>` with piped stdin/stdout/stderr. Two reader threads emit `JqEvent::Output`/`JqEvent::Error`.
4. **JSON validation**: Each stripped line checked with `serde_json::from_str` — gates StreamedJson mode availability.
5. **Reprocessing** (`reprocess_streamed_jq()`): When filter/prefix/suffix regex changes, kills jq, re-strips all raw lines from the complete body, runs jq synchronously per line.

## Per-request state tracking

`App` stores multiple `HashMap<usize, ...>` keyed by request index for state that persists across response view switches:

| Field | Type | Purpose |
|-------|------|---------|
| `request_events_per` | `HashMap<usize, Vec<String>>` | Log of lifecycle events |
| `streamed_body_per` | `HashMap<usize, String>` | Accumulated raw body from BodyChunks |
| `last_response_per` | `HashMap<usize, Result<HttpResponse, String>>` | Final response (or error) |
| `streamed_jq_output_per` | `HashMap<usize, Vec<String>>` | jq-filtered output lines |
| `streamed_stripped_lines_per` | `HashMap<usize, Vec<String>>` | Stripped lines for reprocessing |
| `streamed_jq_last_fed_per` | `HashMap<usize, String>` | Last line fed to jq (for error annotation) |
| `streamed_line_buffer_per` | `HashMap<usize, String>` | Incomplete line buffer between chunks |

## Screen state machine

Three screens managed by `App::current_screen: CurrentScreen`:

| Screen | Entry | Exit actions |
|--------|-------|--------------|
| `Main` | App start, `GoBack` from Request | `EditRequest` → `Request`, `TriggerExit` → `Exiting` |
| `Request` | `EditRequest` from Main | `GoBack` → `Main`, `TriggerExit` → `Exiting` |
| `Exiting` | `TriggerExit` from Main/Request | `ConfirmExit` → terminate, `CancelExit` → `Main` |

`TriggerExit` from the Request screen requires Ctrl+C; `q`/`Esc`/`Backspace` trigger `GoBack` instead.

## Focus cycling

Order: `Url → Headers → Body → Response → RequestEvents → Url`

| Field | Focus behavior | Editing behavior |
|-------|---------------|------------------|
| `Url` | `e`/`Enter` to edit; `u` jump shortcut | Single-line: `Enter` confirms, `Left`/`Right` move cursor, word nav |
| `Headers` | `j`/`k` to select header, `a` to add, `d` to delete, `e`/`Enter` to edit selected; `h` jump shortcut | `Tab`/`Shift+Tab` toggles key/value field, `Enter` cycles through key→value→save, autocomplete on key field |
| `Body` | `j`/`k`/`PgUp`/`PgDn` to scroll; `e`/`Enter` to edit; `b` jump shortcut | Multi-line: `Enter` inserts newline, `^S` saves, word nav |
| `RequestEvents` | `j`/`k`/`PgUp`/`PgDn` to scroll; not editable | N/A |
| `Response` | `j`/`k`/`PgUp`/`PgDn` to scroll; `f`/`p`/`x` to edit filter/regex; `v` to cycle view mode; not directly editable | N/A |

## Header autocomplete

When editing a header key (`EditingField::Headers` with `editing_header_key = true`), an autocomplete dropdown appears showing standard HTTP header names (from the IANA registry, `get_standard_headers()` in `app.rs:1228`).

- **Filtering**: Uses `fuzzy_match()` (case-insensitive subsequence match) against the typed prefix.
- **Sorting**: Headers that start with the query appear first; remainder sorted by length ascending.
- **Navigation**: `↓`/`↑` to move selection (`header_autocomplete_selected`).
- **Selection**: `Enter` applies the selected suggestion via `apply_autocomplete_selection()` — fills the key buffer, switches to value editing, hides dropdown.
- **Visibility**: Autocomplete shows automatically when starting header edit; hides on selection or switching fields.

## Persistence

Requests are persisted as a `Vec<HttpRequest>` serialized to JSON via `serde_json`.

- **Location**: `$XDG_CONFIG_HOME/imhottep/request-library.json` (falls back to `$HOME/.config/imhottep/request-library.json`, then `./request-library.json`).
- **Load**: `load_requests_from_dir()` at startup — if the file doesn't exist, returns an empty list.
- **Save**: `save_requests_to_dir()` via `s` key — creates parent directories if needed, writes pretty-printed JSON.
- **Content**: Each `HttpRequest` stores method, URL, headers, body, jq_filter, stream_prefix_regex, stream_suffix_regex.

## Dependencies

`ratatui` (TUI rendering), `hyper`/`hyper-util`/`http-body-util` (HTTP 1.x client), `hyper-tls` (TLS via native-tls/OpenSSL), `tokio` (async runtime), `serde`/`serde_json` (serialization), `regex` (prefix/suffix stripping), `ansi-to-tui` (ANSI colour rendering for `jq` output), `unicode-width` (cursor positioning).

---
> Source: [feakuru/imhottep](https://github.com/feakuru/imhottep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
