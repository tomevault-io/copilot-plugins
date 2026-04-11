## p5js-workshop

> **Goal:** Create a fast, static website to host a programming workshop for children to learn creative programming with p5js.

# Project: Kids Programming Workshop
**Goal:** Create a fast, static website to host a programming workshop for children to learn creative programming with p5js.
**Tech Stack:** Astro (Static Site Generator), Tailwind CSS (styling), GitHub Pages (hosting).

## Design Philosophy
- **Audience:** Children (ages 12-16).
- **Visuals:** Bright, high contrast, large typography, fun but readable.
- **Navigation:** Simple "Next/Previous" buttons for lessons. Sidebar for easy jumping between modules and lessons. Tasks grouped by difficulty (Easy, Medium, Advanced)
- **Code Snippets:** Must use syntax highlighting and copy buttons. 
- **Workflow:** Maintain a readme.md file with instruction on how to run localy or deploy to github pages

## File Structure Rules
- `src/content/workshop/`: All lesson content goes here as Markdown (.md) files.
- `src/layouts/WorkshopLayout.astro`: The main wrapper for lesson pages (includes sidebar + content).
- `src/pages/index.astro`: The landing page/welcome screen.
- `public/images/`: All screenshots and diagrams.

## Key Features Needed
1.  **Sidebar:** Auto-generated from the files in `src/content/workshop`.
2.  **Deploy:** GitHub Actions workflow for GitHub Pages.
3.  **Content** All content should be in norwegian, inculding site, navigation, and lessons

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mortenaa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mortenaa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
