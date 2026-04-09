## wardwestminster

> Generates 480w, 800w, 1200w WebP variants with JPG fallback.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ward&Westminster is a Hugo theme for long-form editorial content. Mobile-first, dark mode support, optimized for reading. The site is deployed to GitHub Pages at wardwestminster.com.

**Hugo Version:** 0.153.2 (extended)

## Development Commands

```bash
# Run dev server with example site
cd exampleSite && hugo server --themesDir ../.. --theme wardwestminster

# Run with drafts visible
cd exampleSite && hugo server --themesDir ../.. --theme wardwestminster -D

# Build production site (from root - layouts are in root now)
hugo --gc --minify

# Build example site
cd exampleSite && hugo --themesDir ../.. --theme wardwestminster

# Create new content
hugo new posts/article-title.md
```

## Architecture

### Directory Structure
```
wardwestminster/
├── .github/workflows/hugo.yml  # GitHub Pages deployment
├── archetypes/default.md       # New content template
├── assets/
│   ├── css/main.css           # All styles, variables, dark mode
│   ├── js/main.js             # Nav, progress bar, animations
│   └── images/                # Theme images (processed by Hugo)
├── content/
│   ├── posts/                 # Blog articles
│   ├── authors/               # Author archive pages
│   ├── about.md              # About page
│   └── search.md             # Search page
├── data/authors.yaml          # Author metadata
├── layouts/                   # Hugo templates
├── static/
│   ├── CNAME                  # Custom domain
│   └── pdf/                   # PDF files for embedding
├── hugo.toml                  # Site configuration
└── theme.toml                 # Theme metadata
```

### Layout Hierarchy
```
layouts/_default/baseof.html    # Base template (head, fonts, asset pipeline)
    └── layouts/index.html      # Home page (hero + featured + grid)
    └── layouts/_default/single.html  # Article pages
    └── layouts/_default/list.html    # Archive/category pages
    └── layouts/_default/taxonomy.html # Tag/category listing
    └── layouts/_default/terms.html   # Tag/category index
    └── layouts/_default/search.html  # Client-side search
    └── layouts/authors/              # Author pages
    └── layouts/series/list.html      # Series landing page
    └── layouts/series/terms.html     # Series index page
```

### Key Files

| File | Purpose |
|------|---------|
| `assets/css/main.css` | All styles, CSS variables, dark mode, animations |
| `assets/js/main.js` | Nav, mobile menu, progress bar, scroll animations, back-to-top |
| `layouts/partials/nav.html` | Fixed nav with mobile menu toggle |
| `layouts/partials/head.html` | Meta tags, fonts, CSS pipeline |
| `layouts/partials/footer.html` | Site footer |
| `layouts/partials/picture.html` | Responsive image processing (WebP + srcset) |
| `layouts/partials/reading-time.html` | Medium-style reading time (265 WPM + images + code) |
| `layouts/partials/toc.html` | Auto-generated table of contents |
| `layouts/partials/related.html` | Related articles section |
| `layouts/partials/share.html` | Social share buttons (X, Bluesky, LinkedIn, Facebook, Email, Copy) |
| `layouts/partials/head-custom.html` | User custom styles/meta (extension point) |
| `layouts/partials/scripts-custom.html` | User custom JS/analytics (extension point) |
| `layouts/partials/series-nav.html` | Series "Part X of Y" navigation |
| `layouts/partials/corrections.html` | Corrections and versioning display |
| `data/authors.yaml` | Author data (name, bio, avatar, social) |
| `data/series.yaml` | Series metadata (title, description) |
| `hugo.toml` | Site configuration |

### Shortcodes

```markdown
{{< pullquote cite="Author" >}}Quote text{{< /pullquote >}}
{{< break symbol="◇" >}}
{{< figure-wide src="/image.jpg" alt="..." caption="..." >}}
{{< pdf src="/pdf/document.pdf" width="100%" height="600px" caption="Optional caption" >}}
```

### CSS Variables (in main.css :root)

Light/dark mode colors defined via `--color-*` variables. Override in `layouts/partials/head-custom.html`.

**Colors:**
- `--color-accent`: #8b2635 (burgundy)
- `--color-bg`, `--color-bg-dark`, `--color-bg-nav`
- `--color-text`, `--color-text-secondary`, `--color-text-muted`
- `--color-link`, `--color-link-hover`
- `--color-quote-bg`, `--color-quote-border`
- `--color-border`, `--color-border-strong`

**Typography:**
- `--font-display`: 'Cormorant Garamond' (headings)
- `--font-body`: 'Source Sans 3' (body text)
- `--font-mono`: 'JetBrains Mono' (code)

**Layout:**
- `--max-width-article`: 680px
- `--max-width-wide`: 900px
- `--max-width-site`: 1200px
- `--nav-height`: 60px

**Spacing:** `--spacing-xs` through `--spacing-2xl`

**Dark Mode:** Automatic via `prefers-color-scheme`, or manual with `body.dark-mode` / `body.light-mode` classes.

## Content Front Matter

```yaml
---
title: "Article Title"
date: 2025-12-15
draft: false
description: "SEO description and preview text"
subtitle: "Optional subtitle shown below title"
author: "Dr Matt Kneale"  # Must match key in data/authors.yaml
categories: ["Tech"]       # Primary topic
tags: ["AI", "Healthcare"] # Secondary topics
featured_image: "/images/header.png"
featured_image_alt: "Alt text for accessibility"
featured_image_caption: "Optional image caption"
series: ["Series Name"]    # Optional: links article to a series
series_weight: 1           # Optional: order within series (Part 1, 2, 3...)
lastmod: 2025-12-20        # Optional: last modified date (shows "Updated" in meta)
corrections:               # Optional: list of corrections/updates
  - date: 2025-12-18
    type: correction       # correction, update, or clarification
    description: "Fixed error in statistics paragraph"
  - date: 2025-12-20
    type: update
    description: "Added new data from recent study"
---
```

