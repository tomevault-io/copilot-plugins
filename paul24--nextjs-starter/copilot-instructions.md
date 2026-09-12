## nextjs-starter

> - Keep the default experience compatible with `output: "export"`; `pnpm build:static` is a release gate.

# Repository guidance

- Keep the default experience compatible with `output: "export"`; `pnpm build:static` is a release gate.
- Use Server Components unless interaction, browser state, or a client-only library requires a narrow Client Component.
- Enumerate every dynamic route with `generateStaticParams()`.
- Do not add cookies, request headers, Server Actions, runtime route handlers, ISR, authentication, or request-time database calls to the static path.
- Store money in integer minor units and format it through `src/lib/formatting/intl.ts`.
- Put branding in `src/config`, reusable records in `src/data`, and provider contracts in `src/lib` or `src/types`.
- Preserve keyboard access, visible focus, semantic headings, reduced-motion support, and meaningful alt text.
- Use semantic design tokens instead of component-level colour literals.
- Run `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build:static`, and `pnpm build` before release.
- Review every new dependency for necessity and static-export compatibility.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [PAUL24/nextjs-starter](https://github.com/PAUL24/nextjs-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
