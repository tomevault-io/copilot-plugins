## ccstatusline-scripts

> Project memory for Claude Code sessions in this repo.

# CLAUDE.md

Project memory for Claude Code sessions in this repo.

## What this is

A collection of small Node scripts invoked by ccstatusline's **Custom Command** widget on Windows 11. Each script in `scripts/` is one status-line widget.

## Runtime invariants — DO NOT violate

ccstatusline runs each script via `execSync` with these characteristics (verified from `src/widgets/CustomCommand.tsx` upstream):

- **stdin** receives the full Claude Code session JSON. Scripts must drain it without awaiting it, or they will hang.
- **stdout** is captured and rendered. **stderr is discarded** — never write user-facing messages there.
- **Default 1000 ms timeout.** Scripts must finish well under that.
- **No shell wrapper** — the command is parsed as `argv[0] + args`, so on Windows `.js` files are invoked as `node <abs path>`.
- **No caching across invocations** — the script is re-spawned every render. File-cache external data in `%TEMP%` if it's expensive to fetch.
- **Always `process.exit(0)`** even on failure. A non-zero exit visibly breaks the user's status line.

## Conventions

- ESM modules, Node 18+, no runtime dependencies.
- All OpenHardwareMonitor access goes through `lib/monitor.js`. Don't duplicate the fetch/cache logic in individual scripts.
- The OHM server URL defaults to `http://192.168.1.185:8085/data.json` (OHM runs on this machine, addressed by its LAN IP) and is overridable with the `OHM_URL` env var.
- OHM data is file-cached in `%TEMP%\ccstatusline-monitor.json` with a 2 s TTL.
- Output one line of stdout. ANSI colors are fine; the widget's "preserve colors" toggle controls whether they render.

## Windows Defender / Node startup cost

On this machine, real-time AV scanning makes bare `node.exe` startup take 2–5 seconds. The script's own work is sub-100 ms — so the dominant cost is always Node startup, not script logic. When timing the script, account for this; don't chase microoptimizations in the script. The README documents the AV exclusion the user can apply.

## Don't

- Don't add a build step or TypeScript — these are tiny scripts.
- Don't add a test framework — verify manually per the README.
- Don't try to make scripts cross-platform; this repo is Windows 11 only.
- Don't add error logging to stderr (it's discarded) or to a file (silent failure is fine — `--°C` placeholder is the user signal).

## Verifying changes

See the "Verifying a script" section of `README.md`.

---
> Source: [areimel/ccstatusline-scripts](https://github.com/areimel/ccstatusline-scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
