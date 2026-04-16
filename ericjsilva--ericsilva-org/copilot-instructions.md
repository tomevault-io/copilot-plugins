## ericsilva-org

> This project is a personal website for Eric Silva, built using the [Hugo](https://gohugo.io/) static site generator and the [Congo](https://jpanther.github.io/congo/) theme.

# Gemini CLI - Project Context: ericsilva.org

This project is a personal website for Eric Silva, built using the [Hugo](https://gohugo.io/) static site generator and the [Congo](https://jpanther.github.io/congo/) theme.

## Project Overview

- **Technologies**: Hugo (Extended version recommended), Hugo Modules, Docker, Just (task runner).
- **Theme**: [Congo](https://github.com/jpanther/congo) (managed via Hugo modules).
- **Architecture**: A standard Hugo project structure with content organized into several distinct sections:
    - `posts/`: General blog posts.
    - `recipes/`: Personal recipes with stories.
    - `commissioner-minutes/`: Scouting-related leadership content.
    - `woodworking/`, `laser/`, `photography/`: Project-based portfolios and logs.

## Building and Running

The project uses `just` as a command runner to simplify common tasks.

- **Local Development**:
  ```bash
  just serve
  ```
  This runs a Hugo development server inside a Docker container (using `peaceiris/hugo:latest-mod`), accessible at `http://localhost:1313`. It enables draft and future content building by default.

- **Image Optimization**:
  ```bash
  just convert-images <directory>
  ```
  This script (`scripts/convert_images.sh`) uses ImageMagick to convert `IMG_*.png` files in the specified directory to WebP format. Note: The current script renames images using a `printer-stand-XX.webp` pattern.

- **Production Build** (Manual/Inferred):
  ```bash
  hugo --minify
  ```

## Development Conventions

### Content Organization
- **Filenames**: Use lowercase with hyphens (e.g., `my-new-post.md`).
- **Front Matter**: Always include `title`, `date`, `tags`, and `categories`. Use `draft: true` for works in progress.
- **Recipes**: Should combine personal storytelling with clear, tested instructions, including prep/cook times and serving info.

### Image Guidelines
- **Format**: All new images **must** be in WebP format.
- **Quality**: Target 85% quality during conversion.
- **Storage**: Store images in `static/img/` or within the leaf bundle directory of the post (e.g., `content/woodworking/printer-stand/`).

### Styling
- Custom styles should be added to `assets/scss/custom.scss` or `assets/css/custom.css`.
- The project uses Hugo Pipes for asset processing.

## Key Files & Directories
- `config/_default/`: Contains site configuration (`hugo.toml`, `params.toml`, etc.).
- `content/`: All site content in Markdown.
- `layouts/`: Custom HTML templates and shortcodes (e.g., `lightbox-gallery.html`).
- `archetypes/`: Templates for new content types (`laser.md`, `woodworking.md`, etc.).
- `.github/instructions/`: Detailed guidelines for specific content areas (Hugo, Scouting, Recipes, etc.). Refer to these when creating new content.
- `justfile`: Defines the project's task-based workflow.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ericjsilva) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
