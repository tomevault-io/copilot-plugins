## mercadord

> Marketplace estilo eBay para República Dominicana. **HTML + JS vanilla, sin bundler ni framework, sin build.** Sitio estático servido por Vercel.

# MercadoRD — contexto del proyecto

Marketplace estilo eBay para República Dominicana. **HTML + JS vanilla, sin bundler ni framework, sin build.** Sitio estático servido por Vercel.

## Carpetas (importante)
- **Editar aquí:** `D:\mercadord v2\mercadord` (cwd de trabajo, SIN git)
- **Desplegar:** copiar los archivos cambiados a `D:\mercadord` (repo git → GitHub `Warrior-R/mercadord` → Vercel), commit, push. Ver memoria [[mercadord-deploy-setup]]: tras el push a veces hay que `vercel promote <url>` porque los dominios no se reasignan solos.

## Archivos
| Archivo | Qué es | Tamaño aprox |
|---|---|---|
| `index.html` | Toda la UI: modales (auth, verificación, puja, oferta), header, footer, subsecciones | ~60 KB |
| `js/app.js` | Lógica principal: render productos, carrito, checkout, subastas, favoritos, verificación, captura QR móvil | ~70 KB |
| `js/auth.js` | Auth Supabase (login/registro/Google/recuperación), gates `requireAuth`, validación cédula | ~20 KB |
| `js/data.js` | Datos demo (productos, subastas) + contenido legal/informativo de modales | ~25 KB |
| `js/storage.js` | Capa localStorage (`MRD`, claves `K`) | pequeño |
| `css/styles.css` | Global + responsive (breakpoints 900/700/380) | |
| `css/auth.css`, `carousel.css`, `footer.css`, `subsections.css`, `verification.css` | Por componente | |
| `supabase/setup.sql` | Esquema BD (profiles, verifications, products, orders, favorites) + RLS + triggers | |
| `supabase/functions/kyc/index.ts` | Edge Function: crea sesiones Didit + recibe webhook firmado | |

Al editar, lee solo la sección/función relevante (usa Grep), no el archivo entero.

## Estado real (todo activo, jun 2026)
- **Supabase** proyecto `flsixfuzvbapwnfepmwr`, `SUPABASE_ENABLED=true` en `js/auth.js` (la publishable key es pública por diseño; la seguridad la dan las políticas RLS). Cuentas reales con verificación por email + Google OAuth.
- **KYC con Didit** activo — ver memoria [[mercadord-kyc-didit]]. La aprobación de identidad es 100% server-side (Edge Function + webhook firmado); el trigger `protect_profiles_verification` impide auto-verificarse.
- Token CLI de Supabase del usuario y la Management API (cuerpo en bytes UTF-8 desde PowerShell) documentados en memoria.

## Convenciones
- UI y comentarios en **español**. Handlers `onclick` inline en el HTML llaman funciones globales de `app.js`/`auth.js`.
- Orden de carga de scripts: storage → data → auth → app (el SDK de Supabase carga con `defer`, init perezoso en `auth.js`).
- Estado en `userState` (localStorage) + sesión Supabase. `DEMO`/`sb` distinguen modo demo vs real.

## Verificar cambios
Servidor de preview: `preview_start` con `.claude/launch.json` (sirve estático en :4173). Para móvil: `preview_resize` a 390×844 y comprobar `document.documentElement.scrollWidth === clientWidth` (no debe haber scroll horizontal).

## Responsive
Header de dos filas en ≤700px (búsqueda a ancho completo, "+Vender" oculto porque está en `.bottom-nav`). `html/body{overflow-x:hidden}` de seguridad. No introducir elementos de ancho fijo que desborden el viewport.

---
> Source: [Warrior-R/mercadord](https://github.com/Warrior-R/mercadord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
