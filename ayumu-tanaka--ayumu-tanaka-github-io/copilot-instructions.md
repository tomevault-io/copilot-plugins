## ayumu-tanaka-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based academic website for an Associate Professor at Aoyama Gakuin University, hosted on GitHub Pages. The site contains:
- Personal CV and research publications
- Teaching materials and course information
- Research notes and tutorials (R, Stata, LaTeX)
- Curated datasets for student research

## Build and Deployment

- **Build system:** Jekyll with `jekyll-theme-minimal` theme
- **Hosting:** GitHub Pages (automatic build on push to main)
- **No local build required:** Changes pushed to `main` branch are automatically deployed
- **Configuration:** `_config.yml`

To test locally (optional):
```bash
bundle exec jekyll serve
```

## Repository Structure

```
/                     # Root: homepage, CV, research pages
├── teaching/         # Course materials, seminar info, datasets (CSV, DTA, XLSX)
├── Notes/            # Research tutorials organized by topic
│   ├── Rnotes/       # R programming notes
│   ├── StataNotes/   # Stata notes
│   └── OtherNotes/   # VS Code, Overleaf notes
├── research/         # Research paper PDFs
└── photos/           # Personal photos
```

## Content Format

All pages use Jekyll Markdown with YAML front matter:
```yaml
---
layout: default
title: Page Title
---
```

## Language

Content is bilingual (Japanese and English). The main audience is Japanese students, so Japanese content predominates in teaching materials.

## Key Files

- `teaching/readme.md` - Main teaching page with current courses
- `teaching/seminar.md` - Detailed seminar description
- `teaching/Data.md` - Curated list of public datasets
- `Notes/notes.md` - Index of all research tutorials
- `research.md` - Publications and working papers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ayumu-tanaka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