Use `<!--more-->` to mark the summary break point.

## Series / Multi-Part Articles

The theme supports multi-part article series with "Part X of Y" navigation, series landing pages, and inter-article navigation.

### Creating a Series

1. **Add series front matter** to each article in the series:
```yaml
series: ["NHS Data Scandal"]  # Series name (creates taxonomy)
series_weight: 1               # Part number (1, 2, 3...)
```

2. **Optionally add series metadata** in `data/series.yaml`:
```yaml
"NHS Data Scandal":
  title: "NHS Data Scandal"
  description: "An investigative series examining data governance failures"
  featured_image: "/images/series/nhs-data.jpg"
```

### Series Features

- **Series badge** in article header showing "Series Name · Part X of Y"
- **Series navigation box** above and below article content with:
  - Link to series landing page
  - Previous/Next article links
  - Expandable list of all parts
- **Series landing page** at `/series/series-name/` listing all parts in order
- **Series index page** at `/series/` showing all available series

### Key Files

| File | Purpose |
|------|---------|
| `layouts/partials/series-nav.html` | Series navigation partial |
| `layouts/series/list.html` | Individual series landing page |
| `layouts/series/terms.html` | All series index page |
| `data/series.yaml` | Series metadata (title, description) |

## Corrections & Versioning

The theme supports article corrections, updates, and version tracking with visual indicators for transparency.

### Adding Corrections

Add corrections to article front matter:
```yaml
lastmod: 2025-12-20  # Last modified date
corrections:
  - date: 2025-12-18
    type: correction     # Red badge - factual errors
    description: "Corrected the reported percentage from 45% to 35%"
  - date: 2025-12-20
    type: update         # Burgundy badge - new information
    description: "Added response from NHS spokesperson"
  - date: 2025-12-19
    type: clarification  # Blue badge - clarity improvements
    description: "Clarified timeline of events in paragraph 3"
```

### Correction Types

| Type | Badge Color | Use Case |
|------|-------------|----------|
| `correction` | Red | Factual errors, data mistakes, misquotes |
| `update` | Burgundy | New information, follow-up developments |
| `clarification` | Blue | Wording improvements, added context |

### Features

- **"Updated" indicator** in article meta (next to publish date) when `lastmod` differs from `date`
- **Corrections box** displayed above article content with:
  - Correction type badge
  - Date of correction
  - Description (supports markdown)
- **Schema.org support** with `dateModified` for SEO
- **Dark mode aware** styling

### Key Files

| File | Purpose |
|------|---------|
| `layouts/partials/corrections.html` | Corrections display partial |
| `assets/css/main.css` | Corrections styling (search for "Corrections & Versioning") |

## Author System

Authors are defined in two places:

1. **`data/authors.yaml`** - Metadata (bio, avatar, social links):
```yaml
"Dr Matt Kneale":
  name: "Dr Matt Kneale"
  bio: "Emergency medicine clinical fellow..."
  avatar: "/images/authors/matt-kneale.jpg"
  twitter: "drmattuk"
  bluesky: "drmattuk.bsky.social"
```

2. **`content/authors/{slug}.md`** - Archive pages for each author (minimal front matter, just title)

## Responsive Images

Use the `picture.html` partial for automatic WebP conversion and responsive srcset:
```html
{{ partial "picture.html" (dict "src" .Params.featured_image "alt" "Description" "class" "article-image" "loading" "lazy") }}
```

Generates 480w, 800w, 1200w WebP variants with JPG fallback.

## Configuration

```toml
baseURL = "https://wardwestminster.com/"
languageCode = "en-GB"
title = "Ward&Westminster"

[pagination]
  pagerSize = 10

[taxonomies]
  category = "categories"
  tag = "tags"
  series = "series"

[permalinks]
  posts = "/:year/:month/:slug/"

[params]
  description = "Site description"
  tagline = "Medicine · Tech · Politics"
  author = "Author Name"
  topics = ["Medicine", "Tech", "Politics"]  # Hero section pills

  # Social media handles for share buttons (without @ symbol)
  [params.social]
    twitter = "wardwestminster"
    bluesky = "wardwestminster.bsky.social"

[outputs]
  home = ["HTML", "RSS", "JSON"]  # JSON for search

[markup]
  [markup.goldmark.renderer]
    unsafe = true  # Allow HTML in markdown
  [markup.tableOfContents]
    startLevel = 2
    endLevel = 3

[related]
  includeNewer = true
  threshold = 80
  [[related.indices]]
    name = "categories"
    weight = 100
  [[related.indices]]
    name = "tags"
    weight = 80
```

## Deployment

GitHub Actions workflow (`.github/workflows/hugo.yml`) automatically builds and deploys to GitHub Pages on push to main/master.

## Conventions

- **Mobile-first responsive** - Breakpoints: 640px, 768px, 1024px
- **BEM-like class naming** - `.article`, `.article-header`, `.article-title`
- **Dark mode** - Automatic via `prefers-color-scheme` media query
- **Reduced motion** - Animations respect `prefers-reduced-motion`
- **Vanilla JS only** - No frameworks
- **Image processing** - Place images in `assets/images/` for Hugo processing
- **PDFs** - Place in `static/pdf/` for embedding

## Extension Points

- `layouts/partials/head-custom.html` - Add custom styles/meta
- `layouts/partials/scripts-custom.html` - Add custom JS/analytics
- `data/authors.yaml` - Author bios with avatar support

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anonymort)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anonymort)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
