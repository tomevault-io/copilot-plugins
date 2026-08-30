## website-gocas

> Contexto para Claude Code / contribuidores. La app vive en [`gocas-landing/`](gocas-landing/). Para correrla, env vars y deploy: ver [README.md](README.md).

# CLAUDE.md · GOCAS-Website

Contexto para Claude Code / contribuidores. La app vive en [`gocas-landing/`](gocas-landing/). Para correrla, env vars y deploy: ver [README.md](README.md).

## Qué es esto

Sitio web de **GOCAS Automations** — estudio boutique de software/automatización para PYMEs LATAM. Una landing (Next.js 14, App Router) con páginas de servicios y contacto, respaldada por Supabase.

## Comandos

```bash
cd gocas-landing
npm install
npm run dev      # desarrollo · http://localhost:3000
npm run build    # SIEMPRE correr antes de dar por terminado un cambio
npm run lint
```

## Arquitectura

- **App Router** (`app/`). La home (`app/page.tsx`) es `force-dynamic` y compone secciones en este orden: `Nav → Hero → Servicios → Proceso → Casos → Nosotros → CtaContacto → Footer`.
- **Estilos:** inline CSS-in-JS con tokens centralizados en [`lib/tokens.ts`](gocas-landing/lib/tokens.ts) (objeto `G`). NO hay sistema de componentes de UI; Tailwind está disponible pero apenas se usa (`globals.css`). `border-radius: 0` global — estética bauhaus, bordes duros.
- **Datos:** Supabase vía [`lib/supabase.ts`](gocas-landing/lib/supabase.ts) (`getSupabaseAdmin`, **solo servidor**, lazy). `Servicios` lee `service_packages`; el formulario escribe en `leads`.
- **Formulario de contacto:** [`app/actions/leads.ts`](gocas-landing/app/actions/leads.ts) (server action) valida con Zod → inserta en Supabase → notifica por email vía Resend (opcional).
- **Logo:** [`components/Logo.tsx`](gocas-landing/components/Logo.tsx) dibuja el wordmark en código (no es una imagen). Favicons/OG/assets en [`public/`](gocas-landing/public/); OG generado en runtime en `app/opengraph-image.tsx`.
- **Portal interno (`/portal`):** login del equipo con Supabase Auth (`@supabase/ssr`). Clientes de auth en `lib/supabase/{server,client}.ts`; helper `lib/team.ts` (`getCurrentMember`); protección en `middleware.ts`. `/login` (público por URL, no enlazado), `/portal` (panel), `/portal/usuarios` (solo socios: alta de empleados con el admin client). Degrada sin romper la landing si faltan las env de auth.

## Reglas al editar

- **Usa los tokens de `lib/tokens.ts`** para todo color. Brackets del logo y acentos siempre en ámbar (`G.amber`).
- **Marca:** paleta oliva/ámbar/crema, Manrope + JetBrains Mono, regla 70·25·5. Sistema completo en [`design_handoff_gocas_landing/README.md`](design_handoff_gocas_landing/README.md).
- **Voz del copy:** se tutea siempre · resultados, no stack · cálido, anti-cliché · sin emojis decorativos.
- **Responsive:** breakpoints en `globals.css` (`@media max-width: 900px / 560px`) usando clases `gocas-*`. Si agregas una sección con grid, dale una clase `gocas-*` y manéjala ahí.
- Si agregas/quitas un link de navegación, **actualiza ambos**: `components/sections/Nav.tsx` y `components/MobileNav.tsx`.
- Los socios/fundadores **no se muestran** en el sitio (decisión de negocio). La sección "Nosotros" habla de la empresa, no de personas.

## Base de datos

Estructura documentada en [`02_GOCAS_Supabase_Database_Structure.md`](02_GOCAS_Supabase_Database_Structure.md). Migración del formulario: [`gocas-landing/supabase/migrations/0001_leads.sql`](gocas-landing/supabase/migrations/0001_leads.sql).

## No commitear

`.env*.local`, secretos, y documentos internos de negocio (prefijo `01_…`, ya en `.gitignore`).

---
> Source: [GOCAS-Automations/WEBSITE_gocas](https://github.com/GOCAS-Automations/WEBSITE_gocas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
