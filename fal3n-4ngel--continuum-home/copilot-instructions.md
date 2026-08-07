## continuum-home

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# UI Dialogs & Alerts
Always use custom, styled React/HTML modal popups and themed dialogs instead of browser native alerts, prompts, or confirmations (`alert()`, `prompt()`, `confirm()`). Make sure all modal backdrops cover the entire screen (including sidebars and navigation) by rendering them using React Portals (`createPortal`) targeting `document.body` to break out of parent stacking contexts and CSS animations.

---
> Source: [fal3n-4ngel/Continuum-Home](https://github.com/fal3n-4ngel/Continuum-Home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
