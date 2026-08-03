## trapmap

> Static site (index.html, view.html) served by `server.py` (Python http.server, port 8080). Backend is Google Apps Script (`appsscript.gs`).

# TrapMap

Static site (index.html, view.html) served by `server.py` (Python http.server, port 8080). Backend is Google Apps Script (`appsscript.gs`).

## Preview / running locally

The Claude preview launcher cannot read ~/Documents (macOS TCC), so the preview serves a mirror in /tmp. To start the preview:

1. Sync the mirror:
   `rsync -a --delete --exclude .git --exclude .claude /Users/leviticusalmanzar/Documents/TrapMap/ /tmp/trapmap-preview/`
2. `preview_start` with config name `trapmap` (defined in .claude/launch.json — it serves from /tmp/trapmap-preview).

After editing any project file, re-run the rsync, then reload the preview.

---
> Source: [thebugbouncer/TrapMap](https://github.com/thebugbouncer/TrapMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
