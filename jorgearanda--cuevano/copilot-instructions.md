## cuevano

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is Jorge Aranda's personal website and blog, built with Jekyll 4.4 using the Lanyon theme. The site is hosted on AWS S3 with CloudFront distribution at cuevano.ca.

## Essential Commands

### Setup
```bash
bundle install
```

### Local Development
```bash
jekyll serve
```
The site will be available at http://localhost:4000

### Deployment
```bash
./deploy.sh
```
This syncs the built site to S3 and invalidates the CloudFront cache. Requires AWS CLI to be installed and configured with credentials (see S3_SETUP.md for initial setup).

## Architecture

### Site Structure

- **_posts/**: Blog posts in Markdown format, named `YYYY-MM-DD-title.md`
- **_layouts/**: HTML templates
  - `default.html`: Base layout with sidebar and masthead
  - `post.html`: Individual blog post layout with related posts
  - `page.html`: Static page layout
- **_includes/**: Reusable HTML fragments (head.html, sidebar.html)
- **public/css/**: Stylesheets (lanyon.css, poole.css, syntax.css)
- **_site/**: Generated static site (gitignored, not committed)

### Configuration

Jekyll configuration in `_config.yml`:
- Uses `kramdown` with GitHub Flavored Markdown (GFM) for Markdown processing
- Uses `rouge` for syntax highlighting
- Permalink style: `pretty` (no .html extensions)
- Pagination: 5 posts per page via `jekyll-paginate`

Deployment configuration:
- S3 bucket: cuevano.ca (ca-central-1)
- CloudFront distribution ID: E2RXDD0YF5EZHS
- Cache-Control: public, no-transform, max-age=1200, s-maxage=1200
- S3 bucket configured with redirects for legacy URLs (feed.xml, index.php)
- See deploy.sh for deployment script and S3_SETUP.md for bucket configuration

### Content Workflow

Blog posts follow this pattern:
1. Create new file in `_posts/` with format `YYYY-MM-DD-title.md`
2. Add YAML front matter (title, date, author, layout)
3. Write content in Markdown
4. Test locally with `jekyll serve`
5. Deploy with `./deploy.sh`

Static pages (about.md, archive.md, publications.md) live in the root directory and use the `page` layout.

### Theme Customization

The site uses a custom theme variant (`theme-custom`) defined in the body class of `default.html`. The Lanyon theme provides a collapsible sidebar navigation triggered by a checkbox input.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jorgearanda) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
