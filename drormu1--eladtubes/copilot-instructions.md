## eladtubes

> Marketing one-page website (Hebrew, RTL) for אלעד שמחי — contractor in Israel:

# CLAUDE.md

## Project
Marketing one-page website (Hebrew, RTL) for אלעד שמחי — contractor in Israel:
sewer work, earthworks, plumbing. Based in Petah Tikva / central Israel; serves the whole country. SEO copy intentionally leads with local keywords (see 2026-08-31 note below).

## Goals
- Small, clear, conversion-focused. Contact actions first: phone + WhatsApp.
- Content updates (text/images/videos) must never require touching logic.

## Stack & File Roles
Static site, no build step. Must keep working when opened directly from disk (file://) and on GitHub Pages.

- `index.html` — single page. Sections: hero, #services, #equipment, #clients, #projects, #team, #contact. Loads `data.js` BEFORE `script.js` (order matters).
- `data.js` — ALL editable content: `window.siteData = { contact, galleries }`. Contact details live ONLY here (real phone: 050-886-1398). Content changes go here and nowhere else.
- `script.js` — logic only: gallery rendering + wiring `data-phone-link` / `data-whatsapp-link` / `data-phone-display` elements. Do not put content here.
- `styles.css` — responsive RTL design, CSS variables in `:root`, breakpoints at 980px and 600px.

## Working Rules
- You may freely edit files inside this project folder without asking each time.
- Prefer small, safe, incremental changes.
- Preserve Hebrew RTL support and responsive behavior.
- Media lives in `assets/images/<section>` and `assets/videos/<section>`; keep folder names stable.
- Phone numbers displayed in HTML are static placeholders; `script.js` replaces them from `data.js` at load. Galleries are empty without JS — acceptable.

## History / Decisions
- 2026-08-31: "מספר ישיר" copy box removed from #contact per owner request — do not re-add.
- 2026-08-31: "טלפון ישיר" strip removed from hero (redundant next to the call button; number still shows in topbar + #contact) — do not re-add.
- 2026-08-31: data split out of script.js into data.js (chosen over fetch+JSON to keep file:// working).
- 2026-08-31: Mobile (≤980px): compact one-row header with hamburger menu (.nav-toggle toggles .nav-open on .topbar); ≤600px: floating buttons become a fixed bottom action bar (call/WhatsApp). Do not revert to stacked column topbar.
- Keep copy concise and action-oriented.
- 2026-08-31: Owner clarified service area is nationwide. Visible copy says "שירות בכל הארץ"; title/meta/schema still lead with פתח תקווה/מרכז because local keywords convert better — do not strip them.
- 2026-08-31: Canonical service phrase order everywhere (copy, title, meta, schema): "עבודות ביוב, אינסטלציה ועפר" — sewer, then plumbing, then earthworks. Do not reorder.
- 2026-09-01: Accessibility baseline per ת"י 5568 / WCAG 2.0 AA — skip link, visible :focus-visible styles, prefers-reduced-motion support, scroll-margin under sticky header, aria-label on icon-only WhatsApp topbar button and on gallery videos, accessibility statement page (`accessibility.html`, linked from footer + sitemap). WhatsApp greens darkened to `--whatsapp`/`--whatsapp-strong` (#11813a/#0e7a35) so white text passes AA contrast — do not brighten back to #25d366. No third-party accessibility widget by design; the site itself complies.
- 2026-08-31: SEO baseline added — LocalBusiness (Plumber) JSON-LD injected by script.js from `siteData.business` in data.js, OG tags + local keywords in index.html, robots.txt + sitemap.xml. Placeholder `REPLACE-WITH-YOUR-SITE-URL` and empty `siteUrl` must be filled once the site is deployed.

---
> Source: [drormu1/eladtubes](https://github.com/drormu1/eladtubes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
