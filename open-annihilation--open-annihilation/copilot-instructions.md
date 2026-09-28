## open-annihilation

> A portable engine that plays Total Annihilation 3.1c from the player's own

# open-annihilation

A portable engine that plays Total Annihilation 3.1c from the player's own
installed game data, on macOS, modern Windows and Linux: campaigns, skirmish,
computer players, simulation, unit scripts, rendering, sound and save/load,
matching the 3.1c game's behaviour and its save files. Intro, menus and first
gameplay are checkpoints, not completion.

This repository is the engine alone. Multiplayer is not part of it: an
extension library adds optional features through the hook table in
`src/app/include/oa/app/extension.hpp`, and engine code never refers to any particular
extension. Projects that build on the engine, such as an extension or suites
built on recorded data, live in their own repositories. Checkouts of them may
sit inside this tree, each with its own `AGENTS.md`; follow that file when
you work there.

Keep `./run.sh` as the user's entry point for inspecting actual progress. It
must build current source and launch the current native application, stop on
a failed build instead of falling back to stale binaries, and follow the main
frontend/game executable as integration advances. State current limitations
plainly; do not substitute another executable or a mock game.

Keep game assets and local captures out of Git.

## Rules

[docs/conventions.md](docs/conventions.md) holds the rules every change
follows, with their reasons and the directories they cover: compatibility
with 3.1c save files, game data and simulation results; describing behaviour,
never derivation; naming every field by what it holds, and constants; the language rules of
each layer; seams; `///` documentation; comments; tests; the extension
boundary and `run.sh`. [docs/testing.md](docs/testing.md) explains running
and writing tests. Read both before changing code: they bind agents exactly
as they bind people, and this file adds only how agents work.

## How agents work

- Work in your own worktree and branch. Agents own disjoint files: edit only
  the files your task gives you, coordinate interface changes with the
  integrator, and never commit on behalf of another agent. Do not push or
  rewrite shared branches unless asked.
- Delegate independent tasks when useful.
- Before renaming or moving an identifier or a file, read the `AGENTS.md` of
  every checkout nested in this tree: some keep track of engine names and
  paths and say what a rename or move must update. Do what it says in the
  same change, and run that checkout's checks before committing.
- Automated checks never open a window: run the game (`open-annihilation`,
  built by the `oa-game` target) headless, or with
  `SDL_VIDEO_DRIVER=dummy SDL_AUDIO_DRIVER=dummy`.
- Write commit messages as [CONTRIBUTING.md](CONTRIBUTING.md) describes; they
  follow the same wording rules as the sources.
- Before reporting completion, build the tree and run the relevant checks:
  for a change to sources, build files or documentation, the full ctest in a
  build configured with `OA_GAME_DIR` (see
  [docs/testing.md](docs/testing.md)), which includes the source checks
  (`doc-links`, `style-ratchet`, `format-check`, `licensing-check`). Report
  what ran, what skipped and what failed.

---
> Source: [open-annihilation/open-annihilation](https://github.com/open-annihilation/open-annihilation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
