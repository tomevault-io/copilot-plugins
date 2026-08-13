## chrissys-studio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Artist portfolio website for Christine's Studio, built with **Nuxt 4** and **Vue 3** (Composition API). Single-page application with section-based navigation (Gallery, Shop, About, Contact).

## Commands

- `npm run dev` — Start dev server (http://localhost:3000)
- `npm run build` — Production build
- `npm run generate` — Static site generation (SSG)
- `npm run preview` — Preview production build
- `npm install` — Install dependencies (runs `nuxt prepare` via postinstall)
- `npm run lint` — Run ESLint
- `npm run lint:fix` — Run ESLint with auto-fix
- `npm run format` — Format code with Prettier
- `npm run format:check` — Check formatting without writing
- `cd studio && npm run dev` — Start Sanity Studio (http://localhost:3333)

## Architecture

**Single-page app** — All content lives in `app/pages/index.vue`, which composes section components. Navigation uses smooth-scroll to anchor IDs (`#gallery`, `#shop`, `#about`, `#contact`).

**Component hierarchy:**

```
app.vue → pages/index.vue
  ├── NavBar (fixed header, emits 'navigate' for scroll)
  ├── HeroSection
  ├── GallerySection (category filtering: all/landscape/still-life/study)
  │   └── PaintingCard (reusable, hover effects, sold badge)
  ├── ShopSection (prices, available count)
  │   └── PaintingCard
  ├── AboutSection
  ├── ContactSection (form with inquiry types, placeholder submit)
  ├── FooterSection
  └── PaintingModal (Vue Teleport, detailed view with purchase flow)
```

**State management:** Local component state only via `ref()` — no Pinia/Vuex. Key state in `index.vue`: `activeSection`, `scrolled`, `selectedPainting`. Parent-child communication through props and emits.

**Data:** Paintings are managed in Sanity CMS (project ID: `3i77cmnf`, dataset: `production`). The `usePaintings()` composable in `app/composables/usePaintings.js` fetches from Sanity via GROQ and falls back to mock data in `app/data/paintings.js` if Sanity returns nothing. Sanity Studio lives in `studio/` with its own `package.json`.

## Payments

Stripe Checkout (redirect flow) handles purchases. The server route `server/api/checkout.post.ts` creates a Checkout Session and returns the Stripe-hosted URL. After payment, customers land on `/success`. Cancel returns to `/#shop`.

## Webhook & Post-Purchase

The server route `server/api/webhook.post.ts` handles Stripe `checkout.session.completed` events. On a successful payment it:

1. Marks the painting as sold in Sanity (sets `sold: true` via `server/utils/sanity.ts` write client)
2. Sends a purchase confirmation email to the buyer via Resend
3. Sends a sale notification email to Christine via Resend

### Environment Variables

All env vars are in `.env` and exposed to server routes via `runtimeConfig` in `nuxt.config.ts`:

- `STRIPE_SECRET_KEY` — Stripe API key
- `STRIPE_WEBHOOK_SECRET` — Stripe webhook signing secret (from Stripe Dashboard or `stripe listen`)
- `RESEND_API_KEY` — API key from resend.com
- `SELLER_EMAIL` — Christine's email for sale notifications
- `SANITY_API_TOKEN` — Sanity write token (Editor role, from sanity.io/manage)

### Local Webhook Testing

1. Install the [Stripe CLI](https://stripe.com/docs/stripe-cli)
2. Run `stripe listen --forward-to localhost:3000/api/webhook`
3. Copy the webhook signing secret to `.env` as `STRIPE_WEBHOOK_SECRET`
4. Run `npm run dev` and test with card `4242 4242 4242 4242`

## Code Style

- Do not add comments to code. The code should be self-explanatory.

## Styling

- Plain CSS with CSS Custom Properties defined in `app/assets/css/main.css`
- Scoped styles in each component
- Fonts: Cormorant Garamond (display), DM Sans (body) via Google Fonts
- Color palette: warm earth tones (cream, gold, brown, tan)
- Single responsive breakpoint at `768px`

---
> Source: [melissapula/chrissys-studio](https://github.com/melissapula/chrissys-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
