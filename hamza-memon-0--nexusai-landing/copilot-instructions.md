## nexusai-landing

> Notes for AI coding assistants working in this repo. Keep it short and load-bearing — update as the project evolves.

# CLAUDE.md

Notes for AI coding assistants working in this repo. Keep it short and load-bearing — update as the project evolves.

## What this is

A static, single-page landing page for a fictional AI assistant product called **NexusAI**. The page doubles as a portfolio piece for the author (Hamza Memon) to attract clients — visitors should immediately recognise it as a polished, custom-built landing page, not a template.

- **Stack:** vanilla HTML, CSS, JavaScript. No bundler, no framework, no build step.
- **Deploy target:** Netlify (drag-drop, GitHub integration, or `netlify` CLI all work).
- **Repo:** https://github.com/HAMZA-MEMON-0/NexusAI-Landing

## Local dev

```bash
npm start                # python -m http.server 9123
# then open http://localhost:9123/index.html
```

## Tests (Playwright)

```bash
npm test                 # full suite, all 3 projects
npm run test:ui          # debug visually
npm run test:headed
```

The suite is in `tests/landing.spec.js` and runs three projects: `chromium-desktop` (1440), `chromium-tablet` (820), `chromium-mobile` (390). Webserver is auto-started by `playwright.config.js`.

## File map

| File | Purpose |
|---|---|
| `index.html` | All markup. Sections in order: header, hero, trust marquee, awards, stats, bento features, integrations, demo (animated dashboard), how-it-works, showcase, testimonials, pricing, faq, contact form (CTA), designer credit, footer. |
| `styles.css` | Single design system. Sections marked with banner comments. Tokens at the top under `:root`. |
| `script.js` | One IIFE that wires up: preloader, custom cursor, scroll progress + sticky header, mobile nav, IntersectionObserver reveals, animated counters, magnetic buttons, tilt cards, pricing toggle, FAQ accordion, parallax floats, smooth scroll, demo player, contact form. |
| `tests/landing.spec.js` | 70+ Playwright tests covering every interactive feature. |
| `netlify.toml` | Static deploy config + cache headers + SPA fallback redirect. |

## Conventions

- **Don't add a build step.** Keep it `index.html` + `styles.css` + `script.js` so Netlify can publish the folder verbatim.
- **Don't add JS frameworks** (no React, Vue, Svelte). The whole point of the portfolio is "look what's possible with vanilla."
- **Use BEM-ish class naming** (`block__element--modifier`) consistent with what's already there.
- **Always preserve the real contact info.** Hamza's email (`hamza.memon262830@gmail.com`) and GitHub (`https://github.com/HAMZA-MEMON-0/`) appear in the contact form mailto fallback, footer credit, and designer card. Do not replace with placeholders.
- **Custom cursor** is hidden by default and only `is-active` after the first `mousemove` — don't break this; preview was annoying when the dot sat in the corner before any movement.
- **The demo player** plays a self-contained CSS+JS-animated dashboard (KPIs counting, chart bars growing, chat typing, tasks ticking off) on click. It is not a video — don't add a `<video>` tag or YouTube embed without a real source.

## Contact form (Netlify Forms)

- `<form data-netlify="true" netlify-honeypot="bot-field">` plus a hidden `form-name` field is what Netlify needs to detect the form at deploy time.
- After deploy, submissions appear in **Netlify dashboard → Forms → contact**. Email notifications must be added manually in **Forms → Notifications → Add email notification → `hamza.memon262830@gmail.com`**.
- The JS submit handler in `script.js` POSTs to `/` with `application/x-www-form-urlencoded` body, then shows the success card. Locally (without a real Netlify endpoint) the submission silently fails inside a `try/catch` and the success card still displays so the demo works.

## Deploy workflows

```bash
# Option A — drag & drop the folder onto https://app.netlify.com/drop
# Option B — push to main; if the GitHub repo is linked to Netlify, it auto-deploys
# Option C — Netlify CLI:
netlify deploy --prod --dir=.
```

## Avoiding common gotchas

- **Don't include `Co-Authored-By: Claude` or any AI attribution** in commits, PRs, or files. The author wants this to read as his own work — see the persistent memory feedback note.
- **Hidden form/success cards** rely on `[hidden]` attribute. CSS uses `.contact-form { display: flex }` which would normally beat `[hidden]` — there's an explicit `.contact-form[hidden] { display: none !important }` rule. Don't remove it.
- **Mobile project** in Playwright config used to set `isMobile: true` and that caused 30s teardown hangs in headless. We dropped to plain `viewport: 390×844` + `hasTouch: true` — keep it that way unless you've confirmed the hang is fixed upstream.

---
> Source: [HAMZA-MEMON-0/NexusAI-Landing](https://github.com/HAMZA-MEMON-0/NexusAI-Landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
