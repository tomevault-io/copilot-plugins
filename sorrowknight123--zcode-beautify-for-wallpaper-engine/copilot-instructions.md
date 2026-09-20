## zcode-beautify-for-wallpaper-engine

> This repository is two things at once. Pick your path by what the user asked for:

# AGENTS.md — start here

This repository is two things at once. Pick your path by what the user asked for:

| The user asked to… | Read | What you'll do |
|---|---|---|
| **install the beautify plugin into their ZCode** (e.g. "帮我安装美化插件", "install this into my ZCode", "make my ZCode pretty") | [INSTALL-FOR-AI.md](INSTALL-FOR-AI.md) | Guide them through the marketplace UI (30 s), or install it end-to-end yourself if you have shell access, then launch ZCode with the debug port and apply a wallpaper |
| **beautify some other Electron desktop app** (e.g. "给这个 Electron 应用加壁纸和动态配色", "build a theme tool for app X") | [skill-pack/SKILL.md](skill-pack/SKILL.md) | Build a small local tool from the skill: CDP launch + inject, MD3 color extraction, wallpaper layer, optional live-tuning API |

## One-line facts

- Repo: `https://github.com/Logocceai/zcode-beautify` — plugin manifest at the root (`marketplace.json` + `.zcode-plugin/plugin.json`), so the repo itself is a single-plugin marketplace.
- The plugin beautifies the **ZCode desktop client** via CDP injection (`dist/cli.js`, prebuilt, zero install steps). It never modifies app files.
- `skill-pack/` is platform-agnostic (any Electron app, any AI agent) and versioned together with the plugin.

## Rules of thumb

- The end user is non-technical by default: prefer the marketplace-UI path from `INSTALL-FOR-AI.md` unless they explicitly asked you to do it for them and you have shell access.
- Never modify ZCode's installation files; everything is user-level (`~/.zcode/...`) or runtime injection.
- CDP injection requires the app to run with `--remote-debugging-port`. The bundled `launch` command handles that safely (it refuses to race the single-instance lock).

---
> Source: [sorrowKnight123/zcode-beautify-for-wallpaper-engine](https://github.com/sorrowKnight123/zcode-beautify-for-wallpaper-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
