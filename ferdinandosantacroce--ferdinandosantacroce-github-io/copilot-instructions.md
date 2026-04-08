## ferdinandosantacroce-github-io

> This is the personal blog of **Ferdinando Santacroce**, built with **Hugo** and the **Stack theme v3**. It is a bilingual site (English/Italian) hosted at `ferdinandosantacroce.it` and deployed via GitHub Pages.

# GEMINI.md - Project Context & Learnings

# ABOUTME: Personal bilingual blog (Hugo + Stack Theme v3)
# ABOUTME: Strict bilingual parity (En/It), CI/CD deployment, content-first architecture

## Project Overview
This is the personal blog of **Ferdinando Santacroce**, built with **Hugo** and the **Stack theme v3**. It is a bilingual site (English/Italian) hosted at `ferdinandosantacroce.it` and deployed via GitHub Pages.

## Core Principles (The Constitution)

1.  **Bilingual Content Parity**: Every piece of content MUST exist in both English (`index.en.md`) and Italian (`index.it.md`). No placeholders allowed.
2.  **Content-First Structure**: Strict adherence to Hugo conventions. Posts in `content/post/[slug]/`, pages in `content/page/[name]/`.
3.  **Theme Integrity**: Do not modify theme files directly. Use `assets/css/custom.css` for overrides. Updates are managed via Hugo modules.
4.  **Automated Deployment**: GitHub Actions is the only deployment path to `main`. No manual deployments.
5.  **Simplicity & Focus**: Professional content (Software, Agile, Coaching). No unnecessary JS, no comments system, focus on performance.

## Tech Stack & Tools

-   **Generator**: Hugo
-   **Theme**: [Hugo Theme Stack v3](https://github.com/CaiJimmy/hugo-theme-stack) (managed via Go modules)
-   **Icons**: [Tabler Icons](https://tablericons.com/) (SVG)
-   **Package Manager**: Go (for Hugo modules)
-   **CI/CD**: GitHub Actions

## Project Structure

-   `config/_default/`: Modular TOML configuration (params, menu, permalinks, etc.).
-   `content/post/`: Individual directories per post, containing bilingual Markdown and media.
-   `content/page/`: Static pages (About, Talks, etc.).
-   `assets/icons/`: Custom SVG icons. Attributes must be: `width="24" height="24" stroke-width="2" stroke="currentColor"`.
-   `assets/css/custom.css`: Project-specific styling.

## Essential Commands

| Task | Command |
| :--- | :--- |
| **Local Dev** | `hugo server -D` (includes drafts) |
| **Build** | `hugo --minify --gc` |
| **Update Theme** | `hugo mod get -u github.com/CaiJimmy/hugo-theme-stack/v3 && hugo mod tidy` |

## Workflows

### Creating a Post
1.  Create `content/post/[slug]/`.
2.  Create `index.en.md` and `index.it.md`.
3.  Include front matter: `title`, `date`, `categories`, `tags`, `image`.
4.  Remove `draft: true` when ready.

### Adding Icons
1.  Download from Tabler Icons.
2.  Save to `assets/icons/`.
3.  Normalize SVG attributes to project standards.

## Status & TODOs
-   **Recent**: Updated Hugo version in GitHub Actions (001-update-hugo).
-   **Pending**: Italian date formatting.
-   **Pending**: Auto-linking articles across languages.

## Recent Changes
- 003-jekyll-migration: Added Hugo (latest compatible with Theme Stack v3), Go Modules + Hugo Theme Stack v3
- 003-jekyll-migration: Added Hugo (latest compatible with Theme Stack v3), Go Modules + Hugo Theme Stack v3
- 001-talk-page-layout: Added Hugo (managed via Go modules), CSS + Hugo Theme Stack v3

## Active Technologies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ferdinandosantacroce)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ferdinandosantacroce)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
