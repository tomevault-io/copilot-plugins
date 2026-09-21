## pinkless

> This file is the always-loaded rulebook for any AI coding agent working in

# Agent guardrails for Pinkless

This file is the always-loaded rulebook for any AI coding agent working in
this repo (Codex, etc.). Claude Code users get the same rules as
auto-triggering skills in `.claude/skills/` — this file exists so agents that
don't read that format (Codex and others) still get them. Ground truth for
all of it is `Files/REQUIREMENTS.md`; when this file and REQUIREMENTS.md
disagree, REQUIREMENTS.md wins — re-read it if something here looks stale,
since it changes as the team builds.

Product in one line: on a Kroger product page for an item marketed to women,
Pinkless shows a reviewed men's or neutral equivalent that costs less **at the
same Kroger store**, with both prices from Kroger's official API. Kroger is the
only retailer (the CVS/Walmart cross-retailer design is superseded).

Architecture in one line: `apps/extension` extracts product identity on
kroger.com pages and sends only that identity + selected Kroger store to
`apps/api` (Vercel serverless), which holds the Kroger credentials, prices the
current product and its reviewed equivalents through the Kroger provider, and
returns a comparison; `packages/matcher` does eligibility/savings logic;
`packages/catalog` holds products (`products.json`) and reviewed
women's→men's/neutral pairs (`equivalences.json`); `apps/marketplace` calls the
same read-only API (`GET /api/comparisons`).

---

## 1. Frontend craft (`apps/marketplace`, extension popup/badge UI)

- Design tokens live in `packages/tokens/` (`tokens.json` is source of truth,
  `tokens.css` is compiled). See `DESIGN_SYSTEM.md`.
- Always style with the CSS variables (`var(--surface-pink)`, `var(--ink)`,
  `var(--space-4)`, `var(--radius-md)`, etc.) and the type classes
  (`.display`, `.heading`, `.body`, `.caption`, `.label`). **Never** write a
  raw hex value or a magic font-size/spacing number. If a needed value isn't
  a token yet, stop and ask — don't invent one (there's an open question
  about primary/CTA color; don't silently resolve it).
- The extension badge renders inside a Shadow DOM and cannot inherit
  page-level CSS. Never link an external stylesheet into the shadow root —
  use `mountBadgeRoot()` from `apps/extension/src/content/shadow-root.ts`,
  which injects the generated `TOKENS_CSS` into the shadow root's own
  `<style>`. **Render all badge markup inside the `container` it returns:**
  the token variables live on `:root, [data-theme="light"]`, `:root` never
  matches in a shadow tree, so they only exist on/below the container's
  `data-theme="light"`. After changing `packages/tokens`, run
  `pnpm run tokens:sync`; `pnpm run tokens:check` (part of `build` and CI)
  fails on drift between `tokens.json`, `tokens.css`, and the extension copy.
- Accessibility bar (REQUIREMENTS §6), non-negotiable for every badge/popup
  control: keyboard-operable with descriptive accessible labels; no
  assertive ARIA live regions; never steal focus on mount/update; never
  render so it obscures a price, checkout control, or native accessibility
  element. The primary action opens the outbound URL in a new tab — it must
  not navigate the host page away.
- The Marketplace is a Vercel-deployed React app that calls the same
  read-only comparison API (`apps/api`) as the extension — it doesn't fetch
  retailer prices directly, and has no login, checkout, or user tracking.

## 2. Extension MV3 (`apps/extension/**`)

- Content script runs **only** on declared Kroger domains —
  explicit `matches` entries in `manifest.json`, never a broad wildcard.
- One page adapter per retailer, producing a normalized `ProductView`:
  ```ts
  type ProductView = {
    retailer: string;
    canonicalUrl: string;
    productId?: string;
    upc?: string;
    title: string;
    selectedVariant?: string;
    currentPriceCents?: number;
    currency?: string;
    availability: "in-stock" | "out-of-stock" | "unknown";
  };
  ```
- The extension must **never** contain retailer credentials or request
  arbitrary page history. It may send only the current product identity and
  selected retailer/store location to the Pinkless API (`apps/api`) solely to
  obtain a comparison — no other outbound call, no analytics/telemetry.
- Re-evaluate with a **debounced** `MutationObserver` on variant change or
  client-side navigation — debounce so a burst of DOM churn triggers one
  recomputation, not many. Never leave a stale savings badge visible after a
  recompute; if the new state fails any check, remove/hide the badge.
