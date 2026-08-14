## justice-league-pinball

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Justice League homebrew pinball machine (Emerson Engineering). The repo has two coupled halves:
- Repo root: a Mission Pinball Framework (MPF) machine config — declarative YAML defining modes, shows, and hardware.
- `Justice-League-Pinball-Godot/`: a Godot 4.7 project using the vendored `mpf-gmc` addon (Godot Media Controller) to render the display — slides, videos, animations — driven by events and player variables pushed from the MPF machine over the network.

There is no in-repo build/test/lint tooling for either half (no requirements.txt/pyproject.toml, no Godot export scripts). The MPF side runs via the standard `mpf` CLI from the machine folder (repo root); the Godot side is opened and run through the Godot 4.7 editor. Neither is documented further in-repo.

## MPF machine config (repo root)

- `config/config.yaml` is the master config (`config_version=6`) and explicitly includes `config_coils.yaml`, `config_switches.yaml`, `config_ball_devices.yaml`. It defines the hardware platform (`smart_virtual` — runs without real hardware attached), `machine_vars`, the authoritative `modes:` list (which modes are actually loaded), per-character `player_vars` (`*_mode_status`, `*_cycle_mult` for each hero), playfields, the `blinkenlights:` block, and the light driver map.
- `modes/<name>/config/<name>.yaml` — one folder per mode, `config_version=6`. Common keys: `mode:` (start/stop events, priority), `event_player:` (custom event wiring), `slide_player:` (maps mode events to Godot slides), `shots:` (defined centrally in `modes/base/config/base.yaml`, not per-mode), `variable_player:` (scoring), `sound_player:`/`show_player:`.
  - Not every mode folder on disk is wired into `config/config.yaml`'s `modes:` list — check there before assuming a mode is active (e.g. `mother_box_multiball`, `lantern_jets`, `martian_manhunter`, `mystery_award`, `qualify`, `skillshot` are currently unlisted).
  - Some mode folders contain stale files/dirs suffixed `(old)` — ignore those.
- `shows/*.yaml` — light/flasher animation sequences (`#show_version=6`; frame lists of `duration` + `lights:` state), triggered via `show_player:` in mode configs. This is separate from `slide_player`, which drives the Godot display; the two mechanisms stay in sync only via shared event names (e.g. `mode_X_started`), not any direct coupling.
- `data/` — MPF's persisted runtime state: `audits.yaml`, `high_scores.yaml`, `machine_vars.yaml`.
- `monitor/` — config/workspace for MPF's Monitor tool.
- No custom Python code exists anywhere in the machine config — all logic is declarative YAML.

## Godot display (`Justice-League-Pinball-Godot/`)

- Godot 4.7 project (`project.godot`), mobile renderer. Single autoload `MPF` → `res://addons/mpf-gmc/mpf_gmc.gd` is the global entry point used everywhere to talk to the MPF machine (`MPF.server`, `MPF.media`, `MPF.game`).
- `addons/mpf-gmc/` is a vendored third-party framework (the Godot Media Controller). Treat it as upstream/read-only — put game-specific logic outside it.
- `gmc.cfg` — keyboard-to-switch mappings for testing without real cabinet hardware, plus sound bus definitions (`music`, `effects`, `voice`).
- `slides/` — the display slides (~40 `.tscn` files) plus their root scripts (`character_select_slide.gd`, `mpf_character_unlocks.gd`, `character_lock_toggle.gd`, `initials_logic.gd`, `alfred.gd`). Slide root scripts extend an addon base class (e.g. `MPFSceneBase`) and receive machine state via MPF signal callbacks — e.g. `MPF.server.player_variable_changed` → `_on_current_item_var_value_changed()` in `character_select_slide.gd` — rather than polling. Follow that event-driven pattern for new slides.
- `scripts/` — supporting controllers not tied to one slide (`character_select_grid_controller.gd`, `character_info_panel_controller.gd`, `character_state.gd`, `mpf_grid_highlight.gd`). Note: on the live `character_select.tscn` scene, the actually-attached scripts are `character_lock_toggle.gd` (repo root) + `character_select_grid_controller.gd` + `character_select_info_panel.gd` — `character_select_slide.gd`, `mpf_character_unlocks.gd`, `character_state.gd`, and `character_info_panel_controller.gd` are earlier iterations no longer referenced by any `.tscn`.
- `widgets/` — `MPFWidget`-rooted scenes shown via `widget_player:` (not `slide_player:`), addable to any slide or to the display's special always-on-top `_overlay` container (`slide: _overlay` in the widget's YAML settings) so they persist regardless of which slide is currently active. `widgets/roster_icons/` holds the per-hero roster HUD icons (see below).
- `shaders/` + `materials/` — currently just the grayscale "locked character" effect (`grayscale.gdshader`, `locked_grayscale*.tres`).
- `videos/`, `images/`, `sounds/`, `fonts/` — large binary media assets, tracked via Git LFS.
- **Ignore `project-DreamQuest.godot`** — a stray leftover project file, not the active one (`project.godot` is active).
- **Ignore the sibling `justice-league-pinball-(4.4)/` directory** at the repo root — an untracked, pre-upgrade Godot 4.4 backup copy of this project (1.3 GB, not part of git). Never edit files there; always work in `Justice-League-Pinball-Godot/`.

