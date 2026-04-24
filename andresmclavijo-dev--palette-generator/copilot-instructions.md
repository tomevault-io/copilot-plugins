## palette-generator

> - **App:** usepaletta.io

# Paletta — Claude Code context

## Project
- **App:** usepaletta.io
- **Local:** /Users/andresclavijo/palette-gen
- **GitHub:** andresmclavijo-dev/palette-generator
- **Branch:** always work on `main` unless told otherwise (feature branches: `feat/[name]`)
- **Deploy:** Vercel auto-deploys on push to `main`

## Stack
- React + Vite + TypeScript + Tailwind CSS
- Zustand — global state
- chroma-js — color manipulation and post-processing
- react-colorful — color picker component (replaced custom canvas picker)
- Supabase — auth + database (ref: `rumhoaslghadluqhlwzr`)
- Stripe — payments ($9/mo or $79/yr)
- Vercel Serverless Functions — API routes at `/api/*`
- Anthropic API (Claude Haiku) — AI palette generation

⚠️ CRITICAL: This is a Vite + Vercel project, NOT Next.js. Never use `next/server`, `NextRequest`, `NextResponse`, or any `next/*` imports. Middleware and API routes must use standard Web APIs (`Request`, `Response`).

## Design Governance (pre-implementation checklist)
Before every change, verify:
1. **Design tokens** — no hardcoded colors, spacing, radius, or font sizes
2. **Copy consistency** — one verb per action everywhere (e.g., always "Delete" or always "Remove", not both)
3. **Component reuse** — never duplicate; share between desktop/mobile
4. **UX best practices** — destructive actions need confirmation, keyboard shortcuts match button flows
5. **Cross-platform parity** — web ↔ mobile ↔ plugin features must match
6. **Accessibility** — WCAG AA contrast, aria-labels, focus traps

## Key files & paths
- API routes: `api/` (Vercel serverless)
- Supabase client: check `src/lib/supabase.ts`
- Pro hook: `usePro()` — reads `is_pro` from Supabase `profiles` table
- Stripe webhook: `api/stripe-webhook.ts` — flips `is_pro=true` on payment
- OG image: `public/og-image.png` (1200×630)
- Sitemap: `public/sitemap.xml`
- Robots: `public/robots.txt`

## Supabase schema (key tables)
- `profiles` — `id`, `is_pro` (boolean), `stripe_customer_id`, `stripe_subscription_id`, set to true by Stripe webhook
- `saved_palettes` — `id`, `user_id`, `colors` (jsonb), `name`

## Infrastructure gotchas
- **Cloudflare** proxies usepaletta.io — can silently drop POST requests on www redirects. Always verify webhook delivery at the DNS layer after any webhook changes.
- **vercel.json** SPA catch-all rewrite must explicitly exclude `/api/*` routes or they get swallowed.
- **Supabase env vars** — double-check var names in Vercel dashboard if webhook stops working.

## Pro tier
- **Pricing:** $9/mo or $79/yr (~27% annual savings)
- Free: 3 saved palettes, 5 colors max, export with watermark, 3 AI prompts/day, CSS export, Normal + Protanopia accessibility lens
- Pro: unlimited saves, 6/7/8 colors, no watermark, full shade scales 50–900, Tailwind export, image extraction, all 5 accessibility lens modes, all preview mockups, AI unlimited, cloud sync, export without watermark
- **Test Pro on production:** load `usepaletta.io?dev_pro=1` — activates via `paletta_dev_pro` localStorage key
- **⚠️ Always clear after Pro testing:** `localStorage.removeItem('paletta_dev_pro')` in console before switching back to free flow

## Positioning
"A color system builder for real UI work."
Sell implementation: accessibility checks, shade scales, semantic output, production handoff. Not palette inspiration — production-ready color systems.

## Audit rule — run after every deploy
Test BOTH tiers every time:
1. Free (anonymous) — PRO badges visible, gates fire on Image/Vision/AI
2. Pro (`?dev_pro=1`) — no PRO badges, AI shows Unlimited, all features unlocked, no ProUpgradeModal

## Commit convention
Always end every task with:
```
git add -A && git commit -m "feat/fix: [short description]" && git push
```

## Accessibility — Paletta-specific rules
Inherits all global WCAG 2.1 AA rules from ~/.claude/CLAUDE.md. Additional Paletta-specific rules:

