## clerk-waitlist-demo

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Learned User Preferences

- Embed the Clerk waitlist on the marketing home page (anchor `#waitlist`, `ClerkProvider waitlistUrl="/"`) instead of a separate `/waitlist` route unless asked otherwise.
- Keep the join UI minimal: render `<Waitlist />` without an extra marketing card wrapper; put Clerk setup hints in a separate collapsible, not around the form.
- For a waitlist-focused funnel, prefer a brand-only header or no header auth buttons so attention stays on the form.
- On dark landings, verify the real `body` background—Tailwind v4 preflight can yield a light canvas; reinforce with `@layer base` for `html`/`body` and/or inline `backgroundColor` on `<body>`.
- Match embedded Clerk UI to dark pages (e.g. `@clerk/themes` `dark` base theme and aligned `appearance.variables`).
- Use split heroes at `md` (or similar), not only `lg`, so copy and the waitlist sit side by side on tablets.
- Prefer Tailwind v4 `bg-linear-to-*` class names where the toolchain suggests them over `bg-gradient-to-*`.

## Learned Workspace Facts

- The demo landing product name is **Relaymind** (dummy AI SaaS for docs and team alignment).
- Clerk request handling uses root `proxy.ts` with `clerkMiddleware()` from `@clerk/nextjs/server` (Next 16 proxy convention).
- Optional Clerk webhooks live at `app/api/webhooks/clerk/route.ts` via `verifyWebhook`; waitlist events include `waitlistEntry.created` and `waitlistEntry.updated`.
- Magic UI registry components (e.g. `@magicui/globe`, `@magicui/bento-grid`) install with `pnpm dlx shadcn@latest add …`; `components.json` / shadcn init must exist first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sonnysangha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sonnysangha)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
