## systemdkit

> - Use the project Mix aliases; prefer `mix ci` for the full validation suite.

# Agent Guidelines

## Development

```sh
mix deps.get
mix ci
```

## Conventions

- Use the project Mix aliases; prefer `mix ci` for the full validation suite.
- Keep changes small, tested, and formatted.

## systemd integration VM

A Lima Debian VM named `systemd-test` is available for real systemd/D-Bus checks:

```sh
~/.local/bin/limactl shell systemd-test
~/.local/bin/limactl shell systemd-test -- systemctl is-system-running
~/.local/bin/limactl shell systemd-test -- busctl --system list --no-pager
```

Integration tests are excluded by default. Enable them only inside a Linux systemd environment. The `systemd-test` VM has Debian packages installed for this (`elixir`, `erlang-dev`, `git`, `build-essential`).

```sh
SYSTEMD_INTEGRATION=1 mix test
```

From macOS, prefer the checked-in wrapper for a full VM integration run:

```sh
scripts/integration_test.sh
```

---
> Source: [elixir-vibe/systemdkit](https://github.com/elixir-vibe/systemdkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
