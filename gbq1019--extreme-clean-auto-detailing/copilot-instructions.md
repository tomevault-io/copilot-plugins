## extreme-clean-auto-detailing

> Marketing site for **Extreme Clean Auto Detailing** — a mobile detailing business owned by **Blake**, serving Macomb and Oakland counties in Michigan. Premium positioning ("#1 in the area"), phone-call-first conversion, no pricing shown on site.

# CLAUDE.md

Marketing site for **Extreme Clean Auto Detailing** — a mobile detailing business owned by **Blake**, serving Macomb and Oakland counties in Michigan. Premium positioning ("#1 in the area"), phone-call-first conversion, no pricing shown on site.

Stack: Vite + React 18 + TypeScript + Tailwind 3 + `lucide-react` icons. Deploys to Vercel. Current state is a single-page scroll scaffolded by Bolt; actively being rebuilt into a multi-page site.

---

## Known failure modes — do not repeat

**1. Color extraction from reference images.** Claude cannot reliably extract exact hex values from PNG/JPG screenshots. Approximated colors are close but wrong, which breaks brand consistency. When working from visual references: use reference images for *composition and layout only*, never for color picking. Always use the defined brand hex values. If a color not in the palette is genuinely needed, propose the exact hex to Blake/Gabriel for confirmation before using it.

**2. Service area map — use static image + SVG overlay only.** The service area map is a static PNG screenshot saved at `public/projects/service-areas-reference-dark-map-overlays-2026-04-15.png`, displayed at fixed dimensions in the ServiceAreas component, with a freehand SVG `<path>` overlay tracing Blake's coverage area. The outline stroke uses brand.yellow (`#FFC900`) via the `stroke-brand-yellow` Tailwind utility; the interior fill uses brand.yellow at 15% opacity (`fill-brand-yellow` with `fill-opacity="0.15"`) to subtly highlight the served region. Stroke width 5px, rounded linejoins and linecaps. Do NOT use Google Maps JS API, Mapbox, Leaflet, OpenStreetMap tile libraries, or any other map service at runtime. Do NOT revert to a custom hand-drawn SVG of Michigan counties. The map image is a one-time export — if Blake's service area changes, replace the PNG and update the SVG path coordinates. A reference image at `public/projects/service-areas-reference-hand-drawn-boundary-2026-04-22.png` shows the intended outline shape; it is used for deriving path coordinates only and is NOT loaded at runtime.

**3. Copy changes require confirmation.** All existing site copy was pulled from Blake's live Wix site (extremecleanauto.com) and represents his approved voice. Do not rewrite, "improve," or SEO-optimize copy unprompted. See Copy Rules below.

**4. Uninvited design changes.** Don't rearrange sections, swap animations, or restructure components unprompted. Propose-then-implement. See Design Rules below.

---

## Architecture

**Target structure (in progress, not yet built):**

- `/` — Home (teaser sections: About, Service Areas, Reviews, Services, Ceramic Coatings, Contact)
- `/detailing` — Full mobile detailing services page (3 services)
- `/ceramic-coatings` — Full ceramic coatings page (3 packages — content TBD from Blake)
- `/contact` — Inquiry form page (submits to Fieldd CRM)

Current codebase is single-page (`App.tsx` renders all sections sequentially). Routing migration to `react-router-dom` is pending. When splitting sections into pages, ask before refactoring existing components — some may need to serve as both homepage teasers and full-page versions.

**Lead capture flow:** Website inquiry form → `/api/submit-lead.ts` Vercel serverless function → Fieldd CRM webhook → Fieldd's automations handle SMS/email follow-up → Blake manually books appointments inside Fieldd. The site is a lead capture tool, not a booking tool. Blake does not want clients booking their own appointments.

**Supabase is installed but unused.** The `@supabase/supabase-js` dependency is a leftover from the Bolt scaffold. It can be removed — the project uses Fieldd for CRM, not Supabase.

---

## Environment variables

All server-side secrets (no `VITE_` prefix) live in Vercel env settings and locally in `.env.local` (gitignored). Never commit secrets.

- `FIELDD_WEBHOOK_URL` — endpoint for posting leads to Fieldd (server-side). **TBD — Blake's Fieldd integration path is not yet determined.** Build the inquiry form with a clearly-marked `submitLead()` TODO that currently logs to console or shows a success toast; replace with real Fieldd integration once CRM access is available.
- `GOOGLE_PLACES_API_KEY` — for pulling live Google reviews via `/api/google-reviews.ts` (server-side). Not yet obtained.
No other env vars currently planned. Spam protection (Turnstile/hCaptcha) not implemented — can add later if spam becomes a problem. Email notifications to Blake on form submit are handled by Fieldd's automations, not by the site.

