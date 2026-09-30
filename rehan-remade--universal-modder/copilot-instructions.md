## universal-modder

> This repo is a game-modding toolkit for coding agents. When someone opens Claude Code (or Codex, Cursor...)

# universal-modder

This repo is a game-modding toolkit for coding agents. When someone opens Claude Code (or Codex, Cursor...)
here, they almost always want to **mod a game**. Start with the `mod-any-game` skill
(`skills/mod-any-game/SKILL.md`) and follow its loop: intake → recon → route → lab → source of truth →
vertical slice → assets → verify in game → showcase → publish.

## Tools
- `bin/um` is the CLI. A SessionStart hook in `.claude/settings.json` puts it on PATH, so `um ...` works.
  Every group has `--help`:
  - `scan`: installed games, engine, anti-cheat, loaders, saves, routes
  - `fal`: sprites, textures, PBR, 3D, rigs, SFX, music, voice, video
  - `sprite`, `render3d`: art → engine-ready frames
  - `win`: launch / screenshot / input / record on Windows, also from WSL
  - `video`: contact sheets, EDL-based showcase edits, muxing
  - `backup`: snapshot / restore saves
  - `publish`: pre-release lint
- The fal MCP server is configured in `.mcp.json` (needs `FAL_KEY`).
- Skills: `skills/*/SKILL.md`. Engine playbooks: `skills/mod-any-game/references/engines/`.
- Worked examples: `examples/terraria-tmodloader`, `examples/aoe2-de-civ`.

## Rules (full list in skills/mod-any-game/references/safety.md)
- Only games the user owns, single-player/offline. Never touch online clients with anti-cheat; never bypass
  anti-cheat, DRM or ownership checks.
- `um backup` saves before modded launches. Never commit or ship game files or decompiled code; keep
  decompiles outside the repo.
- Kill processes by exact PID (`um win kill`), never `pkill -f`. Ask before driving the user's
  mouse/keyboard, installing loaders into game folders, changing the registry, or publishing.
- Keep a `MODLOG.md` journal in the mod's working folder.

## Working on the toolkit itself
- Python 3.10+, deps Pillow + numpy (`uv` handles them via `bin/um`). Code lives in `um/`, one module per
  CLI group, each with `register(sub)` and a docstring that doubles as its `--help`.
- `tools/win/*.ps1` embed C# 5 (Windows PowerShell 5.1's compiler): no string interpolation, no `out var`,
  no expression-bodied members.
- Test changes against real games when possible (`um scan <installed game>`, `um video compile` on a small
  EDL, `um sprite` on `examples/*/assets`). Keep `um --help` output accurate.

---
> Source: [rehan-remade/universal-modder](https://github.com/rehan-remade/universal-modder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
