## divorce-coach

> - **E-E-A-T:** Write with Google's E-E-A-T framework in mind (Experience, Expertise, Authoritativeness, Trustworthiness). Demonstrate first-hand experience where possible, cite credible sources, include author credentials, use accurate and up-to-date information, and avoid vague or unsubstantiated claims

# Claude Code Instructions

## Blog Posts

### Blog quality guidelines
- **E-E-A-T:** Write with Google's E-E-A-T framework in mind (Experience, Expertise, Authoritativeness, Trustworthiness). Demonstrate first-hand experience where possible, cite credible sources, include author credentials, use accurate and up-to-date information, and avoid vague or unsubstantiated claims
- **Author:** Always include author where the format supports it
- **Schema:** Ensure the article has all correct structured data/schema markup
- **SEO/GEO/AEO:** Write with search, generative, and AI engine optimisation in mind
- **FAQs:** Include FAQ sections where possible (good for featured snippets and AEO)
- **Tables:** Use nicely styled tables for tabular data
- **SVG charts:** Create inline SVG charts/diagrams where applicable to visualise data
- **Internal linking:** Link to other pages/posts on the same site
- **External linking:** Link to authoritative external sources where relevant. External links must open in a new tab (`target="_blank" rel="noopener noreferrer"`) and include a small external link icon (e.g. `↗` or an SVG) so users know they are leaving the site. **Every external URL should be verified with a curl/fetch check (expecting a 200 status) before being added to an article.** If a URL returns a non-200 status, find a working alternative. If you cannot verify URLs (e.g. no network access), still include them but flag which ones were not verified so the user can check them.
- **UK English:** Use UK spelling throughout (colour, organised, centralised, etc.)
- **Heading hierarchy:** Proper H2 -> H3 nesting, never skip levels
- **Meta description:** Under 160 characters, include the primary keyword
- **Short paragraphs:** Max 3-4 sentences, scannable with subheadings and bullet points
- **Primary keyword focus:** Each post should target a specific keyword/phrase
- **Strong opening:** Hook the reader and summarise the value in the first 2-3 sentences (helps with AI answer extraction for AEO)
- **Image dimensions:** No images required for blog posts on this site
- **Alt text:** Descriptive alt text on any SVGs/images for accessibility and SEO
- **Call to action:** Tie back to the coaching services where natural. CTA page: `/book-a-free-call/` (Kabooly form)
- **Reading time:** Calculate based on ~230 words per minute
- **No en/em dashes:** Never use en dashes (-) or em dashes (--) in blog content. Use commas, colons, semicolons, or rewrite the sentence instead
- **Repo structure:** Always check an existing post in this repo before writing a new one
- **Topical content:** Where possible, make blog content topical. If there is a big event or something notable in the calendar in the forthcoming days or weeks, reference it in the blog. This will not always be possible, so only do this when it makes sense. Always confirm the current date before referencing upcoming events — do not assume or guess the date.

## CSS
- **Mobile first:** All CSS must be mobile first. Never use `max-width` media queries. Use `min-width` only.

## What You Can Do
- Read and edit code
- Run development servers and tests
- Search and explore the codebase
- Provide guidance and suggestions

---

# Divorce Coach

Divorce coaching certification course and information site.

## Stack
- Eleventy (static site generator)
- Nunjucks templating
- Tailwind CSS
- Decap CMS at `/admin/`

## Blog System

- **Location:** `src/posts/`
- **Format:** Markdown with YAML frontmatter
- **Layout:** `posts.html` template (via `posts.json` directory data)
- **Frontmatter:** `title`, `seoTitle`, `description`, `date` (ISO 8601), `type` (`article`), `tags` (array)
- **Tags:** Two audience categories:
  - `divorce` = content for people going through a divorce (potential coaching clients)
  - `coaching` = content for people wanting to become a coach (potential training customers)
  - Most posts should have one or the other, rarely both
- **Permalink:** `/{{ title | slugify }}/` (set in `posts.json`)
  - **URLs are derived from the `title` field, NOT the filename.** Filenames often don't match titles. When creating internal links, always read the target post's `title` and slugify it to get the correct URL. Never guess from the filename.
- **After creating:** No index update needed (Eleventy collections are automatic)
- **Schema:** Article JSON-LD generated in `posts.html` layout via `partials/json-ld.html`
- **Reference:** Always check an existing post first to match the structure

## Content Counts

| Type | Count | Location |
|------|-------|----------|
| Blog posts | 43 | `src/posts/` |

## Status: Live
Ongoing content updates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/webdigga)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/webdigga)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
