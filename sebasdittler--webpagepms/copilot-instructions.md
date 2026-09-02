## webpagepms

> Sitio web público de Hagrids, empresa de alquileres turísticos en Villa La Angostura, Patagonia Argentina. Las modificaciones siempre se hacen en este repositorio.

# Webhagrids — Sitio Web Público de Hagrids

Sitio web público de Hagrids, empresa de alquileres turísticos en Villa La Angostura, Patagonia Argentina. Las modificaciones siempre se hacen en este repositorio.

## Estructura

Sitio estático de un solo archivo HTML, sin frameworks ni bundler:

```
index.html   # Todo el sitio: HTML, CSS inline y JavaScript
robots.txt   # Directivas SEO para crawlers
sitemap.xml  # Mapa del sitio (URL canónica en Vercel)
```

## Stack

- HTML/CSS/JS puro — sin frameworks, sin build step
- **Fuentes**: Cormorant Garamond (títulos, serif elegante) + DM Sans (cuerpo)
- **Backend/datos**: Firebase Firestore (solo lectura — carga propiedades y actividades en tiempo real)
- **Deploy**: Vercel (`https://web-page-pms.vercel.app/`)

## Paleta de colores

Variables CSS definidas en `:root`:

| Variable | Valor | Uso |
|---|---|---|
| `--forest` | `#1b3a2d` | Fondo oscuro principal, nav |
| `--forest-mid` | `#2d5a43` | Hover de botones |
| `--leaf` | `#4a8c6a` | Acento principal (botones, tags) |
| `--sage` | `#8ab09a` | Acento secundario, textos sutiles |
| `--cream` | `#f5f0e8` | Fondos claros de secciones |
| `--cream-dark` | `#e8e0d0` | Fondos alternos |
| `--white` | `#fdfaf6` | Fondo de cards |

## Secciones (vistas)

La navegación es por estado `cur` (sin router). Cada sección es un `div.vista` que se muestra con la clase `.on`:

| ID | Sección | Contenido |
|---|---|---|
| `v-inicio` | Inicio | Hero con slideshow, grilla de propiedades, actividades, partners — todo cargado desde Firestore |
| `v-propiedades` | Propiedades | Listado completo de propiedades con galería, amenities, calendario de disponibilidad y CTA WhatsApp |
| `v-preguntas` | Preguntas frecuentes | FAQ expandible |
| `v-nosotros` | Nosotros | Historia y valores de Hagrids |
| `v-servicios` | Servicios | Servicios ofrecidos |

## SEO

- Meta tags básicos + Open Graph configurados en el `<head>`
- Schema.org: `LodgingBusiness` + `FAQPage` (JSON-LD)
- `sitemap.xml` y `robots.txt` apuntando a la URL de Vercel

## Contacto configurado

- WhatsApp: `+5492944717718`
- Email: `hagridsvla@gmail.com`
- Instagram: `@somoshagrids`

## Convenciones

- No introducir frameworks ni bundlers — todo debe funcionar como HTML estático
- Los estilos van en el `<style>` dentro del `<head>` del `index.html`
- El JavaScript va en `<script>` al final del `<body>`
- Textos en español (es_AR)
- Los datos de propiedades y actividades se leen desde Firebase Firestore en tiempo real

---
> Source: [Sebasdittler/WebPagePMS](https://github.com/Sebasdittler/WebPagePMS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
