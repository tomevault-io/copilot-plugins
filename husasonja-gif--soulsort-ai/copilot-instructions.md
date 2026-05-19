## blog-publish-workflow

> Blog publish workflow for SEO, links, and deployment


# Blog Publish Workflow ("Publish this")

When the user says "Publish this" after providing blog text, do all steps below.

## 1) SEO Elements
- Add `<title>` and `<meta name="description">`.
- Add Open Graph tags (`og:title`, `og:description`, `og:type`, `og:url`, `og:image`).
- Add Article schema using JSON-LD (`@type: "Article"`).

## 2) Content Structure
- Use semantic headings (`<h1>`, `<h2>`, `<h3>`).
- Split long blocks into short paragraphs.
- Convert list-like sections into bullet lists where it improves readability.

## 3) Linking Rules
- Add 2 internal links to SoulSort pages (use the bold text anchors the user indicates).
- Add 4 external source links cited in the article.
- All links must open in a new tab (`target="_blank"`). Include `rel="noopener noreferrer"` for safety.

## 4) Image Optimization
- Suggest one abstract hero image in brand colors.
- Provide a concrete image filename and descriptive alt text.
- If the user uploads an image, compress/optimize it before publish.

## 5) CTA
- End with a clear call-to-action linking to the product page.

## 6) Review Checklist
- Confirm all SEO elements are present.
- Verify mobile responsiveness.
- Check page speed basics.

## 7) Deploy Workflow
- Publish to `/blog/ai-dating-apps-women-queers-product`.
- Submit URL to Google Search Console.

## Clarification Guardrails
- If internal SoulSort link targets are not explicitly provided, ask which two internal URLs to use before publishing.
- If four external sources are not provided or inferable, ask for them before final publish.

---
> Source: [husasonja-gif/soulsort-ai](https://github.com/husasonja-gif/soulsort-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
