## lassebenni-github-io

> This is a personal blog ([lasse.be](https://lasse.be)) built with **Hugo** and the **PaperMod** theme. It is hosted on GitHub Pages and deployed via GitHub Actions.

# Gemini Context: lassebenni.github.io

## Project Overview
This is a personal blog ([lasse.be](https://lasse.be)) built with **Hugo** and the **PaperMod** theme. It is hosted on GitHub Pages and deployed via GitHub Actions.

**Key Technologies:**
- **Hugo:** Static site generator.
- **PaperMod:** The active theme (configured in `config.toml`, submodule in `themes/PaperMod`).
- **Just:** Task runner for common commands.
- **Python:** Helper scripts in `scripts/` (e.g., for Medium conversion).
- **GitHub Actions:** CI/CD workflow defined in `.github/workflows/hugo-to-gh-pages.yml`.

## Building and Running
The project uses `just` as a command runner. Refer to `justfile` for the source of truth.

- **Start Development Server:**
  ```bash
  just serve
  ```
  Runs `hugo server -D` (drafts enabled) at `http://localhost:1313`.

- **Create New Post:**
  ```bash
  just post "my-post-slug"
  ```
  Creates a new page bundle at `content/posts/my-post-slug/index.md`.

- **Build for Production:**
  ```bash
  just build
  ```
  Runs `hugo --gc --minify` to generate the `public/` directory.

- **Clean Artifacts:**
  ```bash
  just clean
  ```

## Development Conventions

### Content Structure
- **Page Bundles:** Posts are organized as "leaf bundles" (directories containing `index.md` and assets like images).
  - Path: `content/posts/<slug>/`
  - Content: `index.md`
  - Images: Placed directly in the bundle directory.
- **Front Matter:** YAML format. Includes `title`, `date`, `draft`, `tags`, `cover` image settings.

### Customization & Architecture
- **Theme Overrides:** NEVER modify `themes/PaperMod` directly.
  - **Layouts:** Custom layouts are in `layouts/` (e.g., `layouts/partials/header.html`).
  - **Shortcodes:** Custom shortcodes (like `accordion`, `callout`) are in `layouts/shortcodes/`.
- **CSS:**
  - **Static:** `static/css/` contains custom CSS files (e.g., `custom.css`, `toc-sidebar.css`).
  - **Assets:** `assets/css/extended/` contains extended CSS (PaperMod feature) like `accordion.css`.
- **Configuration:** `config.toml` holds site-wide settings, menu configuration, and params (including Giscus comments and Buttondown newsletter).

### Scripts
- `scripts/convert_for_medium.py`: Helper to format posts for Medium.
- `scripts/add_cover.py`: Automation for cover images.

### Important Note
- The `README.md` mentions `themes/ink`, but the project is actually using **PaperMod**. Rely on `config.toml` and the file structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lassebenni)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lassebenni)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