---

## Brand system

Palette:

- **Blue:** `#1E90FF` — primary accent, CTAs, highlights (existing hex literal convention)
- **Blue dark:** `#0055cc` — gradient partner to blue (existing hex literal convention)
- **Dark:** `#080808` — near-black background (existing hex literal convention)
- **Yellow:** `#FFC900` — new accent, used strategically (NOT everywhere — rare, high-attention). Defined as `brand.yellow` in `tailwind.config.js`.

**Color usage rules:**

- For blue/dark: existing hex literals (`bg-[#1E90FF]`, etc.) remain the convention. Do not migrate to tokens.
- For yellow: **always use `bg-brand-yellow` / `text-brand-yellow` / `border-brand-yellow`** — never the raw hex.
- Never introduce a fifth color without approval. Never infer colors from screenshots.

Typography:

- **Barlow Condensed** — display headings (via `.font-display` class), uppercase, dramatic
- **Inter** — body text (default)
- Both loaded from Google Fonts in `src/index.css`

Custom CSS classes defined in `src/index.css` — reuse these, don't reinvent:

- `.glass` — frosted glass card background
- `.btn-primary` — primary CTA button (blue gradient + shine-on-hover)
- `.text-gradient` — blue-to-white animated gradient text
- `.card-hover` — elevation + blue glow on hover
- `.nav-link` — underline-on-hover nav links
- `.section-divider` — thin blue horizontal divider
- `.animate-fadeInUp`, `.animate-float`, `.animate-fadeIn` — keyframe animations

**Section animation pattern** — every major section uses IntersectionObserver with staggered `reveal` class children:

```tsx
const sectionRef = useRef<HTMLDivElement>(null);
useEffect(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const children = entry.target.querySelectorAll('.reveal');
          children.forEach((el, i) => {
            setTimeout(() => {
              (el as HTMLElement).style.opacity = '1';
              (el as HTMLElement).style.transform = 'translateY(0)';
            }, i * 120);
          });
        }
      });
    },
    { threshold: 0.15 }
  );
  if (sectionRef.current) observer.observe(sectionRef.current);
  return () => observer.disconnect();
}, []);
```

New sections must follow this pattern. Don't replace it with Framer Motion, CSS-only animations, or a custom hook — consistency matters more than novelty here.

---

## Copy rules

**Existing copy is Blake-approved.** Pulled directly from extremecleanauto.com (his live Wix site). Do not rewrite unprompted.

**SEO/GEO optimization is an ongoing goal.** Claude may *propose* SEO-optimized rewrites (local keywords like "Macomb County," "Shelby Township," "mobile detailing near me," "ceramic coating Michigan"; natural-language headings; location mentions in meta tags and structured data), but must surface the proposal and wait for Gabriel's approval before changing visible text.

**New copy written from scratch** (service descriptions, form labels, alt text, meta tags, schema.org JSON-LD, button labels) should be SEO/GEO-optimized by default — natural keyword integration, not keyword stuffing.

**Never invent content.** If ceramic coating package details, certification info, exact review quotes, or similar content is missing, scaffold a placeholder component and leave content as `TODO: awaiting content from Blake` — do not fabricate plausible-sounding substitutes.

**Never invent pricing.** Blake does not want pricing shown anywhere on the site. CTAs use language like "Get a Quote" / "Call for Pricing" / "Request a Quote" — never "Book Now from $X."

---

## Design rules

**Mobile-first, desktop-polished.** Design decisions optimize for mobile (thumb-reachable CTAs, readable without zoom, single-column flow, tap-to-call prominence). Desktop breakpoints (`md:`, `lg:`) must be intentional and look equally premium — not a shrunk-or-stretched afterthought. Test both when making layout changes.

**Propose before implementing design changes.** Claude may suggest layout improvements, animation tweaks, yellow accent placements, component refactors, or structural changes — but must surface the proposal and wait for approval before making the change. Execute requested design changes directly; don't make uninvited ones.

**Conversion priority:**

1. **Tap-to-call** (`tel:586-481-2121`) — primary goal on every page. Phone number visible in navbar on desktop, thumb-reachable on mobile. Every "Book Now" CTA should drive either a call or the inquiry form — never bury the phone number.
2. **Inquiry form** — secondary path for people who don't want to call. Feeds Fieldd pipeline.

**Premium positioning.** Blake is the #1 detailer in Oakland/Macomb. Tone stays confident — "showroom quality," "certified specialists," "trusted by hundreds of clients." Visual design reinforces authority: dark luxury aesthetic, electric blue accents, yellow as a premium-warning-stripe energy (automotive tool/equipment vibe), not cheap or playful. Trust signals (reviews, years in business, service coverage) get prominent placement because #1 has to be *demonstrated*, not just claimed.

