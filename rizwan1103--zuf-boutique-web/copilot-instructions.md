## zuf-boutique-web

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This workspace uses Next.js 16 with the App Router and Tailwind CSS. APIs, conventions, and file structure may differ from older Next.js releases. Read the relevant guide in `node_modules/next/dist/docs/` before writing code.
<!-- END:nextjs-agent-rules -->

# Workspace Agent Instructions

## Project overview
- Minimal Next.js 16.2.4 application created with `create-next-app`.
- Uses **App Router** under `app/`.
- Written in **TypeScript** with **Tailwind CSS v4**.
- No custom API routes or backend services are present.

## Key files
- `app/page.tsx` - main page content and best entry point for local edits.
- `app/layout.tsx` - root layout, metadata, and global font setup.
- `app/globals.css` - global styles.
- `next.config.ts` - Next.js configuration stub.
- `eslint.config.mjs` - ESLint setup using Next.js defaults.
- `package.json` - scripts and dependency versions.

## Recommended workflow
1. Read `README.md` for the initial project purpose.
2. Use `npm install` if dependencies are not installed.
3. Run `npm run dev` to start the local Next.js dev server.
4. Edit `app/page.tsx` or `app/layout.tsx` for UI changes.
5. Run `npm run lint` before larger refactors.

## Important conventions
- Prefer **App Router** patterns and file-based routing with `app/`.
- Keep components as React server components unless client interactivity is needed.
- Use Tailwind utility classes for styling.
- Do not add routing files under `pages/`; this project uses the `app/` directory only.

## Notes for agents
- The app is currently a starter template; most work will be on `app/`.
- There is no monorepo or multi-package structure.
- Avoid guessing missing business logic; this is a scaffold-ready storefront project.
- No test setup is present, so focus on app structure and linting rules.

## Example prompts
- "Update the homepage in `app/page.tsx` to show a boutique landing section with hero text and featured products."
- "Add a new responsive hero section to `app/page.tsx` using Tailwind CSS."
- "Refactor `app/layout.tsx` so the page uses a custom metadata title and description."
- "Add a new global CSS utility class to `app/globals.css` and use it in `app/page.tsx`."

---
> Source: [rizwan1103/zuf-boutique-web](https://github.com/rizwan1103/zuf-boutique-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
