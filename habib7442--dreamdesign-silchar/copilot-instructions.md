## dreamdesign-silchar

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# dream-silchar

Rebuild of dreamdesignsilchar.com — Dream Design Silchar, an interior
design studio and civil contractor in Meherpur, Silchar, Assam. The
site's only job is converting homeowners into WhatsApp leads; see
`DESIGN.md` for the reasoning and `docs/prd.md` for the local-SEO
research behind the service pages.

**Read `DESIGN.md` before writing or editing any UI.** Every color,
radius, spacing, and type size must trace back to a token defined
there. Do not introduce a value — a shadow, a pill radius, a warm
cream background, weight 700 — that isn't in that file. If a design
decision isn't covered by it, treat that as a gap to flag, not license
to improvise a default.

## Stack

- Next.js 16.2.4 (App Router) — see the breaking-changes note above
  before touching routing, data fetching, or metadata APIs.
- React 19, TypeScript.
- Tailwind CSS v4 (`app/globals.css`, `postcss.config.mjs`) — no
  Tailwind v3 config file exists; don't add one.
- shadcn/radix-ui primitives in `components/ui/`, extended via
  `class-variance-authority` and `tailwind-merge`. Check for an
  existing primitive there before writing a new one from scratch.
- `framer-motion` for the two motion patterns defined in `DESIGN.md`
  §6 — page-load reveal and scroll-in fade. Nothing beyond that.
- `lucide-react` for icons, thin-line per `DESIGN.md` §5.

## Structure

- `app/` — route segments (`about`, `contact`, `portfolio`, `services/*`).
  `services/*` holds one route per service; keep new services in that
  pattern rather than a query-param or single catch-all page.
- `components/` — `navbar.tsx`, `footer.tsx`, and shared UI in `ui/`.
- `lib/utils.ts` — the `cn()` helper and other shared utilities.
- `docs/prd.md` — raw research (competitors, keyword targets, client
  history). Background context, not a spec — `DESIGN.md` is the spec.

---
> Source: [Habib7442/dreamdesign-silchar](https://github.com/Habib7442/dreamdesign-silchar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
