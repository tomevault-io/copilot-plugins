## yiwulane-codex-handoff

> Read all files in `/docs` before writing code.

# AGENTS.md — Project-wide Codex Instructions

Read all files in `/docs` before writing code.

## Product objective

Build a premium, trustworthy B2B marketing website for YIWULANE. It must explain sourcing, quality control, branded packaging, China fulfillment, and hybrid fulfillment to overseas Shopify, TikTok Shop, and Amazon FBA sellers.

The site must convert qualified visitors into a **3-SKU Pilot** application.

## Non-negotiable business rules

- This is not a product catalog or a generic dropshipping marketplace.
- Use one primary CTA consistently: **Start a 3-SKU Pilot**.
- The secondary CTA is **See How It Works** or **Book a Sourcing Call**, depending on context.
- Do not claim YIWULANE is the cheapest, fastest, largest, or best.
- Do not publish unsupported shipping times, order volumes, client counts, success rates, certifications, or geographic coverage.
- Do not fabricate customer logos, reviews, case studies, or operational photos.
- Any non-client example must be labeled **Pilot Project Example** or **Representative Workflow**.
- Use genuine-sounding, concise international English. Avoid hype and awkward literal translation.
- Position YIWULANE as an operational partner, not a software platform.
- State “20 years of international trade experience” accurately and without embellishment.
- Avoid regulated or high-risk product categories in examples.

## Engineering rules

- Use Next.js App Router only.
- Use TypeScript strict mode. Avoid `any`.
- Prefer Server Components. Add Client Components only for interaction.
- Use Tailwind CSS 4 and centralized CSS variables/design tokens.
- Keep content data-driven in `src/content` where practical.
- Use reusable components; do not duplicate entire page sections.
- Use semantic HTML and accessible names.
- Meet WCAG 2.2 AA contrast and keyboard requirements.
- Respect `prefers-reduced-motion`.
- Use `next/image`, `next/font`, route metadata, `sitemap.ts`, and `robots.ts`.
- Do not add a heavy component library unless it clearly reduces complexity.
- Avoid runtime dependencies for trivial utilities.
- Never expose server secrets in `NEXT_PUBLIC_*` variables.
- Validate the lead form on the server and client.
- Add a honeypot and an integration seam for rate limiting.
- Do not store lead PII in repository files or browser storage.
- Make all third-party integrations optional and controlled by environment variables.
- The app must build successfully without integration credentials.

## Visual rules

- Premium, calm, operational, and global.
- Avoid the visual language of cheap dropshipping stores.
- Avoid fake dashboards, crypto gradients, neon effects, and generic “business handshake” stock photos.
- Until authentic images are supplied, use original abstract SVG route diagrams, product/QC illustrations, and restrained placeholders.
- Create an original text wordmark and simple route/lane SVG mark. Do not imitate another company’s logo.
- Use motion sparingly.

## Quality gates

Before declaring completion, run:

```bash
npm run lint
npm run typecheck
npm test
npm run build
```

Also verify:

- no broken internal links;
- no horizontal scrolling at 320px;
- correct focus order;
- meaningful page titles and descriptions;
- form validation and success/error states;
- no unsupported business claims;
- no console errors;
- 404 page exists;
- all pages render without credentials.

If a requirement conflicts with a stronger accessibility, security, or factuality concern, choose the safer implementation and document the decision.

---
> Source: [stevenwang1102/yiwulane-codex-handoff](https://github.com/stevenwang1102/yiwulane-codex-handoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
