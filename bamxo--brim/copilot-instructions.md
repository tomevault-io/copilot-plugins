## brim

> Project architecture, file structure, and implementation checklist for brim


# Brim Project Architecture

## Stack

- **Framework**: Shopify Remix (React Router v7) embedded app in Shopify Admin iframe
- **UI**: Polaris web components (`s-*`) — NOT React Polaris (`@shopify/polaris`)
- **DB**: Supabase (service role key — RLS is enabled but bypassed server-side)
- **Auth**: `authenticate.admin(request)` from `@shopify/shopify-app-react-router/server`

## Route → View → Controller pattern

```
app/routes.ts                    ← route config
app/backend/<feature>/views.tsx  ← loader + action (server)
app/frontend/pages/<Name>Page.tsx ← React component (client)
app/backend/<feature>/controller.server.ts ← Supabase queries
```

## Every new feature checklist

- [ ] `loader` returns all data the page needs (parallel with `Promise.all`)
- [ ] `action` uses `intent` field to dispatch multiple operations
- [ ] Each mutation intent returns a **distinct response shape** (not just `{ success: true }`)
- [ ] **FormData from Polaris fields is harvested manually** via `querySelector + .value`
- [ ] Separate `useFetcher` per mutation concern (not shared `useSubmit` for everything)
- [ ] Every controller function returns `{ error: string | null }` — never fire-and-forget
- [ ] Shopify Admin GraphQL queries are validated against the current API version (October25)

## Key domain model facts

- Product ↔ Supplier links live in `reorder_rules` (not in `products` or `suppliers`)
- `reorder_rules.is_active = false` = product removed from supplier (soft remove)
- All tables have RLS enabled; the service role key bypasses it — never use the anon key server-side
- `purchase_orders` references `suppliers` with a nullable FK — set to null before deleting a supplier

## Shopify API version

`ApiVersion.October25` — always verify field names against this version before implementing GraphQL queries.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bamxo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
