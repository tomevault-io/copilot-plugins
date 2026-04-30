## fuego-latino-landing

> Estándar de SEO para landings y sitios web – metadata, JSON-LD, Search Console, Google Business Profile, directorios y keywords.


# SEO – Estándar del proyecto

Todo proyecto web público debe seguir esta convención de SEO para maximizar visibilidad en buscadores.

---

## 1. Metadata (Next.js App Router)

En el `layout.tsx` raíz, exportar `metadata` con al menos:

- **title:** con `default` y `template` (ej. `"%s | Mi Negocio"`).
- **description:** 1-2 oraciones con keywords principales + nombre del negocio + ubicación + CTA.
- **keywords:** array de 15-25 términos relevantes. Incluir variaciones con ciudad, abreviatura de ciudad (ej. "MTY"), y frases coloquiales como "cerca de mí".
- **metadataBase:** URL canónica del sitio (ej. `new URL("https://midominio.com")`).
- **alternates.canonical:** `"/"`.
- **openGraph:** type, locale, url, title, description, siteName, images (1200×630 mínimo).
- **twitter:** card `summary_large_image`, title, description, images.
- **robots:** `{ index: true, follow: true, googleBot: { ... } }`.
- **verification.google:** código de Google Search Console (ver sección 4).

### Páginas privadas (no indexar)

Para rutas que no deben aparecer en buscadores (ej. `/competitions/`, `/admin/`):

- En el `layout.tsx` de esa ruta: `robots: { index: false, follow: false }`.
- En `robots.txt`: añadir `Disallow: /ruta-privada/`.

---

## 2. JSON-LD (Structured Data)

Incluir un bloque `<script type="application/ld+json">` en el `<head>` del layout con:

- **@type:** según el negocio (ej. `["DanceSchool", "LocalBusiness"]`, `["Restaurant", "LocalBusiness"]`).
- **name, description, url, logo, image.**
- **address:** con `streetAddress`, `addressLocality`, `addressRegion`, `addressCountry`.
- **telephone, priceRange.**
- **openingHoursSpecification:** array con días y horarios reales (mantener sincronizado con la UI).
- **sameAs:** links a redes sociales (Instagram, Facebook, etc.).
- **offers / hasOfferCatalog:** servicios que ofrece el negocio.

### Mantener sincronizado

Cuando cambien horarios, dirección, servicios o teléfono en la UI, actualizar también el JSON-LD.

---

## 3. Archivos estáticos de SEO

- **`public/robots.txt`:** Allow /, Disallow /api/ y /_next/, listar rutas privadas, incluir `Sitemap: https://midominio.com/sitemap.xml`.
- **`app/sitemap.ts`:** generar sitemap dinámico con todas las páginas públicas. Usar `changeFrequency` y `priority` apropiados.
- **`public/manifest.json`:** nombre, colores, iconos para PWA.
- **Favicon e iconos:** `/favicon.ico`, `/logo.png` (512×512).

---

## 4. Google Search Console (verificación e indexación)

Pasos para cada proyecto nuevo:

1. Ir a [search.google.com/search-console](https://search.google.com/search-console).
2. Añadir propiedad con **"Prefijo de la URL"** → `https://midominio.com`.
3. Verificar con **"Etiqueta HTML"** (meta tag). Google da un código como: `<meta name="google-site-verification" content="CODIGO" />`.
4. Agregar el código en `metadata.verification.google` del `layout.tsx`:
   ```ts
   verification: {
     google: "CODIGO-AQUI",
   },
   ```
5. Desplegar a producción.
6. Volver a Search Console → **Verificar**.
7. Ir a **Sitemaps** → enviar `https://midominio.com/sitemap.xml`.
8. Ir a **Inspección de URLs** → solicitar indexación de la página principal.

---

## 5. Google Business Profile (SEO local)

Para negocios con ubicación física:

1. Crear perfil en [business.google.com](https://business.google.com).
2. Completar: nombre, dirección exacta, categoría, horarios, teléfono, sitio web.
3. Subir fotos del negocio (mínimo 5-10).
4. Pedir reseñas a clientes/alumnos.
5. Mantener **NAP consistente** (Name, Address, Phone) en todos los directorios y redes.

---

## 6. Directorios y backlinks

Registrar el negocio en directorios relevantes para generar citaciones y backlinks:

- Sección Amarilla (MX)
- Yelp México
- Directorios especializados del rubro
- Google Maps (vía Business Profile)
- Facebook e Instagram (con dirección y teléfono consistentes)

---

## 7. Keywords – convenciones

- Incluir variaciones con ciudad completa y abreviada (ej. "monterrey" y "mty").
- Incluir frases coloquiales: "cerca de mí", "clases de [servicio]".
- Incluir nombre del negocio como keyword.
- Incluir servicios nuevos cuando se agreguen a la UI.
- Actualizar `metadata.description` cuando se añadan servicios o cambie la oferta.

---

## 8. Checklist para proyecto nuevo

- [ ] `layout.tsx` con metadata completa (title, description, keywords, OG, Twitter, robots).
- [ ] JSON-LD con datos del negocio.
- [ ] `robots.txt` con Sitemap.
- [ ] `sitemap.ts` con páginas públicas.
- [ ] `manifest.json` con nombre y colores.
- [ ] Google Search Console verificado y sitemap enviado.
- [ ] Google Business Profile creado (si aplica).
- [ ] Negocio registrado en al menos 3 directorios.
- [ ] Reseñas solicitadas a clientes.

---

## 9. Mantenimiento

- Ejecutar `npm audit` periódicamente y actualizar dependencias con vulnerabilidades.
- Revisar Search Console mensualmente: errores de rastreo, páginas indexadas, consultas.
- Actualizar keywords y description cuando se agreguen servicios o cambien horarios.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fercreek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
