## zmux

> if you create/update/delete key bindings, always add them to the readme to keep it up to date


if you create/update/delete key bindings, always add them to the readme to keep it up to date

## Server logs

Server logs are written to `$XDG_RUNTIME_DIR/zmux/zmux.log.*` (daily rolling files, typically `/run/user/1000/zmux/`). Logging is configured in `src/server.rs` `setup_logging()` using `tracing` + `tracing-appender`. Default log level is `info`, overridable via `RUST_LOG` env var.

## Hot reload (ctrl-b u)

The reload mechanism serializes server state to JSON, then `exec()`s the new binary with `--reload <state_path>`. Key files: `src/server.rs` (`perform_reload`, `serialize_state`, `run_server_restore`), `src/protocol.rs` (`Reload`/`Reloading` messages), `src/client.rs` (`reload()`).

Known pitfall: after `cargo build` replaces the binary, `/proc/self/exe` returns a `(deleted)` path. The code strips this suffix before exec'ing.

---
> Source: [zingbretsen/zmux](https://github.com/zingbretsen/zmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
