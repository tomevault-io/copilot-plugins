## bnb

> Patterns auth Next.js + Supabase - évite boucles et désync.


# Auth Next.js + Supabase - Patterns

## Fetch auth côté SERVER

- `supabase.auth.getUser()` dans layout/client → **risque de boucle** (fetch → setState → rerender → refetch)
- Faire les vérifications auth côté **middleware** ou **Server Components**

## useEffect + auth = danger

```tsx
// ❌ Peut causer boucle
useEffect(() => {
  supabase.auth.getUser().then(({ data }) => setUser(data.user));
}, [pathname]);  // pathname change → refetch → setState → rerender...
```

## Middleware = simple

- Pas de heuristiques (referer, etc.)
- Logique: `if (!session) redirect('/login')`
- Le reste côté layout server ou pages

## Client: createBrowserClient @supabase/ssr

- Utiliser `createBrowserClient` de `@supabase/ssr` (cookies)
- Pas `createClient` de `@supabase/supabase-js` (localStorage)
- Cookies = synchro middleware ↔ client

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/labelmaisoncg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
