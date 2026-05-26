## spiritual-vibes

> Static website ("spiritual-vibes") consisting of a few HTML files with client-side interactivity. No build system, no package manager, no backend.


## Project

Static website ("spiritual-vibes") consisting of a few HTML files with client-side interactivity. No build system, no package manager, no backend.

## Deployment

Deployed on **GitHub Pages**. Everything must work as plain static files served over HTTP(S) — and ideally also when opened directly via `file://` for quick local checks. No server-side logic, no API calls to a private backend, no build artifacts. External resources (Phaser, fonts, etc.) load from public CDNs.

The website makes interactive a document from the Pope to make it easier to dijest.
There is a section to drill down on the encyclic by topic/chapter and a small videogame.
Priority on fun, understanding, respectful but critical thinking.

# Style

Fun, playful, focus on edutainment. Simple language, target is low scolarization adults.

## Running locally

```bash
python3 -m http.server 8000
```

## Layout

- `html/` — the website (serve this folder).
  - `html/assets/` — images and other static assets.
  - `html/css/` — stylesheets.
- `encyclic/` — source material (original encyclical HTML from vatican.va). Do not serve or modify; read-only reference.

## Conventions

- Keep everything client-side (HTML/CSS/JS).
- No build step — edits to files are reflected on browser reload.

---
> Source: [pieroit/spiritual-vibes](https://github.com/pieroit/spiritual-vibes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
