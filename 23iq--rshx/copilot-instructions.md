## rshx

> This file is the working contract for agents and contributors changing Remote

# RSHX agent guide

This file is the working contract for agents and contributors changing Remote
Shell eXtended.

## Product invariants

1. The binary, package, data directory, and protocol namespace are `rshx`.
2. A connection alias is presentation data. Its immutable ID owns keys and
   references, so rename operations never break authentication.
3. The muted canonical endpoint must remain visible beside an alias in every
   destructive or connection UI.
4. Passwords are ephemeral. They must never enter SQLite, logs, errors, command
   arguments, environment variables, disk, clipboard, or shell history.
5. Host-key trust is explicit. Reset removes one exact endpoint and never
   implies acceptance of the next key.
6. RSHX never invokes a shell to build a local command. Every subprocess uses
   a program plus separate arguments.
7. SSH remains behind an RSHX-owned PTY. Do not replace the process with
   `exec(ssh)` or surrender all input handling.
8. Plugins receive no credentials, raw keystrokes, or session output by
   default. New capabilities require documentation, validation, and tests.
9. A broken plugin cannot prevent startup or connection management.
10. Local directories and sensitive files must retain restrictive permissions.
11. Provisioning rows stay hidden until authentication setup is published as
    ready. Preserve recoverable state when the remote commit is uncertain.
12. Plugin remote-command approval belongs to a locally rendered, fit-checked
    screen. Never accept confirmation over the remote PTY.

## Modules

- `domain`: endpoint parsing, alias validation, and connection entities.
- `paths`: XDG/platform data locations and permission creation.
- `storage`: SQLite schema, migrations, lifecycle state, and exact-alias CRUD.
- `ssh`: key scanning, trust, nonce-authenticated askpass provisioning, and
  OpenSSH PTY processes.
- `session`: raw-byte PTY proxying, leader chords, and local plugin consent.
- `plugins`: versioned manifests and capability validation.
- `tui`: pure-ish UI state, rendering, and intent emission.
- `cli` / `main`: orchestration only; business rules stay in shared modules.

## Change checklist

Run:

```console
cargo fmt --all -- --check
cargo test --all-targets
cargo clippy --all-targets -- -D warnings
```

For changes to authentication or host trust, add a regression test and update
`docs/security.md`. For new keybindings, update the typed registry and generate
UI help from that source rather than duplicating strings. For new plugin
capabilities, document what data crosses the boundary and when consent occurs.
For provisioning lifecycle changes, test interrupted recovery, discard, and
the uncertain remote `authorized_keys` commit case.

Never add real endpoints, known-host entries, private keys, passwords, database
files, or captured production terminal output to fixtures.

---
> Source: [23iq/rshx](https://github.com/23iq/rshx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
