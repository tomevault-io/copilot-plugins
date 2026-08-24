## web-kreta

> Landing page for **Kreta** (tecnología + finanzas). Brand is **Kreta**, not "Web-Emprendimiento".

# Kreta Web — AGENTS.md

## Project

Landing page for **Kreta** (tecnología + finanzas). Brand is **Kreta**, not "Web-Emprendimiento".

## Stack

HTML + **Tailwind CSS v3 (compiled)** + vanilla JS. No Bootstrap, no JS frameworks.

- CSS compiles from `assets/css/tailwind-src.css` → `assets/css/tailwind.css`.
- `tailwind.css` **is committed** so Cloudflare Pages serves it buildless.
- `node_modules` is gitignored.
- **Tema "Aurora" oscuro** (dark) en todo el sitio: fondo casi negro, glassmorphism, acentos violeta + ember. Tokens/fonts/fontSize in `tailwind.config.js`.
- Uses arbitrary values (`rounded-[100px]`, `rounded-[12px]`) — do NOT override `full` or circles break.

## Key files

- `index.html` — landing principal (hero, el problema, capacidades, BiFrost, FAQ, contacto).
- `secciones/servicios.html`, `nosotros.html`, `contacto.html` — subpáginas, same stack.

**Alcance del contenido (decisión de negocio, respetar):** el sitio ofrece **solo
BiFrost + servicios financieros** (asesoría, costos/precios, flujo de caja y
presupuesto). Los servicios de desarrollo (landing pages, e-commerce,
automatizaciones, chatbots) se quitaron a propósito para no confundir al cliente
— no reintroducirlos. Tampoco se nombran personas ni se indica el tamaño del
equipo: `nosotros.html` presenta **áreas de trabajo**, no integrantes. No inventar
cifras de plantilla, clientes ni casos.
- `tailwind.config.js` — theme. Tokens claros M3 (legacy) + tokens Aurora oscuros (`ink`, `cool-violet`, `violet-accent`, `violet-strong`, `on-violet`, `ember`) + fonts `garamond`/`grotesk`.
- `assets/css/tailwind-src.css` — Tailwind directives + `.glass`/`.glass-card`/`.inner-glow-top` + scroll reveal + animaciones legacy (marquee, liquid-orb)
- `assets/css/tailwind.css` — **generated**, committed
- `assets/js/script.js` — mobile menu, scroll reveal (`[data-reveal]`), form submit (Web3Forms, AJAX)
- `assets/js/hero-fx.js` — efectos del home oscuro: shader aurora/mesh (WebGL, `#aurora-shader`), dot grid interactivo (`#dot-grid`), mouse parallax (`.parallax-layer`). Cada efecto se autodesactiva si falta su elemento o con `prefers-reduced-motion`. Se incluye en todas las páginas (subpáginas solo usan el shader de fondo).
- `stitch/` — export de referencia de Stitch (mockup + design_md). No se sirve.
- `assets/img/` — images + favicon + OG image
- `sitemap.xml` / `robots.txt` — SEO, served from root
- `.assetsignore` — controls which files Cloudflare serves (excludes toolchain files)

## Commands

```bash
npm install        # deps: tailwindcss v3 + @tailwindcss/forms
npm run build      # tailwind-src.css → tailwind.css (minify)
npm run watch      # recompile on change
```

After editing HTML/JS: `npm run build`, then commit `tailwind.css`. Preview by opening any `.html` in browser.

## Design conventions

- **Tema Aurora (dark), todo el sitio.** `<html class="dark">`, `body` = `bg-ink font-grotesk text-white/90`.
- **Headings:** EB Garamond (`font-garamond`), `font-normal` (400) — editorial.
- **Body:** Hanken Grotesk (`font-grotesk`).
- **Palette:** fondo `ink` `#0a0a0a` / `ink-soft` `#131313`. Acentos: `violet-accent` `#c5c0ff`
  (primario), `cool-violet` `#9d85ff`, `ember` `#f2994a`. Texto: `white/90` · `white/60` · `white/40`.
  Bordes: `white/10`. Botón texto: `on-violet` `#1e0090`.
- **Nota tokens:** los tokens M3 claros (`primary`/`surface`/`secondary`/…) siguen en el config pero
  YA NO se usan en las páginas (quedaron como legacy). El home usa los tokens Aurora nuevos + utilidades
  `white/xx`. No mezclar: para superficies usar `.glass`/`.glass-card`, no `bg-surface-*`.
- **Glass:** `.glass` (paneles estáticos) y `.glass-card` (con hover) en `tailwind-src.css`.
- **Buttons:** pill (`rounded-[100px]`/`rounded-full`), primario `bg-violet-accent text-on-violet`.
- **Cards:** `.glass`/`.glass-card` + `rounded-lg`. Inputs: `bg-white/5 border-white/10`.
- **Form:** POST to `https://api.web3forms.com/submit`, `access_key` in hidden input, `class="contact-form"`
  + `.form-note` para el estado. Submission AJAX via `script.js` (no page reload).
- **WhatsApp:** `wa.me/5491138550991`
- **BiFrost demo:** `https://demo.bifrost-software.com.ar/login` (demo@test.com / demo123)
- **Dominio:** `kreta.com.ar`
- **OG image:** `assets/img/og-image.svg`

## Deploy

Cloudflare Pages — static, no build step. Push to main, compiled `tailwind.css` is served.

---
> Source: [SethBastianUade/Web-Kreta](https://github.com/SethBastianUade/Web-Kreta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
