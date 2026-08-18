## altgallery

> Generate AltStore sources (`apps.json`) for IPA projects on GitHub.

# AltGallery — Agent Guide

## Project Goal

Generate AltStore sources (`apps.json`) for IPA projects on GitHub.

- Generator: Python **AltGen**, invoked via `uvx altgen`

## Comments

Write comments in English. Any comment you add or modify in code
(`config.toml`, Python, shell scripts, etc.) must be written in English —
never in Chinese or other languages.

## Reading JSON

Prefer `jq` to read or inspect JSON files (`apps.json`, `all-apps.json`), e.g.
`jq '.apps | length' PiliPlus/apps.json` or `jq '.apps[0]' PiliPlus/apps.json`.
Only read the full file when you actually need the whole content.

## Directory Layout

Each IPA gets its own folder under `apps/`, e.g. `apps/PiliPlus/`:

```
apps/<AppName>/
├── config.toml      # altgen configuration file
├── news.toml        # news image config (name, tagline, optional colors)
├── icon.png         # app icon
├── images/          # screenshots + news.png promo image
│   ├── home.png     # screenshots (referenced from [app] screenshots)
│   └── news.png     # generated promo image (referenced from [news] image_url)
└── apps.json        # generated AltStore source (gitignored), do not hand-edit
```

Each folder's `apps.json` is an independent AltStore source.

## Generating apps.json

`./update.sh` regenerates every app source and merges them into
`all-apps.json`. For a single app:

```bash
cd apps/<AppName> && uvx altgen -c config.toml   # note: config.toml, not config.json
```

altgen reads the GitHub Releases API; if rate-limited or the repo is private,
pass a token (`--token` > `GITHUB_TOKEN` env > `[github].token` in
config.toml).

**Rule: after ANY `config.toml` change, regenerate — never leave the two out
of sync, and never hand-edit `apps.json`.**

`apps/<AppName>/apps.json` and the repo-root `all-apps.json` are gitignored —
contributors never commit them. `.github/workflows/update.yml` regenerates and
commits them on every push to `master`, every 6 hours, and on manual
`workflow_dispatch`.

## Merging into all-apps.json

After all app sources are regenerated, merge them into the repo-root
`all-apps.json` (the "ultimate" AltStore source for the whole project).
The whole flow — regenerate every source, then merge — is scripted in
`./update.sh` (run from anywhere); prefer it over the individual commands.

```bash
uvx altgen merge -c assets/merge.toml apps/<AppName>/apps.json ...
```

Pass **every** `apps/<AppName>/apps.json` in the repo as an input. The merge
config lives in `assets/merge.toml`: merge mode only reads `[source]` (root
metadata: name, icon_url, tint_color) and `[output]` (`path = "../all-apps.json"`,
resolved against the config's directory → repo root).

**Rule: always run `./update.sh` (or the merge) whenever an app's `apps.json`
changes (regenerated, new app added, app removed) to verify the sources.
`apps.json` and `all-apps.json` are gitignored, so leave them untracked — the
workflow regenerates and commits them. To refresh them immediately, run the
`Update all-apps.json` workflow manually (`workflow_dispatch`).**

## Generating News Images

Each app has a shared `news.png` ("NEW UPDATE" — icon, name, tagline)
referenced by all its news entries via `[news] image_url`. Re-render every
app's image (from each app's `news.toml`: `name`, `tagline`, optional
`[colors]`) with:

```bash
./update_news.sh
```

or a single app with `.venv/bin/python3 templates/render_news.py --out
apps/<AppName>`.
Unset colors are auto-derived from `config.toml` tints and the icon's dominant
color (needs the uv venv set up once: `uv venv && uv pip install -r
requirements.txt`; Pillow is the only image dependency — no external
rasterizer). The rendered `apps/<AppName>/images/news.png` stays in the working
tree — the URL in `config.toml` `[news] image_url` already points at it. Do
not auto-commit it.

## Icon Color Sampling (PIL)

Sampling a dominant color from an icon — for a new app's `tint_color`, or the
news background derived by `render_news.py` — is already implemented in
`templates/render_news.py` → `extract_icon_color()` (it downsamples the icon
with Pillow and buckets the pixels). Just call it:

```bash
PYTHONPATH=templates .venv/bin/python3 -c \
  "from render_news import extract_icon_color; from pathlib import Path; \
   print(extract_icon_color(Path('apps/<AppName>/icon.png')))"
```

**Pillow/PIL is installed only in the project venv (`.venv/`), not in the
system Python** — any command that imports PIL (`render_news.py`,
`extract_icon_color`) must run with the venv's interpreter:
`.venv/bin/python3` (or `source .venv/bin/activate` first). The bare system
`python3` will raise `ModuleNotFoundError: No module named 'PIL'`.

When a new app lacks an official tint, use the result as `[app] tint_color`
(brand color) — but eyeball the icon first: a multi-color or pale icon may
not have an obvious single brand color, so the sample needs human
confirmation.

## Adding a New App

End-to-end procedure lives in the **add-app** skill — invoke it with
`/add-app` (it also auto-loads when you ask to add a new app). It covers
extracting fields from the project's own AltStore source (when one exists),
the folder/config/icon/news setup, tint sampling, `apps.json` generation,
merge, and README update. The reference sections below ([Icon Color Sampling
(PIL)](#icon-color-sampling-pil), [Generating News
Images](#generating-news-images), [Merging into
all-apps.json](#merging-into-all-appsjson)) remain authoritative for the
shared details.

---
> Source: [bebound/AltGallery](https://github.com/bebound/AltGallery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
