## ui-lab

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# ui lab: how to work here

ui lab (lab.xevrion.dev) is Yash's (xevrion) personal lab of small interaction experiments: things he made because he liked how they felt. It is **not a UI library** and must never be pitched as one. Every piece is a live demo with its source on GitHub. The bar is high: each component should be something a designer would screenshot and share, while staying simple, fast and accessible.

Stack: Next.js 16 App Router (read `node_modules/next/dist/docs/` before using a Next API), React 19, Tailwind CSS 4, Motion (`motion/react`), Bun. Deployed on Vercel; a push to `main` is a production deploy. `simple-icons` is available for brand logos (tree-shaken; import named icons only).

## Making a component: the workflow

1. **Understand the brief.** When Yash describes a component, follow his spec closely; ask only if something is genuinely ambiguous. If he gives a reference image, keep the idea but render it in this lab's style (monochrome tokens, restrained, one accent at most).
2. **Scaffold:** `bun run new <slug> <category>`. It creates `src/lab/components/<slug>.tsx` and registers it in three files: `src/lab/registry.ts` (metadata only, never import components there), `src/lab/demos.tsx` (code-split loader for its own page) and `src/lab/previews.ts` (static bundle for the index). Never remove the `// new-component:` markers. Categories: `buttons`, `inputs`, `navigation`, `feedback`, `data`, `cards`, `objects`, `playground`, `text` (text effects are ranked last on the index).
3. **Flag it new:** add `isNew: true,` under its `slug:` line in the registry. New entries lead the index and get a quiet "New" tag (a static pill with a red dot; the old self-drawing circle was too loud in a grid), and the newest one is linked from the hero's "New" pill. When a new *batch* lands, clear the previous batch's flags; single additions can stay alongside the latest batch.
4. **Build it** (rules below). Export the reusable component by name with sensible props, and `export default function <Name>Demo()` with believable, real content.
5. **Verify visually** (see "Seeing it"): light and dark, mid-animation frames, 375px width, console clean, reduced motion.
6. **Register:** `bun scripts/register.ts '[{"slug":"x","description":"...","keywords":"..."}]'`. Description: one short present-tense line about what it *does* ("Leans toward your cursor before you even reach it."). Keywords: 4 to 8 lowercase search words. Add `"anchor":"top"` if the demo grows downward.
7. **Fit its card:** measure the demo's rendered size at 1440px, then `bun scripts/set-scale.ts '{"slug":[w,h,max?]}'` (fits into 316x200; `max` like 1.6 lets tiny controls grow). Very tall demos can use `previewCrop: true`. Then look at the card on the index, at rest and hovered.
8. **Hover show:** add a short hover demonstration for the index card (see "Card previews"). If the component has nothing meaningful to show, add nothing.
9. `bunx eslint <file>`, `bunx tsc --noEmit`, `bun run build`. Commit only when asked (single-line message, no Co-Authored-By trailer).

Helper scripts live in `scripts/` (not the scratchpad, which gets wiped): `new-component.ts`, `register.ts`, `set-scale.ts`, `set-description.ts '{"slug":"text"}'`, `fetch-contributions.ts` (runs before every build).

## Design rules (from Yash's feedback; each one was a real complaint)

- **Simple, unique, not generic.** One signature idea per component, drawn from what the user is doing at that moment, executed perfectly. Not a shadcn/Radix/Aceternity clone, and not over-engineered: small footprint, calm at rest, alive when touched. If a component looks complex or crowded, it's wrong.
- **Real-object quality when it's an object.** Physical metaphors (dials, receipts, lamps, tape measures) must look like beautifully made objects with believable materials, not clip art or grey placeholders.
- **The thing you click never moves** out from under the cursor. Content grows downward from a fixed top (then set `anchor: "top"`); never auto-collapse siblings; never scroll on toggle.
- **Nothing appears out of thin air.** New controls unfold from the element that caused them (e.g. call controls slide out from behind the End button); the pressed button morphs into its next role rather than being replaced.
- **No muddy crossfades between hues.** Half-transparent red over green reads brown; over grey reads pink. Swap colours with a clip-path wipe or keep them on separate layers.
- **Motion is calm, never frantic.** Drifting/ambient motion must be slow (a lap takes 10s or more). Continuous angles must be integrated per frame (`angle += speed * dt`), never computed as `time * speed`, or changing speed flings everything.
- **Centred and balanced.** Demos sit centred in their stage and in their index card, in every state. Avoid tiny objects floating in big empty stages and dead space.
- **Newly revealed content blurs in** (opacity plus blur 4px to 0, a slight y), never pops. Text that differs between morph states isn't shared across the morph.
- **Text inside a scaled/layout-animated element** uses `layout="position"` or it stretches.
- **No native scrollbars inside demos** that mock a page (`[scrollbar-width:none] [&::-webkit-scrollbar]:hidden`); the component itself shows position.
- **Hand-drawn accents** use the red-pen `--marker` token and the Caveat font (`next/font/google`, `preload: false`). Handwriting draws as one continuous stroke (arrowheads included) so nothing pops in separately; it animates once when first seen, never on hover.
- **Sign-offs stay small.** Signatures, badges and flourishes are footer-sized and quick.
- **Arrows are drawn, not typed:** use the SVG `Arrow` component (`src/components/arrow.tsx`), never ↗ ← → glyphs.
- **Keyboard shortcuts for occasional actions animate** exactly like the mouse path. Only constant actions skip animation.
- **Placeholders hide instantly** when typing starts.
- **Drags** keep release velocity; `dragMomentum={false}` if you animate the value yourself; never block the next interaction on an exit; never widen the page.

