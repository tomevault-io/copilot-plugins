## website

> Monorepo for **Spark901** public web presence (`apps/web`, Next.js). Spark901 builds **open-source software** so nonprofits and mission-driven organizations can do more with less. **Donations and sponsorships** fund engineering and infrastructure; the product intent is **lasting quality** (maintainable code, honest messaging, tools that scale across many orgs).

# Spark901 website — Claude Code context

## What this repository is

Monorepo for **Spark901** public web presence (`apps/web`, Next.js). Spark901 builds **open-source software** so nonprofits and mission-driven organizations can do more with less. **Donations and sponsorships** fund engineering and infrastructure; the product intent is **lasting quality** (maintainable code, honest messaging, tools that scale across many orgs).

## Mission and positioning

- **Geography**: **Memphis, Tennessee** (area code **901**). Local to the Mid-South; audience is global but roots are Memphis.
- **Nonprofit ecosystem**: Memphis is a **nonprofit hub**. The team is part of the **Digital Delta**—a frame for regional tech talent **supporting** nonprofits, not competing with them.
- **Legal**: Spark901 operates as an **LLC**, **not** a **501(c)(3)** at this time. **Contributions are generally not tax-deductible.** Any copy about taxes or “nonprofit status” must stay accurate (see `transparency` strings and legal pages).
- **Ethos**: Social good through **public goods** (open source), transparency, and **rigorous** engineering—not through misleading donors about tax treatment.

## Memphis nonprofit hub statistics (for copy/UI)

Single source of truth: `apps/web/lib/brand.ts` → `memphisNonprofitHub`.

1. Memphis **leads major U.S. metros** in nonprofits **relative to population**: **69.9 per 10,000** residents.
2. **Over 6,500** tax-exempt nonprofits in the Memphis area, **employing over 96,000 people**, with **over $46 billion** in assets.

UI: use the `MemphisNonprofitHubStats` component and `messages/*.json` under `memphisNonprofitHub` so EN/ES stay aligned.

## Domain

- Canonical site: **`https://spark901.com`** (not `.org`).
- Contact email pattern: `@spark901.com` (e.g. `hello@spark901.com`).
- Stripe webhook / Checkout return URLs use `spark901.com`.

## Conventions for agents

- Match existing patterns: `next-intl`, `components/ui/*`, `lib/brand.ts` for brand constants.
- Do not add tax-deduction promises or “we are a nonprofit” claims without explicit legal verification.
- When adding fundraising or impact copy, tie it to **open infrastructure** and **verified** legal disclosures.
- **Public forms / lead APIs**: protect with Cloudflare Turnstile (`TurnstileField` + `verifyTurnstileToken` in `apps/web`). Env: `NEXT_PUBLIC_TURNSTILE_SITE_KEY`, `SPARK901_TURNSTILE_SECRET_KEY`. See `.cursor/rules/spark901-turnstile.mdc`. Skip for Stripe Checkout/portal and signed webhooks only.

---
> Source: [Spark-901/website](https://github.com/Spark-901/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
