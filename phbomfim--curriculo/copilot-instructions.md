## curriculo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
bundle install

# Run local development server (available at localhost:4000)
bundle exec jekyll serve

# Build the site
bundle exec jekyll build
```

> **Note:** Changes to `_config.yml` require restarting the Jekyll server to take effect.

## Architecture

This is a Jekyll-based resume site deployed via GitHub Pages. There are no tests.

### Content is data-driven

All resume content lives in `_data/` as YAML files. The layout reads from these files via Liquid templating:

- `experience.yml` — work history
- `education.yml` — academic background
- `projects.yml` — personal/professional projects
- `associations.yml`, `skills.yml`, `recognitions.yml`, `links.yml`, `interests.yml` — optional sections

### Site configuration controls what sections appear

`_config.yml` is the central control file. It sets personal info, toggles which sections are visible (e.g., `resume_section_skills: false`), and configures social links. The single layout `_layouts/resume.html` uses `{% if site.resume_section_* %}` guards to conditionally render each section.

### Styling

Sass files are in `_sass/`. `css/main.scss` imports them. The layout applies a theme class from `_config.yml`'s `resume_theme` setting to `<body>`. A `no-print` CSS class hides interactive elements when printing; `print-only` shows elements only in print.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phbomfim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/phbomfim)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
