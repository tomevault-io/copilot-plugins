## hugo-blog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal blog built with [Hugo](https://gohugo.io/) using the [Stack theme](https://github.com/CaiJimmy/hugo-theme-stack). Content is authored in Emacs Org-mode and exported to Markdown via ox-hugo.

**Working Directory**: `/Users/fengh/Documents/RDS/BLOG/hugo` is a symlink to `hugo.stack/`

---

## Development Commands

```bash
# Start development server (with drafts)
hugo server -D

# Build for production
hugo --minify

# Output directory: public/
```

---

## Content Workflow (Org-mode → Hugo)

Blog posts are authored in `content-org/` as Org-mode files and exported to `content/post/` via ox-hugo.

### Org-mode Post Structure

```org
#+hugo_base_dir: ../
#+hugo_section: /post/
#+options: author:nil
#+options: ^:nil
#+options: tex:t
#+startup: inlineimages

* ✔ DONE Post Title                                          :tag1:tag2:@category:
CLOSED: [2026-03-21 Sat 19:34]
:PROPERTIES:
:EXPORT_FILE_NAME: 20260321-post-slug
:END:

Content here...
```

### Key Properties

- `EXPORT_FILE_NAME`: Output filename (without extension)
- Tags after title: `:tag1:tag2:` - tags starting with `@` become categories
- `CLOSED`: Date for post metadata

### Export Process

Posts are exported from `content-org/all-posts-*.org` files. Each Org file can contain multiple posts (as headings). The ox-hugo package in Emacs handles the export.

---

## Configuration Structure

Uses modular configuration in `config/_default/`:

| File | Purpose |
|------|---------|
| `hugo.toml` | Base Hugo config, menu, pagination |
| `params.toml` | Theme parameters (sidebar, widgets, comments) |
| `languages.toml` | Language settings (zh-cn primary) |
| `markup.toml` | Markdown rendering, syntax highlighting |
| `social.toml` | Social media links |

**Alternative**: Single-file config exists as `hugo.toml.stack` (backup)

---

## Theme Customization

The Stack theme is overridden in `layouts/`:

```
layouts/
├── _partials/           # Custom partials
│   ├── article/         # Article-related components
│   ├── article-list/    # List rendering
│   ├── sidebar/         # Sidebar customization
│   └── widget/          # Widget overrides
├── archives.html        # Custom archive page (shows categories + timeline)
└── partials/            # Additional partials
```

### Custom Archive Page

`layouts/archives.html` overrides the theme default:
- Shows category tiles at top (with images from `data/categories.yaml`)
- Displays timeline grouped by year below

### Category Images

Category cover images are configured in `data/categories.yaml`:

```yaml
category_name:
  image: "https://example.com/image.jpg"
```

---

## Content Structure

```
content/
├── _index.md            # Homepage
├── _index.zh-cn.md      # Homepage (Chinese)
├── about/
│   └── _index.md        # About page
├── archive/
│   └── _index.md        # Archive page
├── categories/          # Category taxonomy
├── tags/                # Tag taxonomy
└── post/                # Blog posts (exported from org files)
```

---

## Theme Features

- **Three-column layout**: Sidebar (left) + Content (center) + Widgets (right)
- **Widgets**: Search, archives, categories, tag-cloud, TOC (configurable in `params.toml`)
- **Dark mode**: Auto-switching based on system preference
- **Syntax highlighting**: Uses `catppuccin-macchiato` style (configurable)
- **Built-in search**: Client-side search via Fuse.js

---

## Static Assets

Place images and static files in:
- `static/img/` - Images (referenced as `/img/filename.png`)
- `static/favicon.ico` - Site favicon

---

## Taxonomies

- **Tags**: `tags` (e.g., `:tech:hugo:`)
- **Categories**: Prefixed with `@` in org files (e.g., `:@教育:`)
- **Main section**: `post`

---

## Build Output

- Production build outputs to `public/`
- Deploy target is typically the root directory of a web server
- No build scripts needed - Hugo handles everything

---

## Migration Notes

Previously used PaperMod theme. Backup configs exist as:
- `hugo.toml.papermod` - Old single-file config
- `layouts.jane-backup/` - Old layouts (if present)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddsteed)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ddsteed)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
