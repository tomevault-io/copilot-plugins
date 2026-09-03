## nexpathadmin

> Vite + React admin dashboard for managing NexPath resources.

# NexPath Admin

Vite + React admin dashboard for managing NexPath resources.

## Setup

```bash
npm install
npm run dev
```

## Supabase

Public credentials live in `src/lib/supabase/config.ts`. Admin mutations run in the browser with the signed-in user's session. Ensure Supabase Row Level Security policies allow authenticated admin writes.

## Scripts

- `yarn dev` — React Router / Draftbit dev server
- `yarn build` — production build to `build/client` (required by Draftbit deploy)
- `yarn preview` — preview production build

## Draftbit / React Router

This app uses React Router 7 framework mode (`ssr: false`) so CI can copy `build/client` into the worker package. Routes live under `src/app/`; page UI stays in `src/pages/`.

---
> Source: [draftbitexperts/nexPathAdmin](https://github.com/draftbitexperts/nexPathAdmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
