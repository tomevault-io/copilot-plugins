## fintune-platform

> name: seo_html_defaults

# SEO Cursor Rules – version 1
# These rules are applied by Cursor AI when generating or editing pages intended for SEO performance.
# References: Google SEO starter guide, Core Web Vitals, etc.

<rule>
name: seo_html_defaults
description: Enforce essential on‑page SEO elements on creation/modification of HTML/JSX/TSX/MDX files representing pages or components.
filters:
  - type: file_extension
    pattern: "\\.(html?|jsx?|tsx?|mdx?)$"
  - type: event
    pattern: "file_create|file_modify"
actions:
  - enforce: |
      ## Meta
      - Title tag (≤ 60 chars) includes primary keyword near start.
      - Meta description 140‑160 chars, unique.
      - rel="canonical" if potential duplicates.
      - Open Graph + Twitter Card tags with same title/description.

      ## Headings & Content
      - Exactly one <h1>; hierarchical headings thereafter.
      - First 100 words contain primary keyword naturally.
      - Use semantic HTML5 (<header>, <main>, <article>, <aside>, <footer>).

      ## Media
      - All <img> tags: descriptive alt ≤ 125 chars; lazy-load off‑screen.
      - Serve images as AVIF/WebP when supported, fallback to JPEG/PNG.

      ## Links
      - ≥ 2 relevant internal links, ≥ 1 authoritative external link.
      - Descriptive anchor text (no “click here”).

      ## Structured Data
      - Inject JSON‑LD “WebPage” or “Article” schema with headline, description, datePublished, dateModified, author, image.

      ## Performance
      - Defer non‑essential JS, preload critical resources.
      - Inline critical CSS ≤ 15 kB.
      - Target Core Web Vitals thresholds: LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1.

      ## Words
      - A page must have at least 800 hundread words
</rule>

<rule>
name: accessibility_and_vitals
description: Maintain accessibility and UX that impact SEO rankings.
filters:
  - type: file_extension
    pattern: "\\.(html?|jsx?|tsx?)$"
actions:
  - enforce: |
      - All interactive elements are keyboard‑navigable with ARIA labels when needed.
      - Contrast ratio ≥ 4.5:1 for text.
      - Avoid content shifts (reserve space via width/height attributes).
</rule>

<rule>
name: markdown_blog_post
description: Checklist for SEO‑oriented Markdown/MDX articles.
filters:
  - type: file_extension
    pattern: "\\.(md|mdx)$"
actions:
  - enforce: |
      - YAML/TOML front‑matter fields: title, description, canonicalUrl, date, tags, coverImage.
      - Title ≤ 60 chars; description ≤ 160 chars.
      - Intro paragraph includes target keyword within first 100 words.
      - Automatically generate Table of Contents after the intro.
      - End with call‑to‑action linking to related page/pillar content.
</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maximemarsal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
