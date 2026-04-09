## pankajcoding-github-io

> portfolio.html  ← editable source (gitignored, local only)

# Portfolio Build Workflow

## Architecture
```
portfolio.html  ← editable source (gitignored, local only)
     ↓  npm run build
index.html      ← minified + class-obfuscated output (committed to git → GitHub Pages)
```

## After Any Change to portfolio.html

Run this single command:
```bash
npm run build
```

That's it. It will:
1. Replace all 103 CSS class names with Harry Potter spell/character names
2. Minify HTML, inline CSS, and inline JS
3. Output `index.html` with the wizard ASCII art comment at the top

Then commit:
```bash
git add index.html && git commit -m "your message"
```

## What the Build Does (build.js)
- Reads `portfolio.html`
- Obfuscates class names via `CLASS_MAP` (e.g. `hero-title` → `bellatrix`, `loading` → `potter`)
- Passes through `html-minifier-terser` with:
  - `collapseWhitespace`, `removeComments`, `minifyCSS`, `minifyJS` (mangle + compress)
- Prepends the wizard ASCII art HTML comment
- Writes to `index.html`

## Class Name Mapping
Defined in `build.js` → `CLASS_MAP`. If you add a new class to `portfolio.html`:
1. Add it to `CLASS_MAP` in `build.js` with a Harry Potter word
2. Run `npm run build`

## Files
| File | Purpose |
|------|---------|
| `portfolio.html` | Readable source — edit this |
| `index.html` | Build output — never edit directly |
| `build.js` | Build script + class map |
| `package.json` | Defines `npm run build` |
| `.gitignore` | Ignores `portfolio.html` and `node_modules/` |

## Never
- Edit `index.html` directly — it gets overwritten on every build
- Commit `portfolio.html` — it's gitignored intentionally

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pankajcoding)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pankajcoding)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
