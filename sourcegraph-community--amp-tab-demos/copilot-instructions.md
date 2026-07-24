## amp-tab-demos

> - No build, test, or lint commands - this is a static HTML demo site

# AGENTS.md

## Commands
- No build, test, or lint commands - this is a static HTML demo site
- Deploy: GitHub Actions automatically deploys to GitHub Pages on push to main
- Local preview: Open `index.html` directly in browser or use `python3 -m http.server`

## Architecture
- **Static demo site**: Single HTML page with embedded JavaScript and CSS
- **Structure**: `index.html` (main page), `demos/` (video files), `.github/workflows/pages.yml` (auto-deploy)
- **No backend**: Pure frontend with embedded demo data in JavaScript
- **Videos**: MP4 files stored in demo subdirectories alongside README.md files

## Code Style
- **HTML**: Semantic structure, Tailwind CSS classes, mobile-responsive design
- **JavaScript**: ES6+ features, embedded in HTML, uses template literals for dynamic content
- **CSS**: Tailwind utility classes with custom styles in `<style>` tag
- **Files**: Consistent naming (amp_tab.mp4, cursor_tab.mp4), markdown READMEs in each demo folder
- **Data**: Demo metadata embedded in `DEMOS_DATA` object with videos, tags, and reproduction info

## Adding Demos
1. Create new folder in `demos/` with videos and README.md
2. Add entry to `DEMOS_DATA` object in index.html
3. Commit and push - GitHub Actions handles deployment

---
> Source: [sourcegraph-community/amp-tab-demos](https://github.com/sourcegraph-community/amp-tab-demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
