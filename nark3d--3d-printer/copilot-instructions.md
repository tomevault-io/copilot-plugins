## 3d-printer

> This repo documents and manages a Klipper-driven 3D printer (ZeroG Mercury One, CoreXY, BTT Octopus Pro + EBB36 Gen2 toolhead, Beacon probe, Orbiter 2.5 extruder, hotbox chamber heater). Adam is the owner/operator.

# Project instructions for Claude

This repo documents and manages a Klipper-driven 3D printer (ZeroG Mercury One, CoreXY, BTT Octopus Pro + EBB36 Gen2 toolhead, Beacon probe, Orbiter 2.5 extruder, hotbox chamber heater). Adam is the owner/operator.

## Read these FIRST when starting a new session

1. **`NEXT_STEPS.md`** — the current resume point. Status, pending items, and the "first concrete action" for the next session.
2. **Auto-memory files at `~/.claude/projects/-Users-adamlewis-Repos-3d-printer/memory/`** — specifically `MEMORY.md` which indexes entries. Load relevant memories for whatever task the user asks about.

## Safety-critical rules for interacting with the printer

**On 2026-04-12 I destroyed a 2-hour print and damaged hardware by sending Klipper RESTART without checking print state, then a G28 that drove the nozzle through the print into the bed.** Read `memory/never_send_printer_commands_without_permission.md`. These rules exist because I've already failed at them.

### The absolute rules

- **NEVER send motion, temperature, or restart commands without explicit per-command permission from Adam.** This includes G0, G1, G28, G29, G30, Z_TILT_ADJUST, BED_MESH_CALIBRATE, SMART_PARK, LINE_PURGE, PROBE_ACCURACY, RESTART, FIRMWARE_RESTART, M104, M109, M140, M190, SET_HEATER_TEMPERATURE, and anything that moves or heats.
- **ALWAYS check `print_stats.state` before any command that could affect a running print.** Query `GET /printer/objects/query?print_stats`. If state is `printing` or `paused`, STOP.
- **Read-only queries are safe autonomously** — GET requests to Moonraker, SSH reads of log/config files, SCP pulls of result PNGs, analysis of local files. Adam explicitly authorised this workflow on 2026-04-12.
- **The safety wrapper and Claude Code hook are hard blocks**, not suggestions. `~/printer_cmd.sh` filters commands, and `.claude/hooks/block-printer-writes.sh` intercepts raw curl POSTs. Even with permission, respect these — they're the mechanical enforcement of the policy.

### The correct workflow for any command that changes printer state

1. Show Adam the exact command and why
2. Wait for explicit approval (`yes`, `go`, `do it`, or similar)
3. Before executing: check `print_stats.state` is `standby` (or acceptable)
4. Execute via `~/printer_cmd.sh` if the command is permitted by the wrapper's blocklist
5. If the wrapper blocks it (motion, temperature, restart): Adam runs it himself from Mainsail console

### If in doubt, ask

Pausing to confirm is free. An unwanted printer command is potentially hundreds of pounds of hardware damage. The cost asymmetry is extreme. Err on the side of asking.

## Working with Adam — communication and collaboration

- **Adam is technically sophisticated.** Don't over-explain basics. He builds printers from scratch and calibrates Klipper fluently.
- **Be terse.** Skip preambles and summaries unless he asks. Answer the question asked.
- **Don't over-claim.** Multiple memory files document my past pattern of presenting hypotheses as facts (`memory/dont_push_unneeded_tuning.md`, `memory/beacon_vs_lis2dw_uncertainty.md`). Frame non-obvious technical claims as: *hypothesis X, evidence Y, confidence Z, alternatives A/B/C*.
- **Don't push tuning steps when there's no concrete problem.** Ask "what problem are we solving?" first. NEXT_STEPS.md is a menu of options, not a checklist.
- **Listen to "no".** When he corrects an approach, update memory so the correction persists. Don't re-propose the same rejected idea in later sessions.

## Project structure

