## litapp-blog

> Litapp Blog is a TiddlyWiki-based static site generator (SSG). It leverages TiddlyWiki's flexible data structure and templating system to author content and render it into a modern, static blog.

# Litapp Blog

Litapp Blog is a TiddlyWiki-based static site generator (SSG). It leverages TiddlyWiki's flexible data structure and templating system to author content and render it into a modern, static blog.

## Project Overview

- **Core Technology:** [TiddlyWiki 5](https://tiddlywiki.com/)
- **Plugin:** The main logic resides in `plugins/litapp/blog`, which includes templates, styles, and configuration for the blog.
- **Static Rendering:** Uses TiddlyWiki's `--render` command to generate HTML files from tiddlers.
- **Styles:** Uses `mvp.css` as a base for a minimalist and responsive design, with custom extensions in `plugins/litapp/blog/styles/`.

## Directory Structure

- `plugins/litapp/blog/`: The TiddlyWiki plugin source code.
  - `templates/`: TiddlyWiki templates (`.tid` files) for various page types (home, entry, tag, gallery).
  - `styles/`: CSS stylesheets.
  - `build.txt`: Definition of the TiddlyWiki render commands for building the blog.
- `editions/demo/`: A demonstration TiddlyWiki edition containing sample content.
  - `tiddlers/`: Markdown and TiddlyWiki text files containing blog posts and metadata.
- `scripts/`: Node.js utility scripts for development and maintenance.
- `package.json`: Project dependencies and script definitions.

## Building and Running

### Development Environment

To start the development environment, which includes a TiddlyWiki server and a live-reloading static blog preview:

```bash
yarn dev
```

- TiddlyWiki will be available on its default port (usually 8080).
- The static blog preview will be served on port `9021`.

### Build Commands

- **Build Static Blog:** Generates the static HTML files in `editions/demo/output/static/`.
  ```bash
  yarn build-blog
  ```
- **Build TiddlyWiki Index:** Builds the standalone `index.html` for the demo edition.
  ```bash
  yarn build
  ```

### Other Scripts

- **Format Code:** Uses Prettier to format the codebase.
  ```bash
  yarn prettier
  ```
- **Set Plugin Version:** Updates the plugin's version to match `package.json`.
  ```bash
  yarn set-plugin-version
  ```

## Development Conventions

- **TiddlyWiki Templates:** Most of the UI logic is written in TiddlyWiki's wikitext and filter expressions within the `.tid` files in `plugins/litapp/blog/templates/`.
- **Static Assets:** Images and other static assets are managed within the TiddlyWiki environment and exported during the build process.
- **CSS:** Custom styles should be added to `plugins/litapp/blog/styles/custom.css`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carlo-colombo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/carlo-colombo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
