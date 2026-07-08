## design-recruitment-draft

> - **Invoke the `frontend-design` skill** before writing any frontend code, every session, no exceptions.

# CLAUDE.md — Frontend Website Rules

## Always Do First
- **Invoke the `frontend-design` skill** before writing any frontend code, every session, no exceptions.

## Reference Images
- If a reference image is provided: match layout, spacing, typography, and color exactly. Swap in placeholder content (images via `https://placehold.co/`, generic copy). Do not improve or add to the design.
- If no reference image: design from scratch with high craft (see guardrails below).
- Screenshot your output, compare against reference, fix mismatches, re-screenshot. Do at least 2 comparison rounds. Stop only when no visible differences remain or user says so.

## Local Server
- **Always serve on localhost** — never screenshot a `file:///` URL.
- Start the dev server: `node serve.mjs` (serves the project root at `http://localhost:3000`)
- `serve.mjs` lives in the project root. Start it in the background before taking any screenshots.
- If the server is already running, do not start a second instance.

## Screenshot Workflow
- Puppeteer is installed locally via the project's `node_modules` (macOS).
- **Always screenshot from localhost.** The simple path is `node screenshot.mjs http://localhost:3000`; for viewport-specific shots (mobile/tablet) or scrolled-section captures, use an inline `node --input-type=module` script that drives Puppeteer directly.
- Screenshots save to `./temporary screenshots/screenshot-N.png` (auto-incremented, never overwritten). Optional label suffix: `screenshot-N-label.png`.
- After screenshotting, read the PNG with the Read tool — Claude can see and analyze the image directly.
- When comparing, be specific: "heading is 32px but reference shows ~24px", "card gap is 16px but should be 24px".
- Check: spacing/padding, font size/weight/line-height, colors (exact hex), alignment, border-radius, shadows, image sizing.

## Stack & Output Defaults
- Plain hand-written HTML, CSS, and JavaScript. No build step, no framework, no Tailwind.
- Shared design tokens live in `tokens.css` (root level). Both the homepage and `about/index.html` link to it. Edit tokens once and the rest of the site follows.
- Per-page styles live in a `<style>` block in the page's `<head>`. Reuse class names across pages where the design is shared (e.g. `.section-tag`, `.fade-up`, `.serif-i`, `.btn-primary`, `.btn-ghost`, `.btn-dark-ghost`).
- Pages: `index.html` (homepage), `about/index.html` (About), plus `privacy.html` / `cookies.html` / `terms.html`. New top-level pages live in their own subfolder (`/<name>/index.html`) so URLs stay clean.
- Fonts in `/Fonts/` (Canela woff/woff2). Brand assets in `/Brand_assets/`. Reference both with absolute paths so they resolve from any subfolder.
- Mobile-first responsive. Breakpoints used elsewhere on the site: 1024px (tablet), 768px (mobile), 640px (small mobile). Stick to those.
- Placeholder images: `https://placehold.co/WIDTHxHEIGHT` only when no real asset is available.

## Brand Assets
- Always check the `brand_assets/` folder before designing. It may contain logos, color guides, style guides, or images.
- If assets exist there, use them. Do not use placeholders where real assets are available.
- If a logo is present, use it. If a color palette is defined, use those exact values — do not invent brand colors.

## Anti-Generic Guardrails
- **Colors:** Use the brand palette declared in `tokens.css` (`--color-midnight`, `--color-navy-deep`, `--color-blue`, `--color-gold`, `--color-cream`). Derive variants from those — don't invent new brand colours.
- **Shadows:** Use layered, dark-tinted shadows with low opacity (e.g. `0 2px 4px rgba(7,7,26,0.04), 0 12px 28px rgba(7,7,26,0.08)`). No flat single-layer shadows.
- **Typography:** Canela Deck (serif, italics enabled) for display/headings via `--font-display`; Montserrat for body via `--font-body`. Tight letter-spacing (`-0.02em`) on large headings, line-height ~1.7 on body.
- **Gradients:** Layer multiple gradients (linear + radial) for depth. The site uses a fixed grain overlay; reuse it on new pages where appropriate.
- **Animations:** Only animate `transform` and `opacity`. Never `transition-all`. Use the eased curves declared in `tokens.css` (`--ease-out-expo`, `cubic-bezier(0.25,0.46,0.45,0.94)`). Sequence reveals with `transition-delay` rather than firing everything at once.
- **Interactive states:** Every clickable element needs hover, focus-visible, and active states. No exceptions.
- **Images:** Pair full-bleed photos with a vignette gradient overlay so foreground text is always legible. Use the existing `data-bg="light"` attribute on cream sections so the floating nav pill auto-inverts.
- **Spacing:** Use the spacing scale and section padding tokens declared in `tokens.css` (`--space-*`, `--pad-section-y`, `--pad-section-x`). No random pixel values.
- **Depth:** Surfaces follow a layering system (`--bg-base` → `--bg-surface` → `--bg-elevated`). Don't park everything on the same z-plane.

## Hard Rules
- Do not add sections, features, or content not in the reference.
- Do not "improve" a reference design — match it.
- Do not stop after one screenshot pass.
- Do not use `transition-all`.
- Do not invent brand colours — use the palette in `tokens.css`.
- Do not introduce a build step (Vite, Tailwind, etc.) on the production site without confirmation. The codebase is intentionally plain HTML/CSS/JS.

## Project Workflow Instructions

This site is deployed via Netlify on a credit-based plan. Every push to `main` triggers a production deploy that costs 15 credits out of a 300/month allotment. The instructions below exist to conserve credits.

### Branch policy
- The default working branch is `dev`. All commits and pushes go to `dev` unless I explicitly say otherwise.
- Never push to `main` without explicit confirmation in the chat. Treat phrases like "commit", "save", "push my changes", or "update the site" as meaning `dev`.
- Only treat phrases like "publish", "go live", "ship it", or "merge to main" as a request for a production deploy.

### Preview policy
- **Default: local only.** Run `node serve.mjs` and view at `http://localhost:3000`. For mobile/tablet checks, use a Puppeteer script that drives localhost at the viewport you care about (e.g. 390×844 for mobile).
- **Manual Netlify preview is opt-in.** Only run `npx netlify deploy --dir=.` (without `--prod`) when I explicitly ask for a shareable preview URL — for example, sharing the work-in-progress with someone or testing on a real device. Each preview deploy consumes ~1–3 Netlify credits, so never trigger one as part of the default workflow.
- Never auto-deploy after a commit. If you think a preview is needed, ask first.

### Going live (production deploy)
When I confirm I want to publish:
1. Confirm with me one more time before merging, stating: "This will trigger a production deploy on Netlify (15 credits). Confirm?"
2. Once confirmed, merge `dev` into `main` and push `main` to GitHub.
3. Switch back to `dev` afterwards so the next session starts on the working branch.

### Image handling
- Before committing any new image, flag it if it's over 500KB and suggest compressing it with Squoosh or TinyPNG first.
- Prefer modern formats (WebP) where possible.

### Default behavior
- After completing a change, push to `dev` (not `main`) by default.
- If I ask for a production deploy without realising the cost, remind me of the 15-credit cost and confirm.

---
> Source: [armani-44/Design-Recruitment-Draft](https://github.com/armani-44/Design-Recruitment-Draft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
