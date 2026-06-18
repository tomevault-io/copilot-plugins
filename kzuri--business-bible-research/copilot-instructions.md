## business-bible-research

> Create comprehensive company, competitor, product, or market "business bible" reports from public research. Use when a user asks to research a business deeply; understand a company, competitor, SaaS, fintech, marketplace, or product; build a sales/BD/CEO-ready dossier; capture website screenshots and source URLs; analyze features, business model, customers, use cases, reviews, pain points, GTM, and founder strategy.


# Business Bible Research

## Overview

Produce a sourced, screenshot-backed Markdown report that lets a sales, BD, strategy, or founder team understand a target business well enough to sell against it, partner with it, buy from it, or build something similar.

Default output is a single `*_bible.md` plus supporting folders for screenshots, text extracts, and URL inventory.

## Workflow

1. Clarify only if the target is ambiguous. Otherwise infer a short slug from the company/product name and proceed.
2. Create a workspace folder structure:
   - `<slug>_screenshots/`
   - `<slug>_extracts/`
   - `<slug>_sitemap_inventory.csv` when a sitemap exists
   - `<slug>_bible.md`
3. Browse the target website like a real user. Prefer Chrome or the browser requested by the user when available. Navigate the homepage, top nav, footer, pricing, product/features, solutions/use cases, customers/case studies, integrations, security/trust, resources, blog, alternatives/comparison pages, and contact/demo pages.
4. Save full-page screenshots for important pages and write visible text extracts to JSON. Every screenshot reference in the report must include the source URL beside it.
5. Build a URL inventory. Use sitemap/robots where available as a supporting index, but do not substitute sitemap crawling for normal browser navigation when the user asks to inspect the site.
6. Research external sources: LinkedIn/company profile, review sites, Reddit/community discussions, press/funding, customer case studies, competitor pages, and trusted databases. Browse for current facts when facts may have changed.
7. Synthesize into a business bible using the outline in `references/report-outline.md`.
8. Add a confidence/gaps section. Be explicit about what public research cannot answer, such as exact revenue, live pricing quotes, unit economics, private roadmap, internal support data, and regulated payment economics.
9. Verify that all local screenshot links exist and that all screenshot references include source URLs.

## Research Standards

- Separate confirmed facts from inference.
- Prefer primary sources for company/product claims.
- Treat company-owned alternative/comparison pages as biased positioning, not neutral truth.
- Do not over-weight Reddit unless there are detailed first-hand accounts. Summarize weak social signal as weak.
- Capture pain points from third-party reviews and community sources, not only the target company's own marketing.
- Include absolute or workspace-relative local links to screenshots and artifacts.
- Keep a page atlas with page title, H1, word count, URL, and screenshot.

## Browser Capture Pattern

For each important page, capture:

- `slug`
- requested URL
- final URL
- page title
- H1/H2/H3 headings
- visible body text
- internal links
- full-page screenshot path

If using Chrome through the Chrome skill, use a single tab where practical, write extracts incrementally, and work in batches to avoid tool timeouts. If a guessed URL is blank or stale, find the live link from the current website and record the stale URL as excluded.

## Report Requirements

The final report must include:

- Executive summary.
- Company snapshot.
- Website positioning.
- Product/module architecture.
- Use-case map.
- Business model and pricing.
- Target customer and buyer personas.
- Customer evidence and case studies.
- Competitive positioning.
- Review/community pain points.
- Sales and BD playbook.
- CEO/founder blueprint for building a similar product.
- Implementation/evaluation checklist.
- Questions to ask sales/customers.
- Page atlas with URLs and screenshots.
- Source appendix.
- Gaps and confidence level.

Read `references/report-outline.md` for the full section outline and `references/research-checklist.md` for collection checklists.

## Helpful Script

Use `scripts/create_business_bible_scaffold.py` to create the standard folders and a starter Markdown file:

```bash
python3 /path/to/business-bible-research/scripts/create_business_bible_scaffold.py "Company Name" --out .
```

The script is intentionally small. Patch or extend it in-task if the user needs a custom artifact layout.

---
> Source: [kzuri/business-bible-research](https://github.com/kzuri/business-bible-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
