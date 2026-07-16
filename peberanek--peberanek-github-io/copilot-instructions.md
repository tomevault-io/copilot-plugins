## peberanek-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog and portfolio website built with MkDocs Material. The site is authored in Czech and English and covers topics related to AI, generative AI, programming, and personal projects. The site is deployed to GitHub Pages via GitHub Actions.

## Technology Stack

- **Static Site Generator**: MkDocs with Material theme
- **Package Manager**: uv (modern Python package manager)
- **Python Version**: 3.12+
- **Key Plugin**: mkdocs-material with blog plugin
- **Deployment**: GitHub Actions → GitHub Pages (branch: main)

## Architecture

### Directory Structure
- `docs/` - All content source files
  - `docs/posts/` - Blog posts (Markdown files)
  - `docs/index.md` - Blog homepage
  - `docs/cv.md` - CV and contact page
  - `docs/projects.md` - Projects showcase
  - `docs/tags.md` - Tag index page
  - `docs/assets/` - Images and other assets
- `mkdocs.yml` - Main configuration file
- `.github/workflows/building.yml` - CI/CD pipeline

### Blog Post Format
All blog posts must include YAML frontmatter with:
- `date:` (YYYY-MM-DD format)
- `tags:` (list of tags for categorization)
- Optional: `draft: true` (for unpublished posts)

Posts use `# Title` as the main heading and `<!-- more -->` to mark excerpt cutoff.

## Content Guidelines

### Language
- Blog posts are written mostly in Czech for the ease of writing, with some content in English
- The rest of the website is written in English

### Links
- Internal links use relative paths: `../posts/other-post.md`
- Always verify link targets exist before creating references

## Working with Blog Posts

When creating or modifying blog posts:
1. Use the `./create-post` script for new posts (it sets up frontmatter correctly)
2. Ensure date format is YYYY-MM-DD
3. Add relevant tags from existing tag set where possible
4. Place `<!-- more -->` after the first paragraph(s) to define excerpt

## Branches

- `main` - Production branch (triggers deployment)
- Simple commits (like new blog posts) go to `main`, but if there are lots of changes, a feature branch may be needed

## Python development
- When developing Python code, use:
  - `uv run ruff check` for linting
  - `uv run ruff format` for formatting
  - `uv run ty check` type checking

---
> Source: [peberanek/peberanek.github.io](https://github.com/peberanek/peberanek.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
