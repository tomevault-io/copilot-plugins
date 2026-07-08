## encsostenible

> Sitio web personal y de campaña de **Eduardo Noriega Campos**, candidato al Parlamento Andino por Integridad Democrática. Construido con Next.js 16 App Router.

# Guía del Proyecto — encsust4in4ble.earth

## Descripción general

Sitio web personal y de campaña de **Eduardo Noriega Campos**, candidato al Parlamento Andino por Integridad Democrática. Construido con Next.js 16 App Router.

**URL de producción:** https://www.encsust4in4ble.earth

---

## Stack tecnológico

| Herramienta | Uso |
|---|---|
| Next.js 16 (App Router) | Framework principal |
| React 19 | UI |
| Tailwind CSS | Estilos |
| TypeScript | Tipado |
| `react-pdf` + PDF.js | Visores de PDF |
| `embla-carousel-react` | Carrusel del hero |
| `lucide-react` | Iconografía |
| `sharp` | Procesamiento de imágenes para favicons/OG |

---

## Estructura de páginas

```
app/
├── page.tsx                          → Sitio público principal (landing page)
├── layout.tsx                        → Root layout con metadatos, polyfill, structured data
├── calculadora-fiscal/
│   ├── page.tsx                      → Calculadora fiscal (dos tabs)
│   └── layout.tsx                    → Metadatos SEO de la calculadora
├── intranet/
│   ├── page.tsx                      → Página de login (contraseña: ID2026)
│   └── hub/
│       └── page.tsx                  → Panel principal del área privada
├── videos/page.tsx                   → Galería de videos (protegida)
├── propuesta-parlamento-andino/
│   ├── page.tsx                      → Visor PDF (protegido)
│   └── PDFViewer.tsx                 → Componente react-pdf
├── integridad-democratica/page.tsx   → Visor PDF (protegido)
├── robots.ts                         → Genera robots.txt
├── sitemap.ts                        → Genera sitemap.xml
├── structured-data.tsx               → JSON-LD (Person, PoliticalParty, WebSite)
├── polyfills.ts                      → Promise.withResolvers para móviles antiguos
└── [icon/apple-icon/opengraph-image] → Imágenes de metadatos generadas con sharp
```

---

## Componentes principales

```
components/
├── principal/
│   ├── Navigation.tsx          → Navbar del sitio público
│   ├── HeroSection.tsx         → Sección hero con carrusel de imágenes
│   ├── CredibilitySection.tsx  → Sección de credibilidad (video + logos)
│   ├── IntegrationShowcase.tsx → Video embebido de YouTube
│   ├── EcosystemGrid.tsx       → Grid de logos de alianzas
│   ├── ManifestoSection.tsx    → Texto del manifiesto personal
│   ├── ModelSection.tsx        → Hélice Quíntuple (5 actores)
│   ├── SolutionsSection.tsx    → Tres soluciones / servicios
│   ├── ImpactSection.tsx       → Casos de estudio (6 proyectos)
│   ├── ContactSection.tsx      → Información de contacto + Calendly
│   ├── ContactForm.tsx         → Sección CTA de la calculadora fiscal
│   └── Footer.tsx              → Pie de página
├── IntranetGuard.tsx           → HOC que protege páginas con localStorage
├── IntranetNavbar.tsx          → Navbar compartido de todas las páginas de intranet
└── ui/                         → Componentes shadcn/ui (Button, Card, etc.)
```

---

## Área privada (Intranet)

### Credencial de acceso

| Campo | Valor |
|---|---|
| URL de login | `/intranet` |
| Contraseña | `ID2026` |
| Almacenamiento | `localStorage` key: `enc_intranet_access` = `"granted"` |

### Flujo de autenticación

1. Usuario visita una página protegida (ej. `/videos`)
2. `IntranetGuard` detecta que no hay acceso → guarda la URL en `sessionStorage("enc_intranet_redirect")` → redirige a `/intranet`
3. Usuario ingresa contraseña → `grantAccess()` guarda en `localStorage`
4. Redirect a `/intranet/hub` (o a la URL guardada si vino de una página específica)