- Render **exactly one** badge, ever, inside a Shadow DOM root, with a fixed
  unique root element ID. On remount, replace the existing root's contents —
  never append a second badge instance.
- Matching/eligibility logic belongs in `packages/matcher` / the API, not
  duplicated inline in the content script.

## 3. API and provider adapters (`apps/api/**`)

- Keep all retailer credentials in Vercel environment variables; never
  expose them to the extension or Marketplace, and never use a client-visible
  `VITE_` prefix for them.
- One provider adapter per retailer (`apps/api/src/providers/**`), each
  returning the shared `Offer` shape — don't leak a provider's raw response
  shape upward.
- Cache by `retailer + product ID/UPC + location + fulfillment method`, with
  an observation timestamp and provider-specific TTL/rate limit.
- Distinguish online and store-specific prices. Never present an online price
  as a local-store price, or compare unlike fulfillment contexts (`online` vs
  `store-pickup`/`in-store`) as equivalent.
- Fail closed: no approved credentials, ambiguous identity, no price, or
  unavailable → suppress that retailer's offer, never a partial or invented
  price.
- `PINKLESS_PROVIDER_MODE=mock` enables deterministic local Kroger fixtures
  (`apps/api/src/providers/mock-data.ts`); any other value uses the
  fail-closed live registry. Kroger is the only provider; it uses
  `KROGER_CLIENT_ID`/`KROGER_CLIENT_SECRET` with the `product.compact` scope
  only (never Cart/Profile). Don't wire in other retailer keys (including
  scraped-data services such as Canopy) without a REQUIREMENTS.md change.
- The API prices **both** sides of a comparison itself, at the same store and
  price context. The page price is only a consistency check; a mismatch
  suppresses.
- `.env.example` documents variable **names** only, never real values. `.env`
  and `.env.*` are gitignored — never commit them.

## 4. Catalog and matcher (`packages/catalog/**`, `packages/matcher/**`)

This is the product's trust boundary. Treat every rule here as a hard
constraint.

- Money is always integer minor units: `amountCents`. **Never** use a
  floating-point value for a price, anywhere, including intermediate
  arithmetic. Flag any PR that introduces float pricing.
- Exact shared types live in `packages/catalog/src/schema.ts` and
  `Files/REQUIREMENTS.md` §4: `Product` (with `marketedTo: "women" | "men" |
  "neutral"`, no `equivalence` field), `ProductEquivalence` (exactly one
  women's product + one men's/neutral product, `rationale`,
  `matchedAttributes`, non-empty `knownDifferences`, `reviewedBy`,
  `reviewedAt`, `status`), `Offer`, `RetailerIdentity` (`retailer: "kroger"`).
  Don't resurrect the older `Comparison`/`TargetListing`/`AlternativeListing`
  shapes or the per-product `equivalence` block. Don't add fields, loosen
  types, or make required fields optional without updating
  `Files/REQUIREMENTS.md` in the same PR.
- Catalog validation (`pnpm run catalog:validate`) must reject: duplicate
  product IDs, UPCs, or retailer identities; an `active` product without a
  Kroger identity; invalid sizes, URLs, or identity patterns; an equivalence
  that references a missing product, pairs a product with itself, repeats a
  pair, isn't exactly one women's + one men's/neutral product, crosses
  categories or size units, links an inactive product while active, or lacks a
  rationale, known differences, reviewer, or review date.
- Matching resolves an active product by, **in this order**: exact UPC, Kroger
  product ID, then canonical URL pattern. Title/category matching may help a
  human diagnose a mismatch, but must never independently trigger a badge.
  Only a women's product with an active reviewed pair is compared. Both offers
  must be USD, in stock, positive, unexpired, Kroger's **regular** price (never
  promo), at the **same store and price context**. Savings = women's price −
  men's/neutral price scaled to the women's amount (integer math, rounded up,
  `packages/matcher/src/unit-price.ts`); zero or negative → `no-match`.
- Per-unit comparison only within one size unit (oz↔oz, count↔count); never
  across units or categories. No loyalty/membership prices, and being the same brand never makes two
  products equivalent. Every pair is written and reviewed by a person — never
  inferred automatically.
- Copy may state who a product is marketed to (as its listing says), prices,
  store, and date. It must never say why prices differ or claim
  discrimination.
