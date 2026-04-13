## economist

> These rules apply to **all content and page updates** in this repo. Cursor must follow them in order.

# Cursor Rules – Sarah Zou Consulting

These rules apply to **all content and page updates** in this repo. Cursor must follow them in order.

---

## 0. Project Assumptions

- Site: `https://sarahzou.com`
- Content type: consulting, pricing/monetization, SaaS economics
- Author: `Dr. Sarah Zou`
- Main content folders: `/newsletter/`, `/wiki/` (or equivalent content folders)
- If folder changes, update the base path in **Section 2**.

---

## 1. UI / Theming Rules (Global)

Use these for **pages, sections, and components**.

### 1.1 Colors

- **Primary (CTA) color:** `#ff5722`
- **Primary hover:** `#e44e1f`
- **Text / headings (primary):** `#1f2933`  
  - All main body text and headings use this color.
- **Muted text:** `#3b4652`  
  - Use ONLY for low-priority UI text (timestamps, captions, helper labels).  
  - Never use muted text for main paragraphs or long-form content.
- **Background:** `#ffffff`
- **Surface / cards:** `#f6f7f9`
- **Border:** `#e2e6ea`
- **Optional accent (links, very sparing):** `#2563eb`
- **Orange is ONLY for CTAs, highlights, or key metrics.** Do not use orange as a full background or as body text color.

### 1.2 Typography

Use a consistent typography system across the entire site (marketing + wiki).

**Fonts**

- **Body font:** `Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`
- **Heading font:** `DM Sans, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`

**Base text**

- **Body size (desktop):** 17–18px
- **Body size (mobile):** ≥16px
- **Line-height (body):** 1.6–1.75
- All long-form content (wiki, articles, posts) must use primary text color and these base sizes. Do not shrink body text below 16px.

**Heading scale (guideline)**

You may adjust slightly for breakpoints, but keep the hierarchy:

- **H1:** 32–36px, `font-weight: 700`
- **H2:** 24–28px, `font-weight: 600`
- **H3:** 20–22px, `font-weight: 600`
- **H4:** 18–20px, `font-weight: 500`

Headings:

- Always use the heading font (`DM Sans`).  
- Always use the primary text color (`#1f2933`).  
- Never use orange for headings.

### 1.3 Layout for Content Pages

- **Max width (articles/wiki):** ~`max-w-3xl` (approx 680–720px)
- **Horizontal padding:** `1–1.5rem` (e.g. `px-4`)
- **Vertical spacing:** sections 4–5rem top/bottom; cards 1.5rem padding.
- Long-form pages (e.g. wiki) should be wrapped in a consistent container with these properties and should not define ad-hoc font sizes/colors inside the content body unless absolutely necessary.

### 1.4 CSS Tokens (reusable)

```css
:root {
  --color-primary: #ff5722;
  --color-primary-hover: #e44e1f;
  --color-text: #1f2933;
  --color-text-muted: #3b4652;
  --color-bg: #ffffff;
  --color-surface: #f6f7f9;
  --color-border: #e2e6ea;
  --color-accent: #2563eb;

  --font-body: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --font-heading: "DM Sans", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;

  --radius-lg: 1rem;
  --radius-md: 0.75rem;
  --shadow-card: 0 12px 40px rgba(15, 23, 42, 0.04);
}

/* Global typography */
body {
  font-family: var(--font-body);
  color: var(--color-text);
  font-size: 17px;
  line-height: 1.65;
}

h1, h2, h3, h4 {
  font-family: var(--font-heading);
  color: var(--color-text);
  letter-spacing: -0.01em;
}

/* Common components */
.btn-primary {
  background: var(--color-primary);
  color: #fff;
  padding: 0.7rem 1.4rem;
  border-radius: 9999px;
  border: none;
  font-weight: 600;
}
.btn-primary:hover {
  background: var(--color-primary-hover);
}

.card {
  background: var(--color-surface);
  border: 1px solid rgba(226, 230, 234, 0.6);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-card);
  padding: 1.5rem;
}

/* Muted text is for captions/labels only */
.text-muted {
  color: var(--color-text-muted);
  font-size: 0.875rem;
}


2. Content / SEO Rules
Use these for blog/newsletter/wiki/insight-style markdown.

2.1 Frontmatter (canonical)
---
title: ""
date: YYYY-MM-DD
description: ""
image: "/assets/images/newsletter/{{slug}}.webp"
canonical: "https://sarahzou.com/newsletter/{{slug}}"
tags: ["pricing", "saas", "monetization"]
---
date = today, unless intentionally backdating.

{{slug}} = kebab-case from title.

If content is not a newsletter, adjust path to /assets/images/posts/{{slug}}.webp and /posts/{{slug}}.

2.2 Filenames
Markdown files: YYYY-MM-DD-slug.md

Image files: /public/assets/images/newsletter/{{slug}}.webp

Keep paths consistent so canonical URLs don’t break.

2.3 Images
Format: WebP

Size target: 1200×630px

Filenames: descriptive, lowercase, hyphenated

If no image is available, insert a placeholder path using the standard structure.

2.4 Internal links
Each new piece of content must link to:

One pillar page (e.g. /consulting or your main service page)

One related concept (e.g. /pricing, /monetization-narrative, /saas-benchmarks, /wiki/...)

This is to keep the site interlinked and founder-friendly.

3. Tone & Content Pattern
Tone: Professional, minimalist, energetic.

Short paragraphs (2–4 lines).

For service/content pages, prefer this order: Who it’s for; What problem it solves; What’s delivered; Why it’s founder-/investor-friendly.

Avoid jargon piles; keep examples grounded in early-stage tech/SaaS.


4. Generation Protocol
When generating or editing content/code:

4.1 Read this file.

4.2 Decide whether the user is editing a page/layout (use Section 1) or a post/wiki article (use Sections 1 and 2).

4.3 If it’s a post/wiki:

- Create/validate frontmatter using the template.

- Ensure long-form content uses the global typography rules: Primary text color (#1f2933); Base font sizes (≥16px mobile, 17–18px desktop); Comfortable line-height

- Do not use muted grey for main paragraphs.

4.4 Apply global UI/theming if any JSX/HTML is generated:

- Use the shared CSS tokens and .btn-primary, .card, etc.

- Do not introduce new colors or fonts.

4.5 Add at least 2 internal links (pillar + related concept).

4.6 Output final code/markdown only.


5. Change-Safe Parts
To avoid future breakage:

Do not hard-code years (e.g. 2024). Use the current date when generating.

Do not hard-code /newsletter/ in JSON-LD; use the current content path.

If the folder changes, update this section once and reuse.

6. Example JSON-LD (with placeholder)
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "{{title}}",
  "datePublished": "{{date}}",
  "author": {
    "@type": "Person",
    "name": "Sarah Zou"
  },
  "mainEntityOfPage": "https://sarahzou.com/{{content-path}}/{{slug}}"
}
Use this only if the existing file already has JSON-LD or the project explicitly requires it. Do not add JSON-LD to files that previously had none without an explicit instruction.

7. Pre-publish Checklist
 Frontmatter present and valid

 Body text uses primary color (#1f2933) and base font size (≥16px)

 No main paragraphs use muted grey text

 CTA uses #ff5722 or .btn-primary

 Headings use primary text color and heading font

 At least 2 internal links

 Image path in standard folder

 No new colors or fonts introduced

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sarah-Zou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
