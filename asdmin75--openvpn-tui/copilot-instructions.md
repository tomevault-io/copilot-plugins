## openvpn-tui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
# Build (go.work in parent dir must be bypassed)
GOWORK=off go build -o openvpn-tui .

# Full build + launch under sudo (what users invoke)
./run.sh

# Static checks
GOWORK=off go vet ./...

# Update module deps
GOWORK=off go mod tidy
```

There is no test suite yet. When adding tests, run with `GOWORK=off go test ./...`.

`run.sh` unconditionally rebuilds every launch — that is intentional, not a caching bug.

## Runtime preconditions

- The binary refuses to start unless `euid == 0`. A valid launch is `sudo -E ./openvpn-tui`.
- `./config/*.ovpn` must exist and is resolved relative to CWD first, then alongside the binary. `config/` is gitignored — never commit it.
- `openvpn` must be in PATH. `rdesktop` is optional; missing rdesktop degrades the `r` key to a status message.
- RDP store lives at `~/.config/openvpn-tui/rdp.yaml` **of the real user**, not root. `internal/rdp/rdp.go` resolves HOME via `SUDO_USER` → `/home/<user>` when running under sudo, and `internal/rdp/ownership.go` chowns the created files back to `SUDO_UID:SUDO_GID`. Do not bypass this when touching rdp code.

## Architecture

Three internal packages plus `internal/tui`:

- **`internal/configs`** — one-shot directory scan of `*.ovpn` and a `HasEncryptedKey` detector that greps for `ENCRYPTED PRIVATE KEY` in the file. Used by TUI to decide whether to show the password modal before `manager.Start`.

- **`internal/vpn`** — process supervision. `Session` wraps one `exec.Cmd` (`openvpn --config <path>` plus `--askpass <tmp>` if a password was set). Two goroutines per session: `reader` parses stdout line-by-line (regex matches `Initialization Sequence Completed` → `StatusConnected`, `ifconfig <ip>` → session IP) and streams `Message` values; `waiter` emits a terminal status event on exit. `Manager` is a registry of sessions plus a password cache (in-memory only); it owns the single outbound channel that **all sessions** write to. Do not fan out per-session channels — the TUI's Bubble Tea loop subscribes to exactly one source.

- **`internal/rdp`** — YAML store plus `Launch()` which spawns `rdesktop` detached (own goroutine reaps it) so SIGINT on the TUI does not kill the RDP window.

- **`internal/tui`** — Bubble Tea `Model/Update/View` split across `model.go` (state + `waitForVPN` cmd bridging the manager channel back into tea messages), `update.go` (main / password / editor dispatch + vpn event handler), `view.go` (lipgloss two-pane layout with selected-row highlight and footer status bar), `editor.go` (modal RDP editor with `bubbles/textinput`).

### The channel bridge

`waitForVPN(mgr)` returns a `tea.Cmd` that blocks on one message from `mgr.Events()` and delivers it as `vpnMsg`. `Update` re-issues `waitForVPN` after every delivery — this is the **only** way VPN logs and status events reach the TUI. If you add new event types, route them through `vpn.Message` (either `Log` or `Event`), not a second channel.

### Password flow

Encrypted-key detection is intentionally at the UI layer, not in `vpn.Start`. On `Enter`, `updateMain` checks `configs.HasEncryptedKey` and `manager.Password`; if key is encrypted and cache is empty it opens the password modal, otherwise calls `manager.Start` directly. The modal caches the password in `Manager.passwords` (memory-only, never persisted), then re-invokes `Start`. `vpn.Start` writes the password to `/dev/shm/ovpn-askpass-*` mode `0600`, passes `--askpass <path>`, and deletes the file 3 seconds after spawn (openvpn reads `--askpass` synchronously at startup, well before 3s). Keep this deletion goroutine — the file must not linger.

## Branding / icon

`openvpn-tui.svg` is the custom icon referenced by absolute path in `openvpn-tui.desktop`. When moving the project, edit `Exec=` and `Icon=` in the `.desktop` file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ASdmin75) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