```
/
├── CLAUDE.md                       ← this file
├── NEXT_STEPS.md                   ← current resume point, read first
├── HARDWARE.md                     ← hardware reference
├── .mcp.json                       ← local MCP printer server config (gitignored, contains real IP)
├── docs/
│   ├── references.md               ← best-practices library
│   ├── adaptive_extruder_current_deep_dive.md  ← research doc, adaptive TMC current
│   └── klipper_config_cleanup_plan.md          ← research doc, config cleanup plan
├── orca_slicer/
│   ├── changelog/                  ← dated changelog entries for slicer setting changes
│   ├── backups/                    ← per-round profile snapshots for rollback
│   ├── process/                    ← process profile research (seam, ironing, PLA docs)
│   ├── printer/                    ← printer profile research + LED effects research
│   ├── filament/                   ← filament profile research (currently SUNLU PLA Matte)
│   └── user/default/               ← local OrcaSlicer profile snapshots
├── tuning/
│   ├── 2026-04-10/                 ← Shake&Tune baseline session
│   └── claude-fucked-up-big-time/  ← 2026-04-12 post-disaster recalibration with apology
└── backups/                        ← printer config + database snapshots
    ├── 2024-04-01/, 2024-04-27/    ← historical
    └── 2026-04-10/                 ← pre-disaster baseline
```

## On-printer files (not in repo)

- `~/printer_data/config/*.cfg` — Klipper configs. `printer.cfg` is main; includes `ebb_gen2.cfg`, `macros.cfg`, `led_macros.cfg`, `hotbox_macros.cfg`, `mainsail.cfg`, `KAMP_Settings.cfg`, `moonraker_obico_macros.cfg`. Full audit + cleanup plan in `docs/klipper_config_cleanup_plan.md`.
- `~/printer_data/config/.pre-*-backup-*/` — per-change backups Adam or I have created during edits.
- `~/printer_data/logs/klippy.log`, `moonraker.log`, etc. — operational logs. Always read-only.

## Memory entries (auto-loaded)

Location: `~/.claude/projects/-Users-adamlewis-Repos-3d-printer/memory/` — NOT in this repo. `MEMORY.md` is the index.

Current entries cover: relay trip gotcha, chamber PID skip, don't push unneeded tuning, Beacon-vs-LIS2DW uncertainty (now with 2026-04-12 update supporting the mount-flex hypothesis), Klipper estimator static config, always check print state before restart, never send printer commands without permission, Shake&Tune doesn't autosave, autonomous analysis commands OK.

**Don't duplicate memory content into CLAUDE.md.** Memory is for project-specific facts that evolve; CLAUDE.md is for session-invariant rules.

## Git workflow

- Main branch: `main`. Remote: `github.com:nark3d/3d-printer.git`.
- Commits require explicit user request — don't commit proactively.
- Attribution: add `Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>` to commit messages.
- History contains dead Obico tokens (rotated on 2026-04-12, no longer valid) from the PII scrub that kept git history intact (Option A). Not a security concern.
- `.mcp.json` and `moonraker-obico.cfg` must NEVER be committed — they contain the real printer IP and the active Obico auth token respectively.

## When adding new docs to the repo

- Match existing patterns. `orca_slicer/*/deep_dive.md` for slicer research. `tuning/YYYY-MM-DD-something/` for calibration sessions. `docs/` for general reference.
- Include reference lists on research docs (we aim for 30+ citations on deep-dives).
- Don't create README.md files unless explicitly requested. Don't add docstrings or comments to code that doesn't need them.

## When running calibration workflows

- Shake&Tune macros (AXES_SHAPER_CALIBRATION, COMPARE_BELTS_RESPONSES, CREATE_VIBRATIONS_PROFILE) are DIAGNOSTIC only. They write PNGs but DON'T persist values to SAVE_CONFIG. See `memory/shaketune_doesnt_autosave.md`. For values to persist, either use Klipper's native SHAPER_CALIBRATE (auto-applies and queues for SAVE_CONFIG) OR manually edit the `[input_shaper]` auto-save block in printer.cfg.
- Shake&Tune results live in `~/printer_data/config/ShakeTune_results/{belts,input_shaper,vibrations}/` — pull PNGs from there.
- Input shaper calibration is blocked by my safety wrapper (matches `SHAPER`). Adam either runs it himself or explicitly authorises via the wrapper.

## Don't

- Adopt a Klipper macro framework (Klippain, klipper-macros) — documented decision in `docs/klipper_config_cleanup_plan.md`
- Recommend TMC autotune on this printer — tried, regressed, reverted (memory entry)
- Propose chamber heater PID calibration — Adam declined (memory entry)
- Add `shutdown_value: 1` to the BTT relay — Adam keeps default for fail-safe (memory entry)
- Use Moonraker-URL-based Klipper Estimator — flaky on this Mac; use static config (memory entry)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nark3d) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
