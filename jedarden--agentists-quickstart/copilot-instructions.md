## agentists-quickstart

> This repo is two scripts for turning a bare VPS into a Claude Code/Codex + herdr coding

# Operating this repo

This repo is two scripts for turning a bare VPS into a Claude Code/Codex + herdr coding
environment. If you're an agent asked to "set this up" on a server, here's what that
means concretely.

## Setting up a new server

1. Confirm the target is actually a headless server, not someone's personal machine —
   `bootstrap.sh` will refuse to proceed non-interactively on hardware that looks
   personal (laptop/desktop chassis, battery present, active `$DISPLAY`). If you're
   running this yourself on a real VPS and it still triggers the guard (e.g. a VM with a
   virtual display device), that's a false positive — re-run with `--force` or
   `BOOTSTRAP_FORCE=1` rather than working around the check some other way.
2. Run `bootstrap.sh` once (via the curl one-liner in README.md, or a local checkout).
   It is idempotent — re-running it is always safe and just skips anything already
   installed.
3. After that, `start.sh` is the entry point for every session — not an agent directly,
   and not `herdr` directly. It handles both cases (plain terminal vs. already inside a
   herdr pane) so the same command works everywhere.

## How the two scripts fit together

- `bootstrap.sh` installs: Claude Code CLI, Codex CLI, herdr, both Herdr integrations,
  a copy of `start.sh` at `~/.local/bin/start.sh`, a passwordless-sudo rule scoped to
  exactly `choom -n -1000 -p <pid>` (used for OOM protection), and a small `~/.bashrc`
  hook.
- The `~/.bashrc` hook is the load-bearing piece: it makes every pane herdr spawns
  immediately re-exec `start.sh`. Combined with `start.sh`'s own `HERDR_ENV` check, this
  means "launch herdr" and "launch the selected agent inside herdr" collapse into one script with no
  separate pane-orchestration step — `start.sh` from a plain terminal execs `herdr`,
  which creates a pane, which re-execs `start.sh`, which sees `HERDR_ENV` set and execs
  the selected agent. Do not replace this with explicit `herdr agent start --kind ... --pane
  <id>` calls unless you have a specific reason to target a particular existing pane —
  the hook already covers the common case.
- `HERDR_ENV` (plus `HERDR_PANE_ID`/`HERDR_TAB_ID`/`HERDR_WORKSPACE_ID`) is set by herdr
  itself in every pane it spawns — this is not something either script sets.

## Constraints

- Debian/Ubuntu (apt) only. Don't add support for other package managers without
  updating both scripts and the README's stated prerequisites.
- No tmux, no zellij, no DevPod/devcontainer path. herdr is the only supported
  multiplexer here — this repo previously (as `agentists-quickstart-deprecated`) tried a
  branch-per-workspace devcontainer model and a custom zellij status hook; both were
  abandoned in favor of this simpler bare-VPS design.
- Don't add fleet/multi-agent-naming logic (NATO phonetic names, etc.) unless asked —
  out of scope for this repo's job, which is just "get Claude Code or Codex running via
  herdr on a fresh box."

---
> Source: [jedarden/agentists-quickstart](https://github.com/jedarden/agentists-quickstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
