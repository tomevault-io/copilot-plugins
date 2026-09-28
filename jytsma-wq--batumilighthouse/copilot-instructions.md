## batumilighthouse

> These instructions apply to the entire Batumi Lighthouse repository.

# AGENTS.md

## Scope

These instructions apply to the entire Batumi Lighthouse repository.

## Project

Batumi Lighthouse is a founder-led website development studio in Batumi for hotels, guesthouses, clinics, dental and aesthetic practices, beauty studios, salons, and local service businesses.

The site must feel human, clear, practical, local, and credible. Avoid AI/SaaS dashboard language, fake agency scale, vague manifesto copy, and technical decoration unless the task explicitly asks for it.

## Stack

- Next.js App Router
- TypeScript strict mode
- Tailwind CSS
- next-intl localized routing
- Prisma for form submission storage
- pnpm as the package manager

## Important Routes

- `/en`, `/ka`, `/ru`, `/tr`
- `/en/website-audits`
- `/en/pricing`
- `/en/work`
- `/en/templates` and 18 localized template detail pages
- `/en/about`
- `/en/contact`
- `/en/privacy`
- `/en/terms`
- Equivalent main pages for Georgian, Russian, and Turkish
- API routes: `/api/leads`, `/api/audits`
- Buyer preview routes: `/preview/[templateId]/...` (always noindex)
- Isolated demo sites: `/template-sites/[templateId]/...` (always noindex)

## Hard Rules

- Do not invent testimonials, logos, metrics, conversion lifts, fake clients, fake awards, fake reviews, fake locations, fake addresses, or fake opening hours.
- Do not break multilingual routing or locale-aware links.
- Do not remove forms, metadata, structured data, CTAs, privacy copy, or legal pages without a clear task-specific reason.
- Do not add heavy dependencies casually.
- Do not expose secrets or private environment variables.
- Do not send real production emails or spam during tests.
- Do not redesign unless the task explicitly asks for redesign.
- Do not make broad refactors during QA, launch, security, SEO, or accessibility tasks.
- Keep the Batumi Lighthouse business site as the root experience. Do not turn the homepage into the template showcase.
- Keep template demo rendering isolated from the localized studio layout, analytics controls, floating actions, and business navigation.
- Do not publish fictional testimonials inside template demos. Use an explicit no-review placeholder until verified, permissioned proof is supplied.

## i18n Rules

- Keep `src/content/messages/en.json`, `ka.json`, `ru.json`, and `tr.json` key-compatible.
- When adding or changing message keys, update every locale file or keep a safe fallback.
- Do not leave raw keys, `MISSING_MESSAGE`, or English-only UI in localized pages unless explicitly accepted.
- Preserve locale-aware navigation from `src/lib/i18n/navigation.ts`.
- Public template catalog and detail pages must be complete in all four locales. Raw fictional demo sites may remain English because they live outside localized public routes and are clearly disclosed.

## Design Rules

- Keep the founder-led local studio direction: warm, credible, restrained, and practical.
- Avoid radar/signal/dashboard visuals, fake browser chrome, excessive grids, neon technical effects, and generic agency hype.
- Do not change design direction during performance, security, SEO, analytics, or accessibility passes unless needed to fix a confirmed bug.
- Keep mobile layouts readable at 320px and avoid horizontal overflow.
- Preserve visual distinction across the 18 imported templates. Shared packages may handle primitives, models, tokens, forms, and routing, but not unique hero compositions or category-specific rhythm.

## Testing Rules

Run relevant checks before completion. For most code changes, run:

- `pnpm lint`
- `pnpm typecheck`
- `pnpm test`
- `pnpm build`

Also run:

- `pnpm test:e2e` when configured or supported
- `pnpm audit` for security tasks

Use Preview Browser for UI, layout, forms, navigation, accessibility, visual QA, analytics, and production launch tasks. If Preview Browser is unavailable after documented recovery, state that clearly and use the safest available fallback.

## Security Rules

- Server-side validation is required for form/API changes.
- Client-side validation is not enough.
- Do not log unnecessary PII.
- Keep honeypot/spam protection and rate limiting unless a stronger replacement is added.
- Missing env vars must fail gracefully with user-safe errors.
- Keep `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, disabled `X-Powered-By`, and CSP or CSP-Report-Only.
- Use strict CSP only when verified not to break Next.js, analytics, inline scripts, forms, or static generation.

## SEO Rules

- Keep page titles, meta descriptions, canonical URLs, hreflang alternates, Open Graph metadata, Twitter metadata, robots, sitemap, and JSON-LD valid.
- Do not add fake `aggregateRating`, review schema, physical address, or opening-hours schema.
- Keep Batumi, Adjara, Georgia, local SEO, multilingual pages, audits, hotel websites, clinic websites, beauty/salon websites, and contact/booking flow references natural.
- Do not keyword-stuff.

## Accessibility Rules

- Keep one H1 per page and logical heading order.
- Links and buttons need meaningful accessible names.
- Forms need associated labels, clear errors, and accessible success/failure states.
- Decorative icons/SVGs/images should be `aria-hidden` or empty-alt.
- Preserve visible focus states, keyboard navigation, Escape behavior for dialogs/drawers, and reduced-motion behavior.
- Floating controls must not cover important form fields or CTAs.

## Forms, Env, And Analytics

- Forms must not expose stack traces to users.
- Do not track names, email addresses, phone numbers, form messages, website URLs, or other sensitive form content.
- Analytics must respect cookie consent and must not block navigation or form submission.
- `NEXT_PUBLIC_*` values are public; never put secrets there.
- Keep `.env.example` current when env usage changes.

## Deployment Checks

Before saying the site is safe to deploy, check:

- Required verification commands pass.
- Production build succeeds.
- `robots.txt` and `sitemap.xml` work.
- Security headers are present.
- No console errors, hydration errors, network 404s, raw i18n keys, broken images, or horizontal overflow on critical routes.
- Template preview iframes load, internal preview navigation works, demo routes are noindex, and raw demo headers permit same-origin framing without weakening normal page framing rules.
- Contact and audit forms validate safely.
- Cookie consent, language switcher, mobile drawer, CTAs, footer links, WhatsApp fallback, and back-to-top controls work.

## Final Response Requirements

Final responses for implementation, QA, launch, or deployment tasks must include:

- Files changed
- Commands run
- Results
- Remaining risks
- Whether the site is safe to deploy, when relevant

---
> Source: [jytsma-wq/batumilighthouse](https://github.com/jytsma-wq/batumilighthouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
