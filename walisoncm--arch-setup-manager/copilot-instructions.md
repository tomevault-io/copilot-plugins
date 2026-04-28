## arch-setup-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Arch Setup Manager is a graphical app manager for CachyOS/Arch Linux, built on top of [BigBashView](https://github.com/biglinux/bigbashview). The UI runs as a local HTTP server that serves shell-generated HTML pages in a WebView window. Every page is produced by a Bash script that outputs complete HTML documents.

## Running the App

```bash
./launcher.sh
```

Requires `bigbashview` (install with `paru -S bigbashview-git`). Must not be run as root.

## Architecture

### Request Flow

BigBashView routes URLs of the form `/execute$./script.sh args` by running the script and serving its stdout as an HTTP response. Navigation between pages works through `<a href="/execute$./script.sh arg">` links. Each script outputs a full HTML page.

### Core Scripts

| Script | Role |
|---|---|
| `launcher.sh` | Entry point; starts BigBashView pointing at this directory |
| `main.sh` | Home page; renders category grid |
| `category.sh CAT` | Lists apps in a category with install/remove/manage buttons |
| `action.sh ACTION APP CAT` | Action page; triggers `run.sh` via JS fetch and polls `log-tail.sh` for output |
| `run.sh ACTION APP` | Executes `install_app`/`remove_app` from `backend.sh`; streams ANSI-stripped HTML to a log file at `/tmp/bbv-action-<app>.log` |
| `confirm.sh APP CAT` | Removal confirmation page |

### Shared Modules

- **`common.sh`** — CSS variables, HTML header/footer helpers, and `format_output` (converts ANSI log colors to HTML `<span class="t-ok/t-warn/t-err/t-step">`)
- **`backend.sh`** — Dynamically sources all category and app files; provides `install_app`, `remove_app`, `status_app`, `manage_app`, `launch_app`, `name_app`, `desc_app`
- **`lib/helpers.sh`** — Low-level package helpers: `has_pkg`, `has_fpk`, `has_cmd`, `svc_enabled`, `install_pkg`, `remove_pkg`, `install_fpk`, `uninstall_fpk`, and logging functions `log`/`warn`/`err`/`step`/`ok`

### Real-Time Terminal Output

`action.sh` renders a terminal UI in the browser. It fires `run.sh` via `fetch()`, then polls `log-tail.sh OFFSET` every 500ms to stream new bytes. The process writes to `/tmp/bbv-action-<app>.log` and appends `___DONE_<exitcode>___` when finished. `kill.sh` terminates the process tree. `stdin-relay.sh` writes user input into `/tmp/bbv-stdin-<app>.fifo`.

### sudo Handling

`askpass.sh` (a Zenity password dialog) is set as `SUDO_ASKPASS`. `run.sh` wraps `sudo` to use `-A` (askpass) only when passwordless sudo fails. A background loop (`sudo -n true; sleep 40`) keeps credentials alive for the duration of long installs.

## Adding a Category

1. Create `categories/<id>/category.sh` and set `CAT_ID`, `CAT_TITLE`, `CAT_ICON`, `CAT_DESC`. Set `CAT_TYPE=config` for configuration categories (vs the default `app`).
2. Add app files `categories/<id>/<app-id>.sh`.

## Adding an App

Create `categories/<cat>/<app-id>.sh` and set `APP_ID`, `APP_NAME`, `APP_DESC`. Override any of these functions as needed (all are optional):

- `status_<id>()` — returns 0 if installed (default: `has_pkg` or `has_fpk` based on whether `APP_ID` contains a dot)
- `install_<id>()` — custom install logic (default: `install_pkg` or `install_fpk`)
- `remove_<id>()` — custom remove logic (default: `remove_pkg` or `uninstall_fpk`)
- `manage_<id>(bbv_base)` — if defined, adds a ⚙ button; must echo two lines: line 1 is the JS onclick expression, line 2+ is HTML (modal markup + `<script>`) appended to the category page
- `launch_<id>()` — if defined, echoes a URL; adds a ↗ button that calls `xdg-open` via `launch.sh`

App IDs with dots are treated as Flatpak IDs (e.g. `com.usebottles.bottles`). IDs without dots are pacman/AUR packages. Function names normalize `.` and `-` to `_` (e.g. `APP_ID=my-app` → `install_my_app()`).

## Dedicated Manage Scripts

Complex apps use standalone manage scripts (e.g. `ollama-manage.sh`, `noctalia-manage.sh`) called via BBV fetch from JS. These scripts handle actions like `list`, `delete`, `service-start`, `service-stop` and return HTML fragments (not full pages).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/walisoncm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
