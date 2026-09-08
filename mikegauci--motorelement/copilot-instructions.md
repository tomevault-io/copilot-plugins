## motorelement

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start Next.js dev server (port 3000)
- `npm run build` — production build
- `npm run lint` — ESLint via `next lint` (config: `next/core-web-vitals` + `next/typescript`)
- `npm run knip` — detect unused files / exports / deps (`knip@^6`)

There is no test runner configured in this repo.

## Required environment variables

See `.env.example`. The app will crash at runtime if these are missing:

- **Supabase**: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` (server-only, used by `lib/supabase/storage.ts#getServiceClient`). `SUPABASE_SECRET_KEY` is accepted as an alias.
- **Stripe**: `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`
- **fal.ai**: `FAL_API_KEY` (image generation in `app/api/generate` + `lib/api/fal.ts`)
- **Printify**: `PRINTIFY_API_KEY`, `PRINTIFY_SHOP_ID`
- **Resend**: `RESEND_API_KEY`

## Architecture

This is a **Next.js 14 App Router** TypeScript app for selling AI-customized car-enthusiast apparel. Three integrations carry the product flow: **fal.ai** generates artwork, **Stripe** collects payment, **Printify** fulfills the physical merch. **Supabase** is the system of record for auth, generated-image storage, products, customers, orders, and illustrations.

Path alias: `@/*` → repo root (set in [tsconfig.json](tsconfig.json)).

### Route groups

`app/` uses two Next.js route groups whose folder names do **not** appear in URLs:

- `app/(shop)/` — public storefront + signed-in `/account`. Includes `/product/[slug]`, `/cart`, `/checkout`, `/order-confirmation`, `/login`.
- `app/(admin)/admin/` — internal admin pages (dashboard, customers, orders, products, illustrations).

[middleware.ts](middleware.ts) gates `/admin/*` and `/account/*` behind a Supabase session; unauthenticated users are redirected to `/login`. The matcher excludes `_next/static`, `_next/image`, and `favicon.ico`.

### Supabase clients — three flavors, do not mix

- [lib/supabase/client.ts](lib/supabase/client.ts) — browser (`createBrowserClient`). Anon key. Use in `'use client'` components.
- [lib/supabase/server.ts](lib/supabase/server.ts) — RSC / route-handler (`createServerClient` + cookies). Anon key + user session. The `setAll` swallow is intentional — read-only cookie stores in Server Components are refreshed by middleware instead.
- [lib/supabase/storage.ts](lib/supabase/storage.ts) `getServiceClient()` — service-role key, **server-only**, bypasses RLS. Used by image-persistence helpers and webhooks. Never import into a client component.

`lib/supabase/queries/` holds typed DB query helpers per table (customers, illustrations, metrics, orders, products).

### Customizer / generation flow

The customer journey on a product page ([components/shop/customizer/ProductCustomizer.tsx](components/shop/customizer/ProductCustomizer.tsx)) is a multi-step state machine driven by [CustomizerContext](components/shop/customizer/CustomizerContext.tsx):

1. **Vehicle input** → uploaded car photo + customer notes posted to `POST /api/generate` (`action: 'submit'`).
2. **fal.ai job** — `app/api/generate/route.ts` uploads the photo and a fixed style reference (`public/style-reference/style-reference.png`) to fal, queues the model (`openai/gpt-image-2/edit` for cars, `fal-ai/nano-banana-2*` for backgrounds, `fal-ai/gemini-3-pro-image-preview/edit` for tweaks), and polls via `action: 'status'`. The same route handles background generation (`mode: 'background'`) and iterative tweaks (`tweakImageUrl` / `backgroundTweakImageUrl`).
3. **Persistence** — when a job completes, `lib/customizer/persistGeneratedToSupabase.ts` downloads the fal-hosted image and writes it to the `car-images` or `backgrounds` Supabase bucket, inserting a row in `generated_car_images` / `generated_backgrounds`. Failures are logged but do not break the response.
4. **Composite editing** — `CompositeEditor`, `TextLayerEditor`, `BackgroundPresets`, `WhiteGapEraser` etc. let the user position artwork + text on the t-shirt. The hook `useCompositeCanvas` rasterizes the layers; `lib/customizer/persistArtworkToSupabase.ts` and `persistCustomBackground.ts` save the final assets. Live Mockup (`MockupPreview`) overlays the design on a blank garment; the Preview modal opens a larger view.
5. **Cart & checkout** — `useCart` (a hook backed by `CartProvider`) holds items; `POST /api/checkout` builds a Stripe Checkout Session via `lib/stripe/helpers.ts#createCheckoutSession`, attaching the artwork URLs (front/back) in metadata.
6. **Fulfillment** — Stripe redirects to `/order-confirmation`. The `app/api/webhooks/stripe/route.ts` webhook creates the order row in Supabase and submits the Printify order via helpers in [lib/printify/helpers.ts](lib/printify/helpers.ts). `app/api/webhooks/printify/route.ts` receives shipping/status updates back from Printify.

### Printify integration

[lib/printify/client.ts](lib/printify/client.ts) is the only place that talks to `https://api.printify.com/v1` — it injects auth headers, throws on non-2xx, and provides `shopPath()` to scope to the configured shop. All other Printify helpers (`helpers.ts`, `variants.ts`, `uploads.ts`) go through `printifyFetch`.

### Images / Next image config

[next.config.mjs](next.config.mjs) only whitelists three remote-image hosts: Supabase public storage, `images.printify.com`, and `images-api.printify.com`. New external image sources must be added there or `<Image>` will reject them.

## Conventions

- **No code comments by default** — enforced by [.cursor/rules/no-comments.mdc](.cursor/rules/no-comments.mdc). Don't add `//`, `/* */`, or `/** */` blocks unless the user explicitly asks. Use clear naming instead.
- The codebase mixes single and double quotes / with-and-without semicolons across files — match the surrounding file's style rather than imposing a project-wide preference.
- Server-only modules (anything using `SUPABASE_SERVICE_ROLE_KEY`, `STRIPE_SECRET_KEY`, `PRINTIFY_API_KEY`, `FAL_API_KEY`) must never be imported from a `'use client'` component or its transitive deps.

---
> Source: [mikegauci/motorelement](https://github.com/mikegauci/motorelement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
