## sphinx-strength

> - Never create a file longer than 500 lines of code. If a file approaches this limit, refactor by splitting it into modules or helper files.


## General Rules

- Never create a file longer than 500 lines of code. If a file approaches this limit, refactor by splitting it into modules or helper files.
- Always extract any UI element that repeats or could be reused (buttons, input fields, cards, forms, ...) into a separate component file. Do not implement detailed UI or logic directly inside page files; pages should only import and compose these components.
- All styling in the app should be done using Nativewind (TailwindCSS) and the `@/styles/global.css` file.
- Always use `expo-linear-gradient` for any gradient effects. Never use CSS gradients or TailwindCSS gradient classes. Always apply styles to LinearGradient components using the `style` prop with regular React Native StyleSheet objects, not NativeWind/TailwindCSS classes.
- Always use the `@components/CustomButton.tsx` component whenever a button is needed and customize it with props to achieve the desired results.
- Organize code into clearly separated modules, grouped by feature or responsibility.
- Always define TypeScript types in the `types` folder at the project root. Place generic, widely used types in `types/type.d.ts`, and create separate files (e.g. `driver.d.ts`, `products.d.ts`) for domain-specific types. Never define types inline in components or pages.

## Database access & API routes

- Never run privileged queries (insert/update/delete with service role key, or wide `select`) directly from client-side code. Instead:
  1. Create an Expo Router server route by adding a file that ends with `+api.ts` inside the `app/` directory. The file must export a corresponding HTTP-verb function (`GET`, `POST`, etc.).
  2. Inside that file instantiate a `supabaseAdmin` client using the environment variable `SUPABASE_SERVICE_KEY` (not prefixed with `EXPO_PUBLIC_`); safe because API routes are server-only.
  3. Implement the DB logic and return a JSON response with `Response.json(...)`. Handle errors and translate PostgREST codes when helpful.

- On the client create a thin helper in `api-client/` that performs `fetch('/api/route', …)` and returns parsed JSON. Components/screens may only call these helpers (never the DB directly).

## Global state

- Use the central Redux store (`store/index.ts`) for any data that needs to persist across screens.
  – `userSlice` holds auth/user
  – Create new slices when new domains emerge; keep each slice small and co-located with feature code.
- Do not store supabase session objects in Redux. Rely on `supabase.auth` listeners instead and only keep derived data in the store.
- Keep slices < 300 lines; split actions/thunks into feature files if they grow.

## Docs

Always use context7 (the MCP server) to fetch the latest documentation for any library or framework you are working with. Never rely solely on outdated knowledge; check context7 to confirm APIs, usage examples, and best practices before implementation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rathetsu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
