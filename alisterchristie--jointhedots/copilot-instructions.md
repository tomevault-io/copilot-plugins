## jointhedots

> JoinTheDots is a Delphi FireMonkey desktop game based on dots-and-boxes.

# JoinTheDots Agent Notes

## Project Overview

JoinTheDots is a Delphi FireMonkey desktop game based on dots-and-boxes.
Players claim lines between dots; completing a cell scores a point and grants
another turn. The game supports local two-player play, an optional AI opponent,
multiple board sizes, and square or hexagon board styles.

## Project Structure

- `JoinTheDots.dpr` - application entry point.
- `JoinTheDots.dproj` - RAD Studio project file.
- `formJoinTheDots.pas` - FireMonkey form, drawing, mouse input, UI controls,
  and AI turn timer.
- `formJoinTheDots.fmx` - main form resource.
- `uJoinTheDotsGame.pas` - game model: board generation, edges/cells, scoring,
  turn rules, and AI move selection.
- `README.md` - user-facing project notes.

## Development Notes

- This is an FMX project. Prefer compiling through RAD Studio/Kai when available
  so IDE state and unsaved buffers are respected.
- Treat the RAD Studio editor buffer as canonical for files open in the IDE.
- Keep UI drawing/input concerns in `formJoinTheDots.pas`.
- Keep board topology, rules, scoring, and AI logic in `uJoinTheDotsGame.pas`.
- Avoid mixing generated build output or IDE local/cache files into source
  control.

## Git Policy

Do not commit or push changes unless the user explicitly asks for a commit or
push. It is fine to prepare changes and report the working tree status, but leave
Git history and remotes untouched without explicit instruction.

---
> Source: [alisterchristie/JoinTheDots](https://github.com/alisterchristie/JoinTheDots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
