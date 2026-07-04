## financial-dashboard

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Project Conventions & Lessons Learned

## Next.js v16 & React 19 Practices

- **Proxy over Middleware**: Do not create or use `middleware.ts`. Use `src/proxy.ts` (exporting a function named `proxy`) for request-routing / edge logic.
- **Rule of Hooks & State Resetting**: Avoid setting state inside a `useEffect` body to synchronize props to local state. Instead, use key-based component resetting (e.g., `<SubComponent key={entity.id} />`) to automatically recreate the component with fresh initial state.
- **React Compiler & Memoization**: Do not pass locally computed arrays/objects that change on every render into `useMemo` dependencies. Compute those arrays inside the `useMemo` block itself, keeping the dependency array stable (e.g., depending on direct props).
- **Client Mounting / Portal Hydration**: To avoid hydration mismatch warnings for client-only portals, defer `setMounted(true)` to a microtask using `queueMicrotask` in `useEffect`.

---
> Source: [giorobert88/financial-dashboard](https://github.com/giorobert88/financial-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
