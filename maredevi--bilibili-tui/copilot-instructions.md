## bilibili-tui

> This repository is a Rust 2024 terminal client for Bilibili. Keep changes small,

# Repository Guidelines

## Scope

This repository is a Rust 2024 terminal client for Bilibili. Keep changes small,
typed, and testable. Preserve existing keyboard behavior, navigation state, and
configuration compatibility unless the task explicitly changes them.

## Architecture

- `src/api/`: HTTP/WebSocket clients, wire models, signing, and protocol parsing.
- `src/application/`: application actions, network commands, and network events.
- `src/app/`: page orchestration, navigation, runtime event handling, and action dispatch.
- `src/ui/`: Ratatui pages, widgets, themes, and input state machines.
- `src/domain/`: application-domain state such as playback queues.
- `src/player/`: mpv integration, stream failover, proxying, and danmaku scripts.
- `src/storage/`: persisted configuration, credentials, and keybindings.
- `src/infrastructure/` and `src/presentation/`: adapters and public-facing re-exports.

Keep wire-format concerns in `api`, state transitions in `app`/`ui`, and external
process or persistence details in their dedicated modules. Do not make rendering
functions perform network requests.

## Development Workflow

Use the native toolchain when available. On NixOS, run the same commands through
`nix develop --command`:

```sh
cargo fmt --check
cargo test
cargo clippy --all-targets --all-features -- -D warnings
cargo build
```

Run focused tests while iterating, then run all four checks before handoff. Do
not commit `target/`, local configuration, credentials, cookies, or diagnostics.

## Rust Style

- Follow `rustfmt`; use `snake_case` for functions/modules and `PascalCase` for types.
- Prefer small helpers that express one state transition or protocol operation.
- Reuse shared UI helpers such as `shortcut_footer` instead of duplicating styles.
- Avoid cloning collections in draw loops; borrow slices and separate mutable widget state.
- Avoid `unwrap`/`expect` outside tests unless an invariant is established at startup.
- Keep error messages concise and actionable; retain the original error as context.
- Use explicit action/event names such as `OpenVideoDetail` and `VideoDetailLoaded`.

## Behavioral Invariants

- A page must never remain in a loading/deleting/input mode after a request is
  rejected or fails. Every asynchronous transition needs a success and failure exit.
- Preserve history shortcuts: Space toggles, Ctrl+A selects loaded eligible items,
  Ctrl+R inverts them, `d` confirms deletion, and Esc cancels selection/dialogs.
- PGC history uses season and exact episode identifiers; normal archives use BVID/AID.
- Playback quality caps must never silently select a stream above the configured limit.
- Keep footer hints centered, bracketed, color-coded, and built from configured keybindings.
- Paginated and asynchronous results must ignore stale request IDs.

## API and Security

Treat Bilibili endpoints as unstable contracts. Use serde defaults only where the
field is genuinely optional, and add minimized fixtures for numeric/string/null drift.
Keep endpoint details behind API client methods.

Never log or persist cookies, CSRF tokens, response bodies, signed media URLs,
private identifiers, or query strings. Redact sensitive URL components in errors
and diagnostics.

## Testing

Place unit tests beside the implementation. Name tests by observable behavior,
for example `cancel_deletion_restores_selection_mode`. Add tests for:

- mixed or drifting API payloads;
- keyboard and mouse state transitions;
- pagination and stale network responses;
- configuration defaults and round trips;
- playback queue and stream-selection behavior.

Network, login, and mpv contract tests may remain ignored with a clear reason.

## Commits and Pull Requests

Use scoped Conventional Commit subjects (`feat:`, `fix:`, `refactor:`, `test:`,
`chore:`). Pull requests should summarize user-visible effects, implementation
choices, API/configuration assumptions, and exact validation commands. Include a
screenshot when layout or theme behavior changes.

---
> Source: [MareDevi/bilibili-tui](https://github.com/MareDevi/bilibili-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
