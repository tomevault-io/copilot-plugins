## font-preloads-route-aware

> Route-aware Roboto font preload policy — which weights to preload and where.


# Route-aware font preloads

## Policy

Only preload Roboto weights that render above the fold. Keep all `@font-face` declarations in `_font-faces.scss`; on-demand weights still load via CSS with `font-display: swap`.

| Weight | File | Preload |
| --- | --- | --- |
| Regular (400) | `Roboto-Regular.ttf` | Always (`full` and `minimal`) |
| Light (300) | `Roboto-Light.ttf` | `full` only (headings, form controls) |
| Bold (700) | `Roboto-Bold.ttf` | `full` only |
| Italic (400 italic) | `Roboto-Italic.ttf` | Never |
| Medium (500) | `Roboto-Medium.ttf` | Never |

## `@podverse/ui` — `FontPreloads`

- **`variant="full"`** (default): Regular + Light + Bold.
- **`variant="minimal"`**: Regular only.
- Do not add embed pathname detection here; keep the component presentational.

## apps/web

- Root layout uses **`RouteAwareFontPreloads`**, not raw `<FontPreloads />`.
- **`full`**: main app routes and `/embed` demo index (`isEmbedPathname` returns false for `/embed`).
- **`minimal`**: chromeless `/embed/*` iframe routes (`/embed/<resource>`).
- Embed detection stays in the app via [`isEmbedPathname`](apps/web/src/lib/embed/isEmbedPathname.ts) — same signal as `AppChrome`.

## apps/management-web

- Root layout keeps `<FontPreloads />` with default `full`. No embed routes; no route-aware wrapper needed.

## When adding font weights

- Add `@font-face` in `_font-faces.scss` first.
- Preload only if the weight is used above the fold on full-chrome pages.
- Do not restore blanket preloads of all weights from the root layout.

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