- Matching stays deterministic and reviewed. Never introduce fuzzy string
  matching, embeddings, or an AI/LLM call to decide comparability or whether
  a badge should render.

## 5. Suppress by default (any matcher/adapter/provider/badge-mount logic)

Silence is the default. A weak match, unknown price, unavailable offer, or
non-positive savings must never produce a badge.

| Situation | Required behaviour |
| --- | --- |
| Sale, coupon, membership, subscription, or "from" price | Suppress unless both offers identify equivalent ordinary one-time purchasable prices. |
| Price range or unparseable currency | Suppress. |
| Current price is at or below alternative price | Suppress. |
| Different size, refill, bundle, condition, or pack count | Suppress unless a reviewed catalog record explicitly covers it. |
| Current product or alternative offer is out of stock | Suppress. |
| One offer is online and the other is store-specific | Suppress rather than imply an equivalent local-store price. |
| Kroger credentials or API unavailable | Suppress; return no partial or invented price. |
| Offers from different Kroger stores, or no store selected | Suppress. |
| Page price disagrees with the provider price | Suppress. |
| Product is not marketed to women, or has no active reviewed pair | Stay quiet. |
| Third-party marketplace seller | Exclude from the initial catalog. |
| Page is an ad, search result, category page, or quick-view modal | Suppress. |
| Retailer changes DOM / extracted data is incomplete | Suppress, log a development-only diagnostic, rely on fallback demo page. |
| Client-side route/variant change | Debounce and recompute; never leave stale savings visible. |
| Duplicate/injected UI collision | Use a fixed unique root ID and Shadow DOM; replace rather than append. |
| Cached offer is expired | Revalidate through the provider; suppress it if refresh fails. |
| Gender marketing is ambiguous | Do not publish the comparison until reviewer documents the rationale. |

Anything not explicitly covered above or by a reviewed catalog record:
**suppress**. Never add a "best guess" fallback or a permissive default to
make the demo look more populated.

## 6. Demo readiness (pre-demo / pre-merge of the extension↔api↔matcher↔marketplace path)

- A known supported product shows one correct badge in under three seconds
  **on a warm cache**.
- Savings equal `women's product regular price - men's/neutral product
  regular price` at the same store, exactly (integer cents, no rounding
  drift).
- Clicking the badge opens the expected alternative URL in a new tab.
- Unknown product, non-product page, out-of-stock item, and non-positive
  savings all stay quiet.
- Changing a supported product's variant updates or removes the badge —
  never leaves a stale one.
- The Marketplace identifies the store, price context, and observed time for
  every displayed offer.
- `pnpm run catalog:validate` runs cleanly.
- Fixture-based tests cover the Kroger page adapter, the Kroger provider,
  and the matcher's key suppression rules.
- The unpacked extension and Vercel Marketplace use only the Pinkless API and
  Kroger's official API — no page scraping, anywhere.
- Standing reminder: `fixtures/retailers/` and `fixtures/providers/`, plus
  the fallback demo page, exist for when a retailer's DOM changes
  mid-judging. Re-run fixtures whenever an adapter or matcher changes; treat
  a broken `PINKLESS_PROVIDER_MODE=mock` path as demo-blocking; fix the
  fallback page in the same PR if it drifts from what an adapter now
  expects.

## 7. Git flow (multiple people/agents committing to this repo concurrently)

- Branch per feature: `feat/<area>-<short-desc>`; `fix/<area>-<short-desc>`
  for bugs; `chore/<short-desc>` for tooling/docs/deps. Keep branches scoped
  to one area (`apps/extension`, `apps/api`, `apps/marketplace`,
  `packages/catalog`, `packages/matcher`, `packages/tokens`) to minimize
  conflicts.
- Conventional commits: `feat: <what>`, `fix: <what>`, `chore: <what>`.
- Before merging to `main`: run `pnpm run catalog:validate`,
  `pnpm run test`, and `pnpm run typecheck` (or `pnpm run build`, which
  chains all three plus both app builds). Don't merge on a failure and "fix
  it after" — a broken `main` blocks everyone immediately.
- No force-pushing `main`. No direct push to `main` that skips the build
  check above, even for a "trivial" change. Never commit `node_modules/`,
  build output (`dist/`), or `.env`/`.env.*` files — retailer credentials
  live in Vercel, never in the repo; `.env.example` documents variable names
  only.

---
> Source: [jsberesford/Pinkless](https://github.com/jsberesford/Pinkless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
