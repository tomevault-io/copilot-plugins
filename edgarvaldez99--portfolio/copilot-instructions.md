## portfolio

> - Portafolio personal hospedado en **Cloudflare Pages** (plan gratuito)

# Portfolio - valdezdev.work

## Contexto del proyecto
- Portafolio personal hospedado en **Cloudflare Pages** (plan gratuito)
- Dominio: **valdezdev.work**
- Email de contacto: **contact@valdezdev.work** (configurado con Cloudflare Email Routing → Gmail)
- Framework: **Astro** (template minimal)

## Infraestructura Cloudflare (ya configurado)
- SSL/TLS: Full (Strict)
- HSTS: Activado (6 meses, incluye subdominios, no-sniff ON, preload OFF)
- Always Use HTTPS: ON
- Minimum TLS: 1.2
- Automatic HTTPS Rewrites: ON
- Bot Fight Mode: ON
- Browser Integrity Check: ON
- Email Routing: Activo

## Restricciones
- Todo debe ser **gratuito** — no usar servicios de pago ni features "Unlock" de Cloudflare
- Cloudflare Pages free tier: 500 builds/mes, bandwidth ilimitado
- No agregar dependencias innecesarias, mantener el proyecto ligero

## Estructura de subdominios (planeado)
- `valdezdev.work` → Portafolio (este proyecto)
- Subdominios futuros para otros proyectos (blog, demos, APIs)

## Deploy
- Plataforma: Cloudflare Pages
- Conectar repositorio de GitHub para deploy automático
- Build command: `npm run build`
- Output directory: `dist`

## Idioma
- Responder siempre en **Español**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edgarvaldez99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
