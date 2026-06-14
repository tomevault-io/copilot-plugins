## jiaanliux-github-io

> Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## Project: jiaan.me (personal portfolio)

Personal portfolio for Jia An Liu (UNU Macau; applied AI researcher + systems engineer + digital-governance bridge). Positioning is "three-in-one full talent". English only.

**Stack:** Astro 5 + Tailwind v4 (`@tailwindcss/vite`) + astro-icon (Phosphor) + self-hosted Fontsource fonts (Newsreader display serif, Geist body, JetBrains Mono). Static output. No React; small vanilla-JS islands only (scroll reveal, lightbox, nav).

**Content model:** structured data in `src/data/*.ts` (profile, projects, publications, cv). Images in `src/assets/` rendered through `astro:assets` for WebP/AVIF + responsive `srcset`.

**Design is governed by the `design-taste-frontend` (taste) skill** — Editorial direction; restrained single-accent palette (UN ink-blue `#234E7D` + cool slate neutrals, NO rainbow); one corner-radius scale; hairline borders over heavy shadow; asymmetric layout (no equal-column grid as the primary device); Phosphor icons only (never emoji as icons); **no em-dashes in user-facing copy**; motivated, reduced-motion-safe motion. Run the skill's pre-flight check before shipping.

**Contact policy (public site):** show LinkedIn + UNU profile prominently; GitHub + email are subtle footer links. **Never publish a phone number.** Do not copy any job-application materials (cover letters, tailored resumes, contracts, HR emails) into this repo.

**Modular drop-in pages:** self-contained one-off pages live at `public/p/<slug>/index.html` and are served verbatim at `https://jiaan.me/p/<slug>/`. They are unlisted (not in nav, excluded from sitemap, `noindex`) and intentionally independent of the main design system. To publish one: copy `public/p/_TEMPLATE`, rename, edit, rebuild, redeploy. See `public/p/README.md`.

**Deploy (this machine IS the VPS `Jiaan-VM-US-LA`):** follow the `self-hosted-deploy` skill. The site is a container (`nginx:alpine` serving the Astro `dist/`) on the external `docker_shared_network`, exposed (not published). The edge is `xray` (owns :443, SNI cert) then a unix socket then `naiveproxy` (Caddy, reverse_proxy by Host). To serve `jiaan.me` (+ `www.jiaan.me` once its DNS points here): add a route + `tls.automate` entries in `server-configs/.../naiveproxy/config.json`, let naiveproxy issue the certs, then add `certificates[]` entries in `.../xray/config.json` and restart (naiveproxy first, then xray). The apex `jiaan.me` already resolves to this server.

---
> Source: [jiaanliux/jiaanliux.github.io](https://github.com/jiaanliux/jiaanliux.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
