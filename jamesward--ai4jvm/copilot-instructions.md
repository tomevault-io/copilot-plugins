## ai4jvm

> Single-page website (`index.html`) — HTML + inline CSS, no build step.

# AI4JVM

Single-page website (`index.html`) — HTML + inline CSS, no build step.

## Spec

`SPEC.md` is the source of truth for all site content and structure. When updating the site:

1. Update `SPEC.md` first with the new content, links, and descriptions
2. Then update `index.html` to match the spec

## Style Rules

- Keep descriptions concise (2-3 sentences) and factual
- Cards use badge classes: `badge-framework`, `badge-inference`, `badge-assistant`, `badge-resource`
- Each card has a title, description, and links (Docs, GitHub, Website, etc.)
- If a project is abandoned but still useful, note it in the description (e.g. "⚠️ No longer actively maintained"). Remove abandoned projects that are no longer useful.

## Fetching

- When needed use a browser tool to fetch web pages
- **Never infer or guess page contents from URLs.** Always fetch the actual page content (via web search, browser tool, or other means) before writing titles, descriptions, or summaries. If a direct fetch fails, use web search to find the content.

---
> Source: [jamesward/ai4jvm](https://github.com/jamesward/ai4jvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
