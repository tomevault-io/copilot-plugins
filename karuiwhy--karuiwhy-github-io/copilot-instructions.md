## karuiwhy-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal academic website for WU Haoyang, hosted on GitHub Pages at `https://karuiwhy.github.io/`. Static HTML/CSS site with a blog system that dynamically renders Markdown posts with LaTeX math support.

## Development Commands

**Local preview** - A static server is required (file:// protocol blocks fetch):
```bash
python -m http.server 8000
```
Then open `http://localhost:8000` (or `http://localhost:8000/blog.html` for the blog).

**Deploy** - Push to main branch; GitHub Pages auto-deploys:
```bash
git add -A && git commit -m "Your message" && git push
```

## Architecture

**Pages:**
- `index.html` - Homepage with About, Education, Research Interests, Contact sections
- `blog.html` - Blog page that dynamically loads posts from `posts/posts.json`

**Blog System:**
- Posts are Markdown files in `posts/` directory
- Metadata stored in `posts/posts.json` (array order = display order)
- Each post entry: `id`, `title`, `date`, `mdPath` (required), `excerpt` and `tags` (optional)
- Uses `marked.js` for Markdown rendering and `KaTeX` for LaTeX math
- Math delimiters: `\( \)` or `$...$` for inline, `$$ $$` or `\[ \]` for block

**Adding a new post:**
1. Create `posts/<post-id>.md` with Markdown content
2. Add entry to `posts/posts.json` with `mdPath: "posts/<post-id>.md"`

**Styling:**
- Single stylesheet `styles.css` with warm orange/cream color scheme
- Uses Inter font from Google Fonts
- Responsive design with breakpoints at 768px and 480px

**Dependencies (CDN):**
- Google Fonts (Inter)
- marked.js - Markdown parsing
- KaTeX - LaTeX math rendering

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KaruiWHY)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KaruiWHY)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