## Roster HUD (in progress)

Persistent on-screen hero-status HUD: one icon per hero, greyed out while `<hero>_mode_status != complete`, full color once assembled, meant to stay visible on top of every slide during a ball (not just the home screen).

- Mechanism: `modes/base/config/base.yaml` has a `widget_player:` block that plays/removes `widgets/roster_icons/<hero>_roster_icon.tscn` widgets on `mode_base_started`/`mode_base_stopping`, targeting `slide: _overlay` — the mpf-gmc addon's dedicated always-on-top container (`MPFDisplay._get_overlay_slide()` in `addons/mpf-gmc/classes/mpf_display.gd`, `z_index=4000`), not the normal slide stack. This was the first use of `widget_player`/`slide: _overlay` in this repo.
- Each widget reuses existing pieces unmodified: `mpf_variable.gd` (binds a hidden Label to the `<hero>_mode_status` player var) + `character_lock_toggle.gd` (already used on `character_select.tscn`; swaps the icon's material between normal and `materials/locked_grayscale_material.tres` based on that Label's text).
- **Status: Batman only, proof-of-concept.** `batman_roster_icon.tscn` and the `base.yaml` wiring exist and are pending manual in-editor verification (no in-repo test tooling to verify headlessly — see below). Icon art is a placeholder (`images/batman_logo.png`); real per-hero icon art is still needed. Once verified, the same pattern repeats for superman/flash/aquaman/cyborg/wonder_woman.

## Conventions

- Git LFS covers binary media in both halves of the repo (`*.wav`, `*.png`, `*.jpg`, `*.mp4`, `*.ogv`, `*.ttf`, etc. — see `.gitattributes` at repo root and inside `Justice-League-Pinball-Godot/`, which track an overlapping but not identical set of extensions). GitHub's free LFS tier defaults to 1GB storage/1GB bandwidth per month — a push that fails on an LFS quota error is a GitHub billing issue, not a git mistake.

## Git workflow

- Two branches: `Working` is where day-to-day development happens (commit often, push regularly — nothing is backed up until it's pushed to `origin`); `main` is the stable "known-good" checkpoint branch, fast-forwarded from `Working` only when the game is in a solid, demoable state (`git checkout main && git merge Working && git push origin main`).
- `.gitignore` (repo root) excludes `.vs/`, `.claude/settings.local.json`, and the sibling `justice-league-pinball-(4.4)/` backup folder — none of those should ever be committed.
- Annotated tags mark known-good checkpoints worth being able to revert to, named `checkpoint-<date>-<short-description>` (e.g. `checkpoint-2026-07-25-working-game`). Create one whenever `main` is updated to a build worth preserving.

---
> Source: [SpaceEngineer41/Justice-League-Pinball](https://github.com/SpaceEngineer41/Justice-League-Pinball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
