## lancewfisher-v2

> Personal portfolio and brand site for Lance W. Fisher. Static HTML/CSS/JS, no build step.

# lancewfisher-v2

Personal portfolio and brand site for Lance W. Fisher. Static HTML/CSS/JS, no build step.

## Stack
- **HTML/CSS/JS** — vanilla, no frameworks or bundlers
- **Fonts**: Google Fonts (Cormorant Garamond, Inter, JetBrains Mono)
- **Domain**: lancewfisher.com (canonical), one-three.net (brand variant)
- **Deploy**: Static files in `deploy/` directory

## How to Run
```
# Serve deploy/ for live preview (launch.json configured for port 8077):
python -m http.server 8077 --directory deploy

# Or serve root for development:
python -m http.server 8070 --directory .
```

## Source vs Deploy

**`deploy/` is the website and the authoritative source of truth for all content.** It is uploaded in full to Hostinger and is what goes live at lancewfisher.com. All website work must be done in `deploy/`.

### Sync Direction (NON-NEGOTIABLE)

The only safe sync direction is: **`deploy/` → root** (never root → `deploy/`).

- **Edit files in `deploy/` directly.** This is the primary working location.
- **After editing, sync back to root** to keep the repository consistent:
  ```bash
  cp deploy/styles.css styles.css
  cp deploy/main.js main.js
  cp deploy/home.html home.html
  cp deploy/index.html index.html
  # etc. for any changed files
  ```
- **NEVER run `build-deploy.sh`** — it wipes and rebuilds `deploy/` from root, destroying all deploy-side edits and the live site state.
- **NEVER copy root files into `deploy/`** — root files are a mirror/backup and are always older than or equal to deploy. Copying root → deploy overwrites recent work with stale content.
- **NEVER overwrite a deploy file without first comparing line counts or diffing** — if the source and deploy versions differ, `deploy/` wins unless you have explicit confirmation the source is newer.
- The preview server always serves from `deploy/`: `python -m http.server 8077 --directory deploy`

### Bloodlines Content Rule

Bloodlines now uses `D:/ProjectsHome/FisherSovereign/lancewfisher-v2/bloodlines/` as the main working root.

- `D:/ProjectsHome/Bloodlines/` is a compatibility alias to that root.
- `deploy/bloodlines/` is no longer a separate Bloodlines project root and should not be treated as one.
- Future Bloodlines canon, docs, tooling, browser runtime work, and Unity work should happen in `bloodlines/`.
- If Bloodlines content must be exposed under `deploy/`, treat that as a compatibility surface, not as the source of truth.

## Structure
```
index.html              — Main portfolio page (lancewfisher.com)
resume.html             — Resume page
one-pager.html          — One-page summary
404.html                — Custom 404 page
styles.css              — Extracted main CSS (from index.html)
main.js                 — Extracted main JS (from index.html)
concepts/               — Source concept HTML files (12 themes)
noco/                   — NoCo businesses directory (app.js, businesses.js, styles.css)
harmony/                — Serenity Medspa Demo embedded sub-page
dashboard/              — Dashboard sub-page
jumpquest/              — Jump Quest game sub-page
thumbs/                 — Thumbnail images
_originals/             — Original high-res assets
```

## Design Notes
- Dark aesthetic with gold accents (CSS custom properties)
- Sections: Hero, About, Now, Ventures, Projects, One-Three brand, Contact
- Responsive with mobile nav toggle
- Multiple sub-sites served from subdirectories
- **Two themes**: dark (default) and light — both must work. Toggle is top-right pill button.

## Dual-Mode Rule (NON-NEGOTIABLE)
Every CSS or UI change must be verified in BOTH dark and light mode before marking done.

Quick check in browser console (paste and run):
```js
// Switch to light
document.documentElement.setAttribute('data-theme','light');
// Switch back to dark
document.documentElement.removeAttribute('data-theme');
```

## CSS Convention — New Color Rules
**Never hardcode `rgba(240, 235, 224, ...)` for text.** Use the `--fg` variable instead:

```css
/* ONE rule covers both themes automatically: */
.my-element { color: rgba(var(--fg), 0.65); }

/* Dark mode resolves to: rgba(240, 235, 224, 0.65)  ← near-white on dark bg */
/* Light mode resolves to: rgba(26, 26, 28, 0.65)    ← near-black on light bg */
```

`--fg` is defined in `:root` (dark) and `[data-theme="light"]` in `deploy/styles.css`.
For `ftda/ftda.css`, also add `--fg` to its `:root` block if writing new color rules there.

**Alpha guidance for readable text in both modes:**
- `0.85+` — primary text (headings, body)
- `0.65–0.75` — secondary text (descriptions, labels)
- `0.45–0.55` — muted/metadata text
- Below `0.40` — decorative only, do not use for readable content

**Do NOT use `rgba(var(--fg), ...)` for backgrounds** — use the existing `--warm-white-ghost` etc. variables or explicit values for backgrounds.

---
> Source: [lance-fisher/lancewfisher-v2](https://github.com/lance-fisher/lancewfisher-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
