## agent-skin-hub

> `agent-skin-hub` is an **open Codex Desktop skin catalog**. It ships wallpapers + `theme.json` packs under `presets/`, plus concept ads under `docs/ads/`.

# Repository Guidelines (for Codex / agents)

## What this repo is

`agent-skin-hub` is an **open Codex Desktop skin catalog**. It ships wallpapers + `theme.json` packs under `presets/`, plus concept ads under `docs/ads/`.

It is **not** ProvDex source code. Applying a skin uses a reversible local CDP engine; install it on demand via `scripts/apply-hub-skin.sh` (does not patch the official app).

When a user pastes this GitHub URL into Codex, **execute the apply workflow** from `SKILL.md` — do not stop at explaining.

## Layout

| Path | Purpose |
|------|---------|
| `presets/preset-*/` | Installable packs (`theme.json`, `background.jpg`, `SOURCE.md`) |
| `catalog.json` | Generated index for ProvDex / website / agents |
| `gallery.json` | Concept ads + optional `presetId` links |
| `docs/ads/` | Marketing full-window shots — **never** import as wallpaper |
| `docs/previews/` | Optional live thumbs |
| `scripts/apply-hub-skin.sh` | End-user / Codex one-shot apply |
| `SKILL.md` | Codex skill entry |
| `CODEX-PROMPT.md` | Copy-paste deploy prompt |

## Agent commands

```bash
# List skins
curl -fsSL https://cdn.jsdelivr.net/gh/Chiody/agent-skin-hub@main/catalog.json | \
  python3 -c 'import json,sys; c=json.load(sys.stdin); [print(p["id"], p["name"]) for p in c["presets"]]'

# Apply (macOS)
curl -fsSL https://cdn.jsdelivr.net/gh/Chiody/agent-skin-hub@main/scripts/apply-hub-skin.sh | \
  bash -s -- preset-trial-yuexin-miao

# Or from a local clone
bash scripts/apply-hub-skin.sh preset-trial-rose-soft
bash scripts/apply-hub-skin.sh --list

# Rebuild indexes after adding presets
node scripts/build-catalog.mjs
node scripts/build-gallery.mjs
node scripts/validate-preset.mjs presets/preset-your-slug
```

## Safety

- Never modify official `.app` / `app.asar` / signatures / API keys / Base URLs.
- Only import UI-free 16:9 wallpapers from `presets/*/background.jpg`.
- CDP must stay loopback-only (handled by the skin engine scripts).

## Style

Keep preset ids `preset-<kebab-slug>`. Prefer light readable palettes for busy wallpapers. Document rights in each `SOURCE.md`.

---
> Source: [Chiody/agent-skin-hub](https://github.com/Chiody/agent-skin-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