## Craft rules

- **Colour:** tokens only (`bg-background`, `bg-surface`, `text-foreground`, `text-muted`, `border-border`, `bg-danger`, `text-marker`, `shadow-raised`; opacity modifiers are fine). Raw colours only when they ARE data or a physical material, with a comment. Must look right in both themes; the `dark:` variant follows the toggle.
- **Motion:** name exact transition properties (never `transition-all`); custom easing (`cubic-bezier(0.23,1,0.32,1)` out, `(0.77,0,0.175,1)` in-out, `(0.32,0.72,0,1)` drawer); UI motion under ~300ms unless a comment justifies it; exits faster and softer than entrances; springs that keep velocity; `active:scale-[0.96]` on pressables; never animate from `scale(0)` (icon swaps go 0.25 to 1 with opacity and a 4px blur); prefer transform and opacity.
- **Reduced motion:** always import `useReducedMotion` from `@/lib/use-reduced-motion`, never from `motion/react` (Motion's version breaks hydration for reduced-motion users).
- **Focus rings (Tailwind 4 trap):** `outline-hidden` and `outline-none` both set the outline style to none, so always write `outline-hidden focus-visible:outline-2 focus-visible:outline-solid focus-visible:outline-foreground` (same for `has-`, `peer-`, `focus-within:` variants).
- **Accessibility:** real buttons and inputs, a full keyboard path, correct ARIA, polite live regions for async changes.
- **Performance** (the site is judged on it): no per-frame React state (write styles or motion values directly); rAF loops sleep at rest and offscreen (IntersectionObserver); clean up every timer, listener, observer and rAF.
- **Hydration:** no `Date.now()`, `Math.random()` or locale output during server render (render a stable placeholder, or `useSyncExternalStore` with a server snapshot); round trig-derived SVG coordinates.
- **Sizing:** build at full-page size (composites roughly 360 to 560px wide, body text 14 to 15px, nothing under 12px, controls 36 to 44px), `max-w-full` so nothing widens a 375px screen. Never shrink a component to fit a card; the card scales it.
- **Positioned children stay inside:** an absolutely positioned or `sr-only` element inside a demo's own scroll box escapes to the page unless an ancestor is positioned. `sr-only` on a `<table>` doesn't collapse; wrap it in a div.
- **Comments:** sparse, why-only, every magic number explained. No em dashes anywhere, in code or UI text.
- **Components stand alone:** never import from another component file.

## Card previews

- Index cards are `LabCard` (`src/components/lab-card.tsx`) with a stretched title link, so demos may contain links. Previews render inside an `inert`, CSS-scaled box.
- **Double-scale trap:** in a card, `getBoundingClientRect`/`getClientRects` are post-scale. Divide by `rect.width / el.offsetWidth` (or use `offset*`) before drawing inside the component.
- **Hover-to-play:** demos read `usePreviewPlay()` from `@/lab/preview-play`: `null` on the component's own page, `false` in an idle card, `true` while the card is hovered or focused, or, on touch devices (no hover), while it's at least 60% on screen, so phones see every show as they scroll. Design shows so they read well without a cursor. On `true`, run a short, natural show through the component's real state (a person briefly using it), looping calmly; on `false`, cancel timers and ease back to rest. Idle cards must cost nothing: no JS timers, rAF or observers (pause loops when `usePreviewPlay() === false`; cheap CSS ambience is fine). No global side effects from a show: never write the clipboard, change the theme, vibrate, steal focus or fire full-screen effects.
- Left-aligned fixed-width demo boxes look off-centre in a scaled card; centre them when `usePreviewPlay() !== null`.

## Seeing it

Background browser tabs freeze animations, so verify with headless Chrome through `playwright-core` (with the system Chrome): screenshot light and dark, capture frames at 60 to 200ms intervals during interactions, check at 375px width, read console errors and hydration warnings, and test with `reducedMotion: "reduce"`. Measure geometry by script when needed (element positions, per-frame movement, page `scrollHeight`). Look at the actual PNGs and judge them honestly as a demanding design lead would, then iterate.

## Site framing

- Copy is personal: "Things I made because I liked how they felt". It is a lab to experiment in and enjoy, never a UI library: say "experiments", "things", "the lab", not "components" or "library", in visible copy and labels.
- Phones are a first-class audience (most visitors): check every change at 320 to 430px, inputs are 16px on phones (iOS zooms smaller ones), no keyboard hints on touch, nothing widens the page.
- Never put the component count in SEO, OG images, metadata, JSON-LD or hero copy; it keeps changing and social caches go stale. The live count in the index search UI is fine.
- Each component page gets its own OG image automatically (`src/app/lab/[slug]/opengraph-image.tsx` with `generateStaticParams`, drawn by `src/lib/og.tsx`).

---
> Source: [xevrion/ui-lab](https://github.com/xevrion/ui-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
