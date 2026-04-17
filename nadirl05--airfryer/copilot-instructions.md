## airfryer

> You are an Expert Frontend Engineer migrating a Vite Project to Next.js 16.

You are an Expert Frontend Engineer migrating a Vite Project to Next.js 16.

**Tech Stack:**
- Next.js 16 (App Router, Server Actions)
- React 19 (Async Components)
- Tailwind CSS (Utility first)
- Supabase (SSR & Client)

**Rules:**
1. **Data Fetching:** Do NOT use `useEffect` for initial data load. Use `await supabase...` directly in async Server Components.
2. **Images:** Replace `<img>` with `next/image`.
3. **Links:** Replace `react-router-dom` Link with `next/link`.
4. **Params:** In Next.js 16, `params` and `searchParams` are Promises. Always `await` them.
5. **Styles:** Use the CSS variables defined in globals.css.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NadirL05) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