### Páginas protegidas

| Ruta | Contenido |
|---|---|
| `/intranet/hub` | Panel de navegación |
| `/videos` | Galería de YouTube (videos de campaña) |
| `/propuesta-parlamento-andino` | PDF: propuesta al Parlamento Andino |
| `/integridad-democratica` | PDF: presentación del partido |

### PDFs requeridos en `/public`

```
public/
├── propuesta-parlamento-andino.pdf
├── integridad-democratica.pdf
└── pdf.worker.min.mjs              → Worker de PDF.js (necesario para react-pdf)
```

---

## Calculadora fiscal (`/calculadora-fiscal`)

Dos tabs:

### Tab 1: Calculadora Diagnóstica
- Wizard de 3 pasos: Tamaño → Idea → Plan
- Calcula beneficio fiscal por Ley 30309
- Fórmulas clave:
  - MIPYME: ingresos ≤ 2300 × UIT (S/ 5,350) → deducción 240%
  - Gran empresa → deducción 190%
  - `ahorroIR = presupuesto × factor × 0.295`
- Resultado muestra: categoría, ahorro estimado, semáforo de viabilidad, bloque premium bloqueado

### Tab 2: Simulador CONCYTEC
- 5 inputs en tiempo real: Inversión, Ventas, Costos, GCAF, Tipo empresa
- Tabla comparativa SIN vs CON Ley 30309
- 3 métricas: Escudo Total, Escudo Adicional, Costo Neto
- Accesible directamente vía `/calculadora-fiscal?tab=simulador`

---

## SEO y metadatos

- `metadataBase`: `https://www.encsust4in4ble.earth`
- Google Site Verification: configurado vía `verification.google` en `layout.tsx`
- Structured data: JSON-LD en `<head>` vía `StructuredData` component
- Canonical URLs en cada layout de página
- `robots.ts` y `sitemap.ts` generan archivos en tiempo de build
- Redirect HTTP → HTTPS y sin-www → www configurado en `next.config.ts`

---

## Archivos de configuración clave

### `next.config.ts`
- Security headers (X-Frame-Options, X-Content-Type-Options, etc.)
- Redirect `encsust4in4ble.earth` → `www.encsust4in4ble.earth`

### `app/polyfills.ts`
- `Promise.withResolvers` polyfill para browsers móviles (iOS Safari antiguo)
- Necesario porque `embla-carousel` v8 usa ES2024

### `lib/intranet-auth.ts`
- `checkAccess()` / `grantAccess()` / `revokeAccess()`

---

## Comandos de desarrollo

```bash
npm run dev        # Servidor local en http://localhost:3000
npm run build      # Build de producción
npm run start      # Servidor de producción local
npx tsc --noEmit   # Verificar tipos sin compilar
```

---

## Imágenes y assets

```
src/images/
├── photos_background/  → Fotos del hero (001–004img_PORTADA_NOR)
├── proyectos/          → Fotos de casos de estudio (001–006)
└── logos/              → Logos de alianzas (BID, ONU, GIZ, MINAM, etc.)

public/
├── nuevo_favicon.png   → Fuente del favicon (icono circular CIBS)
├── nuevo_imagen.png    → Fuente del OG image (logo CIBS completo)
└── pdf.worker.min.mjs  → Worker PDF.js
```

---

## Notas importantes

- **No hay autenticación server-side.** El sistema de intranet usa `localStorage` del navegador. Es suficiente para proteger contenido no crítico, pero no para datos sensibles.
- **Los PDFs deben subirse manualmente** a `/public`. No están versionados en git.
- **El build es completamente estático** (todos los routes son `○ Static`). Compatible con Vercel, Netlify o cualquier CDN.
- **Sin base de datos ni API routes** propias. Todo el cálculo es client-side.

---
> Source: [Afkaqui/encsostenible](https://github.com/Afkaqui/encsostenible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
