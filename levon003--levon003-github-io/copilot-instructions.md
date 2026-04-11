## levon003-github-io

> This file provides guidance to Claude Code and other coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to Claude Code and other coding agents when working with code in this repository.

## Repository Overview

This is Zachary Levonian's personal website: a Jekyll-based static site hosted on GitHub Pages. The site has two main sections:
- `/` (index page): Includes bio and lists peer-reviewed papers
- `/blog`: Lists blog posts

The repository also contains multiple Python projects in the `src/` directory used for data analysis, research, and content generation for blog posts.

## Jekyll Site (Main Site)

### Building and Running

Install dependencies (requires Ruby 3.1.7 via RVM):
```bash
bundle install
bundle binstubs --all
```

Run the development server:
```bash
bin/jekyll serve
```

Build the site:
```bash
bin/jekyll build
```

Debug build issues:
```bash
bin/jekyll build --verbose
```

### Important Configuration

- Jekyll config: `_config.yml`
- Theme: Minima 2.5
- Posts directory: `_posts/`
- Layouts: `_layouts/`
- Includes: `_includes/`
- Tags directory: `tags/` (auto-generated)

### Content Structure

Blog posts are in `_posts/` and follow Jekyll naming conventions (YYYY-MM-DD-title.md). Each post has YAML front matter with fields like `layout`, `title`, `date`, `tags`, and `excerpt`.

Tags are extracted from posts and auto-generated into stub pages via a pre-commit hook (see below).

### Pre-commit Hook

A pre-commit hook automatically generates Jekyll tag stub pages:
```bash
pre-commit install
pre-commit run --all-files
```

The hook runs `scripts/create_jekyll_tag_stubs.py` which:
- Extracts tags from all posts in `_posts/`
- Creates tag pages in `tags/` for any new tags
- Slugifies tag names for URLs

## Python Projects (src/)

The `src/` directory contains multiple independent Python projects for research, data analysis, and exploration. Most projects use **uv** for dependency management, though some older projects use **poetry**.

### Project Organization Patterns

1. **Packaged Projects**: Most directories have `pyproject.toml` + README.md, often with `src/` subdirectories and tests
2. **Notebook-First Projects**: Minimal Python code, focused on Jupyter notebooks

### Key Projects

- **misc/**: General-purpose utilities, LeetCode solutions, notebooks for various analyses. You can create new project folders, but if you just want a place to put a small Python module or script you can put it in ./src/misc/src/levon_misc.
- **template/**: Boilerplate uv template for new projects

### Working with Python Projects

Navigate to a specific project directory and use the appropriate package manager:

Using uv (most projects):
```bash
cd src/misc
uv sync
uv run pytest
```

## Architecture Notes

### Jekyll Customizations

The site uses custom layouts and includes that extend the Minima theme:
- Custom post layout in `_layouts/post.html` for enhanced post display
- Custom tag layout in `_layouts/tag.html` for tag pages
- Social media icons customization in `_includes/social.html`
- Quiz include in `_includes/quiz.html` for interactive content

### Tag System

The tag system is semi-automated:
1. Tags are added to post front matter (can be YAML list or space-separated string)
2. Pre-commit hook extracts tags and creates stub pages in `tags/`
3. Each tag page uses the `tag` layout to list all posts with that tag
4. Tags are slugified for URLs (lowercase, hyphens, no special chars)

### Content Workflow

Blog posts may have accompanying code and data in `src/`, though each project is independent.

The `scripts/` directory contains utilities for content management, often one-off historical workflows.

## Development Environment

The repository uses:
- Ruby 3.1.7 (specified in `.ruby-version`)
- Python 3.13 for most projects
- Pre-commit hooks for automated tag generation
- Both uv and poetry for Python package management

Files and directories to ignore (per `.gitignore`):
- `.claude/`, `.venv/`, `__pycache__/`, `.ipynb_checkpoints`
- `bin/`, `gembin/` (generated Jekyll/Ruby executables)
- `_site/`, `.sass-cache/`, `.jekyll-metadata` (Jekyll build artifacts)
- `.env` files (may contain API keys)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/levon003)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/levon003)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
