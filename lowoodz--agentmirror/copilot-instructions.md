## release-cycle

> Release cycle — compile, package, test, install. Always follow release-cycle skill; avoid known packaging pitfalls.


# Release cycle (mandatory for agents)

**Canonical doc:** `.cursor/skills/release-cycle/SKILL.md` — read before compile/package/test/install work.

## Entry commands (do not invent ad-hoc flows)

| Host | Full validation | Single platform / phase |
|------|-----------------|-------------------------|
| macOS | `./scripts/release-full.sh` | `./scripts/release-cycle.sh [phase]` |
| macOS package only | `./scripts/package-all.sh` | `./scripts/package-macos.sh` |
| Windows | `.\scripts\windows\release-cycle.ps1` | `-Phase package` / `-CliOnly` |

Always:

```bash
export CARGO_TARGET_DIR="$PWD/target"   # bash
$env:CARGO_TARGET_DIR = "$PWD\target"  # PowerShell
```

## Before every package run

1. `./scripts/sync-admin-ui.sh` (or compile phase — already calls it)
2. `./scripts/clean-dist.sh` — removes stale logs/IExpress junk; cleans UTM `C:\Users\Public\smr-*` when SSH up
3. `./scripts/uninstall.sh` / `uninstall.ps1` before install tests — free :8080

## Windows installer — NSIS only

- **Use:** `scripts/vm/package-windows-gui.sh` (Mac+UTM) or `scripts/package.ps1` (Windows native)
- **Never:** IExpress / `SafeRoute-*-x64-Setup.exe` naming — scripts removed from repo
- **Canonical artifact:** `dist/SafeRoute_{version}_x64-setup.exe`
- NSIS build must **fail hard** if Tauri build fails — do not reuse old `*-setup.exe`

## Stale UI / stale server (do not repeat)

- GUI may reuse an old `smr.exe` on :8080 when semver matches but embedded admin UI differs
- `/health` returns `ui=` digest — GUI must refuse mismatch
- After UI changes: rebuild `smr-cli`, repackage, reinstall

## Logs and paths

- Artifact paths: `scripts/dist-layout.sh` → `dist/LATEST-INSTALLERS.txt`
- Use **fixed** log names under `dist/` (e.g. `macos-release-cycle.log`) — no timestamped `package-all-*.log` in `dist/` root
- Detailed test logs: `dist/test-runs/` only

## Live tests

- Require gitignored `config/test.env` (from `config/test.env.example`) — never commit keys
- UTM tests run via **`SMR_WINDOWS_USER` SSH** only (`vm-ssh.sh`)

## VM (SSH only)

- Account: **`SMR_WINDOWS_USER`** from `config/test.env` — all VM ops via `vm-ssh.sh`
- If `ssh windows-vm` fails: **stop** — user fixes manually in UTM console; no automated SSH fix in repo

---
> Source: [lowoodz/AgentMirror](https://github.com/lowoodz/AgentMirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
