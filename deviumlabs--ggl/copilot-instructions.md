## ggl

> - Institutional website for GGL Móveis de Aço (B2B).

# AGENTS.md — GGL Móveis de Aço (Next.js 15 / Pages Router)

## Product context
- Institutional website for GGL Móveis de Aço (B2B).
- Key SEO surface: product pages and category structure under /produtos.
- Copy language: pt-BR.

## Stack
- Next.js 15 (Pages Router), React 18
- TailwindCSS (prefix: tw-), PostCSS
- Axios service wrapper (SSR-safe baseURL)
- Resend email API (/pages/api/mail.js)
- GTM + consent (dataLayer)
- Dynamic product routing: /produtos/[categoria]/[produto]
- Redirects and middleware support legacy URLs

## Commands
- Install: npm i
- Dev: npm run dev
- Build: npm run build
- Start: npm run start
- Lint: npm run lint

## Non-negotiables (do not break)
- Keep Pages Router (do not migrate to App Router unless explicitly requested).
- Do not introduce new dependencies without explicit request.
- Preserve analytics events (dataLayer / dlPush) and consent behavior.
- Preserve SEO head tags and JSON-LD where applicable.
- Avoid duplicate Header/Footer rendering (either layout-based or per-page, not both).
- Routing must not reuse different param names for the same dynamic segment.

## SEO rules (high priority)
- Every indexable page should have: title, meta description, canonical.
- Product page must include Product JSON-LD and stable canonical URL.
- Legacy URL patterns must redirect (301) to the canonical route.
- 404 should be a true 404 response (Next notFound / 404 page) and not indexable.

## UI/UX rules
- Mobile first: nothing should overlap product content.
- Navbar: keep interaction predictable; active state must be visible.
- Keep brand colors from Tailwind config (blue #0058C2, darkBlue #0F172A).

## Data and safety
- Never log PII from contact forms.
- Never hardcode emails/tokens outside env-configurable defaults.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DeviumLabs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DeviumLabs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
