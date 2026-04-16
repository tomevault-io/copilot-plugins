## optikalbahari

> This project is a static website for **Optikal Bahari**, an optical store based in Kemayoran,

# Project Context: Optikal Bahari

## Project Overview

This project is a static website for **Optikal Bahari**, an optical store based in Kemayoran,
Jakarta. It is built using **Jekyll**, a Ruby-based static site generator. The site provides
information about eye exams, glasses, contact lenses, and includes a blog with articles related to
eye health and eyewear trends.

The codebase is customized from the "Clean Blog" Jekyll theme, utilizing Bootstrap for styling and
Liquid for templating. It features extensive optimizations for performance, including asset
compression (Brotli/Gzip) and minification.

## Tech Stack

- **Static Site Generator:** Jekyll (v4.4.1)
- **Language:** Ruby, Liquid (Templating), SCSS/Sass (Styling)
- **Framework:** Bootstrap (inferred from theme origin)
- **Asset Management:** Cloudinary (for images), local assets in `assets/`
- **Key Plugins:**
  - `jekyll-paginate-v2` (Pagination)
  - `jekyll-seo-tag` (SEO)
  - `jekyll-sitemap` (Sitemap generation)
  - `jekyll-minifier` (HTML/JS/CSS minification)
  - `jekyll-archives` (Category/Tag archives)

## Building and Running

### Prerequisites

- Ruby (check version with `ruby -v` or `.ruby-version`)
- Bundler (`gem install bundler`)

### Development Commands

- **Install Dependencies:**
  ```bash
  bundle install
  ```
- **Start Local Server:**

  ```bash
  bundle exec jekyll serve
  ```

  - With auto-reload: `bundle exec jekyll serve --livereload`
  - With drafts: `bundle exec jekyll serve --drafts`

- **Clean Build Artifacts:**
  ```bash
  bundle exec jekyll clean
  ```

### Production Build

To build the site for production (enables optimizations and minification):

```bash
JEKYLL_ENV=production bundle exec jekyll build
```

_Note: The `_config.yml` defines a custom build command that also runs a post-build script:_

```bash
bundle exec jekyll build && bash _scripts/post-built/post-build.sh
```

## Project Structure

- **`_config.yml`**: Main configuration file (site settings, plugins, build options).
- **`_posts/`**: Blog posts (Markdown). Naming convention: `YYYY-MM-DD-title.md`.
- **`_pages/`**: Static pages (Markdown).
- **`_drafts/`**: Unpublished posts.
- **`_layouts/`**: HTML templates (`default.html`, `post.html`, `page.html`, etc.).
- **`_includes/`**: Reusable partials (header, footer, navigation).
- **`assets/`**: CSS, JS, images, and fonts.
- **`_plugins/`**: Custom Ruby plugins (`cloudinary.rb`, `compression.rb`, etc.).
- **`_docs/`**: Project documentation (Jekyll commands, Cloudinary setup, etc.).

## Development Conventions

- **Content:**
  - Pages should generally be placed in `_pages/` and included via `_config.yml` settings.
  - Posts go in `_posts/` with standard Jekyll front matter.
- **Styling:**
  - SCSS files are located in `_sass/` or `assets/css/` (check `assets/main.scss`).
  - Use Bootstrap classes where applicable.
- **Assets:**
  - Images are often managed via Cloudinary (check `_plugins/cloudinary.rb` and
    `_docs/CLOUDINARY_*.md`).
  - Local assets should be optimized.
- **Performance:**
  - The project uses `smart_asset_cache.yml` and extensive compression settings in `_config.yml`.
  - Ensure any new assets or plugins do not negatively impact site speed (Lighthouse/PageSpeed).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leonism) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
