## jtorreggiani-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal website for Joe Torreggiani, served at **joetorreggiani.com** via GitHub Pages. Based on [Chuck Groom's one-page bio template](https://github.com/chuckgroom/onepage-bio). Single-page portfolio/resume site. Blog infrastructure exists (`_posts/`, `_includes/blog_grid.html`, `_layouts/post.html`) but is currently **commented out** in `_layouts/default.html` and `_includes/nav.html` while the first post is being refined.

## Build & Development

```bash
# Install dependencies
bundle install

# Serve locally with live reload (http://localhost:4000)
bundle exec jekyll serve

# Build static site to _site/
bundle exec jekyll build
```

No CI/CD configuration — GitHub Pages builds automatically on push to `main`.

## Architecture

**Jekyll 4.3.3** static site with a single-page layout:

- `_config.yml` — Site metadata, color scheme, social links, and personal info. Content changes (name, tagline, skills, colors) are configured here, not in templates.
- `_layouts/default.html` — The only HTML layout. Full page structure: navbar, header (profile image + name + tagline), three content sections, footer.
- `_layouts/style.css` — CSS "layout" that concatenates Bootstrap CSS, custom CSS, and any page CSS content via Liquid templating.
- `style.css` — Root CSS file that uses the style layout (entry point for the CSS pipeline).

**Content sections** (in `_includes/`):
- `about_me.html` — Bio narrative (#about)
- `interests.html` — Skills, projects, research areas, personal interests grid (#interests)
- `contact_form.html` — Contact form using Formspree at `//formspree.io/{{ site.email }}` (#contact)

**CSS/JS stack:**
- Bootstrap 3.2.0 (Bootswatch Flatly theme) — in `_includes/css/bootstrap.min.css`
- Custom styles — in `_includes/css/main.css` (fonts: Montserrat headings, Lato body)
- Font Awesome 4.x — in `css/font-awesome/`
- jQuery 1.11.0, Bootstrap JS, jQuery Easing — in `js/`
- `js/freelancer.js` — Smooth scrolling, floating labels, navbar scroll spy, mobile menu toggle
- `js/cbpAnimatedHeader.js` — Navbar shrink-on-scroll effect
- `js/contact_me_static.js` — Client-side form validation (the active contact handler)

## Key Details

- Custom domain configured in `CNAME` file (joetorreggiani.com)
- Site colors are defined in `_config.yml` under `color` and used via `{{ site.color.primary }}` etc. in templates
- Social network links are in `_config.yml` under `networks` and rendered in the footer
- No Sass preprocessing is used despite jekyll-sass-converter being in the dependency tree — CSS is plain and concatenated via Liquid
- The `_includes/css/` directory is an unconventional location for stylesheets (inherited from the template)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jtorreggiani) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