---

## Content & trust signals inventory

What Blake has (use these):

- **Real customer reviews** across Google, Facebook, Yelp — will pull live via Google Places API into a `/api/google-reviews.ts` serverless function, cached to stay under quota. Fallback if API is delayed: curated static reviews in a designed component.
- **Before/after photos** — abundant on his Instagram, Facebook, TikTok. Blake will provide specific photos as content is scoped.
- **Video testimonial / car montage** — Blake is producing a custom montage for the site. Scaffold the section with a placeholder; swap in the video file when delivered.
- **Years in business** — founded 2022, already on site. Keep current.
- **Number of cars detailed / clients served** — he has real numbers. Confirm current count with Blake before displaying.
- **Real social URLs:**
  - Instagram: `https://www.instagram.com/extremeclean.autodetailing/`
  - Facebook: `https://www.facebook.com/profile.php?id=100084377522579`
  - TikTok: `https://www.tiktok.com/@extreme_clean_detailing` — **not on current Bolt site, needs to be added**
- **Phone:** `586-481-2121`
- **Address:** 14214 Bournemuth Drive, Shelby Township, MI (use for local business schema; confirm with Blake before displaying publicly)

TBD — do not fabricate:

- **Certifications** (ceramic coating brand certs, IDA membership) — unknown whether Blake has any. If he provides, add to trust signals. Don't invent.
- **Awards / "best of" recognitions** — unknown. Same rule.
- **Insurance / licensing** — unknown. Same rule.
- **Ceramic coating packages (3 total)** — content coming from Blake. Scaffold the page structure; placeholder the package names, tiers, and inclusions until Blake delivers real content.

---

## Workflow

**Git:**

- Never commit directly to `main`. Always create a feature branch.
- Branch naming: `feat/inquiry-form`, `fix/navbar-mobile`, `refactor/tailwind-yellow-token`, etc.
- If Gabriel forgets to branch, flag it before committing.

**Dev server:**

- Gabriel keeps `npm run dev` running in a background terminal throughout sessions. Assume `localhost:5173` is live and hot-reloading. Don't start/stop the dev server. To verify visual changes, ask Gabriel to refresh or (if Playwright/Chrome DevTools MCP is connected) screenshot directly.

**Before declaring a task complete:**

- Run `npm run lint` — fix any errors or surface them to Gabriel.
- Run `npm run typecheck` — fix any errors or surface them to Gabriel.
- Don't say "done" until both pass.

**Deployment:**

- Target is **Vercel**. Static Vite build + serverless functions in `/api/*.ts`.
- Vercel Analytics enabled for traffic tracking. Custom event tracking via `@vercel/analytics` on key CTAs (Call, Text, Inquiry Submit) so Blake can see which buttons drive action.
- Custom domain: TBD (currently on Wix; will cut over once the new site is ready).

**Dependency discipline:**

- `.bolt/prompt` says: Tailwind for styling, `lucide-react` for icons, don't add UI theme libraries (no shadcn, no Material UI, no Chakra) unless absolutely necessary. Honor this. If a library genuinely solves a problem that can't be solved with what's installed, propose it — don't just install it.

---

## Open architectural questions (resolve when the work forces the decision)

- **Services and CeramicCoatings components** are currently written as homepage sections. When `/detailing` and `/ceramic-coatings` become full pages, decide: (a) same component with a `variant` prop for teaser vs. full, (b) split into `ServicesTeaser.tsx` + `ServicesPage.tsx`, or (c) build full pages first and refactor homepage teasers to match. Ask Gabriel before refactoring.
- **Fieldd integration mechanism** — webhook vs. Zapier bridge vs. direct API — unknown until Blake's Fieldd account is accessible. Build the form interface first with a TODO handler; wire up real integration once path is confirmed.

---

## At a glance

- Client: **Blake**, owner of Extreme Clean Auto Detailing
- Location: Shelby Township, MI — serves Macomb and Oakland counties
- Phone (primary CTA): `586-481-2121`
- Live reference site: https://extremecleanauto.com (Wix — being replaced)
- Stack: Vite + React + TS + Tailwind + Lucide, deploys to Vercel
- CRM: Fieldd (lead capture only — Blake books appointments manually inside Fieldd)
- Conversion priority: phone calls > inquiry form
- Don't: rewrite copy without asking, invent content, extract colors from screenshots, commit to main, book appointments on Blake's behalf, show pricing anywhere

---
> Source: [gbq1019/extreme-clean-auto-detailing](https://github.com/gbq1019/extreme-clean-auto-detailing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
