## supermux

> - **Never `cargo build/test --release`** — small hosts OOM-thrash. Use `cargo check` / `cargo build` (debug). The wrapper at `~/.local/bin/cargo` refuses release unless `SUPERMUX_RELEASE_OK=1`.

# supermux

- **Never `cargo build/test --release`** — small hosts OOM-thrash. Use `cargo check` / `cargo build` (debug). The wrapper at `~/.local/bin/cargo` refuses release unless `SUPERMUX_RELEASE_OK=1`.
- **To deploy from the server**: `bash scripts/deploy-self.sh`. It writes a deploy-request file; a root path-unit builds (as the service user, outside the sandbox), installs, restarts, verifies, rolls back on failure.

---
> Source: [sanderbz/supermux](https://github.com/sanderbz/supermux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