- Brand violet `#6C47FF` on white `#FAFAF8` — contrast 5.9:1 ✅ safe for all text
- Brand violet `#6C47FF` on dark `#1a1a2e` — contrast 3.2:1 ✅ safe for large text/UI only, NOT body text
- PRO badge (violet bg + white text) — always verify contrast when changing badge background
- Bottom sheets / drawers: must trap focus, close on Escape, restore focus to trigger element
- Color swatches: hex code label must always be visible — color alone is never the only identifier
- Lock icons on swatches: must have `aria-label="Locked"` / `aria-label="Unlocked"`
- Generate button: `aria-label="Generate new palette"` always present
- All icon-only toolbar buttons (undo, redo, export, menu): must have `aria-label`
- Vision simulation modes: mode name must be announced via `aria-live="polite"` when changed
- Mobile tap targets: all swatches, buttons, and icons minimum 44×44px


- andresmclavijo@gmail.com — main dev account, manually set is_pro=true in Supabase Table Editor

## Pro user types (all set manually in Supabase)
- `dev` — andresmclavijo@gmail.com, developer access
- `beta` — friends/feedback users comped Pro manually, no Stripe payment
- `stripe` — real paying customers via Stripe webhook
- Note: pro_source column being added in M20 to distinguish these in the DB

## Anthropic API
- Model: claude-haiku (20x cheaper than Sonnet)
- Spend limit: $10/mo, auto-recharge at $5 → tops up to $15
- Key stored in Vercel env vars as `ANTHROPIC_API_KEY`

---

## Design System Agent (auto-loaded)

Read `agents/01-design-system-architect.md` before writing any UI code. All visual decisions must follow the token spec defined there.

Key rules (quick reference):
- NEVER hardcode hex values — use semantic tokens (bg-primary, not bg-[#6C47FF])
- Button heights: 48px (dock) / 36px (action bar + modal) / 32px (bottom bar + close)
- Button radius: ALWAYS 8px (rounded-button) — not 12px, not 6px
- Modal card radius: ALWAYS 16px (rounded-card)
- Gaps between sibling buttons: ALWAYS 6px (gap-1.5)
- Destructive actions (delete): RED (bg-destructive), not violet
- Every button must have aria-label

For full spec, token values, and review checklist: see `agents/01-design-system-architect.md`
For project context and architecture: see `agents/MANIFESTO.md`

## Frontend Expert (auto-loaded)

Read `agents/05-frontend-expert.md` before writing any React components, hooks, or state logic. All component architecture decisions must follow the patterns defined there.

Key rules (quick reference):
- Components > 300 lines → decompose
- No prop drilling > 2 levels → read from Zustand directly
- No inline styles for layout/colors → use Tailwind tokens
- No hand-coded modals/buttons/inputs → use shadcn components
- Use cn() for conditional classes, never string concatenation
- useMemo on expensive calculations (contrast ratios)
- Conditional rendering (not display:none) for inactive views

## Brand & UX Expert (auto-loaded for copy and brand changes)

Read `agents/07-brand-ux-expert.md` before writing any user-facing text or making brand-related decisions. This includes button labels, toast messages, error messages, empty states, tooltips, aria-labels, Pro conversion copy, brand naming, and visual identity standards.

Key rules:
- Strong verbs only (Save, Generate, Export — never Submit, OK, Continue)
- Toasts: max 3 words + icon
- Errors: [what happened] + [what to do], never technical
- Pro gates: lead with what they GET, not what's missing
- Brand name: "Paletta" (capital P, double t) — "paletta" lowercase in wordmarks only
- CTA copy: "Go Pro" or "Upgrade to Pro" — never "Buy", "Purchase", "Subscribe"

## Micro-interactions Designer (auto-loaded for interactive elements)

Read `agents/08-microinteractions-designer.md` before adding any interactive elements, animations, or transitions.

Key rules:
- Every button needs: hover state (150ms) + active:scale-[0.98] + focus-visible:ring-2
- Every modal needs: enter animation (scale 0.96→1 + fade, 200ms ease-out)
- Every card needs: hover:shadow-md (200ms)
- Always respect prefers-reduced-motion
- Paletta is calm — no bounce, no elastic, no overshoot

## Auto-Trigger Reference (which agents activate when)

| Trigger | Agents loaded |
|---------|---------------|
| Writing UI components | Design System Architect + Frontend Expert |
| Writing user-facing text | Brand & UX Expert |
| Adding interactive elements | Micro-interactions Designer |
| Touching API routes, Supabase, Stripe | Backend Expert |
| After deploy (audit) | QA Specialist |
| End of session | Documentation Specialist (roadmap update) |
| Prioritization decisions | Product Strategist |
| Any code change | Design System Architect (token compliance) |

---

## End-of-Session Protocol
Before the final commit of any session, read `agents/04-documentation-specialist.md` and update `paletta-roadmap.jsx` following its end-of-session checklist (mark ✅ items, update session log, refresh compliance section, update timestamp).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andresmclavijo-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
