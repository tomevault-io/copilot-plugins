## silent-autoplay-video-worpress

> - **Source (GitHub repo):** `/Users/mortenbrekkestensland/Documents/GitHub/silent-autoplay-video-worpress/`

# Silent Autoplay Video — Developer Notes (local only, gitignored)

## Project Locations

- **Source (GitHub repo):** `/Users/mortenbrekkestensland/Documents/GitHub/silent-autoplay-video-worpress/`
- **WordPress site:** `schedulecontrol.local` at `/Users/mortenbrekkestensland/Local Sites/schedulecontrol/`
- **Plugin in WP:** `/Users/mortenbrekkestensland/Local Sites/schedulecontrol/app/public/wp-content/plugins/silent-autoplay-video` — this is a **symlink** pointing to the GitHub repo, so edits here are instantly live on the site.

## Workflow

- Always work in the GitHub repo folder. Changes are reflected on the WordPress site immediately via the symlink.
- Do NOT copy files between the two locations — the symlink handles it.
- When updating the plugin version, update it in three files: `silent-autoplay-video.php`, `assets/js/editor.asset.php`, and `assets/js/frontend.asset.php`.

## Build Scripts

- `build-zip-mac.command` — double-click on Mac to create `silent-autoplay-video.zip` for WordPress upload
- `build-zip-pc.bat` — double-click on Windows to create the same zip
- Both scripts exclude `.git`, `.gitignore`, `.DS_Store`, `README.md`, `AGENTS.md`, and themselves from the zip.

## Architecture

- No build step — plain JS (`wp.element.createElement`), no JSX, no Node.js.
- Dynamic block: `save()` returns `null`, PHP `render.php` handles frontend HTML.
- `block.json` registers everything: editor script, view script, styles, render template.
- Safari canvas fallback in `frontend.js`: tries native `video.play()`, retries after setting attributes, falls back to `requestAnimationFrame` drawing, then `setInterval` manual `currentTime` advance as last resort.
- Interaction listener (click/touchstart/keydown — NOT scroll) switches back to native playback.

---
> Source: [MortenBrekkeStensland/silent-autoplay-video-worpress](https://github.com/MortenBrekkeStensland/silent-autoplay-video-worpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
