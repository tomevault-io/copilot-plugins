## mdv8classics

> Este repositorio contiene la reconstrucción completa de la web de MDV8 Classics. Antes de modificar diseño, contenido, SEO, integraciones o comercio, lee `PROJECT_STATE.md` y los documentos relevantes de `docs/`.

# MDV8 Classics — instrucciones del proyecto

Este repositorio contiene la reconstrucción completa de la web de MDV8 Classics. Antes de modificar diseño, contenido, SEO, integraciones o comercio, lee `PROJECT_STATE.md` y los documentos relevantes de `docs/`.

## Orden de autoridad

1. Solicitud actual y aprobaciones explícitas del usuario.
2. Este archivo.
3. `PROJECT_STATE.md`.
4. Documentos numerados de `docs/`.
5. Decisiones registradas en `docs/09-DECISION-LOG.md`.

Si dos fuentes se contradicen, detén la decisión afectada, registra el conflicto y solicita confirmación. No sobrescribas decisiones aprobadas silenciosamente.

## Hechos comerciales confirmados

- Nombre público: MDV8 Classics.
- Dirección física oficial: 1445 Brooks St Ste H, Ontario, CA 91762, United States.
- Web actual: https://mdv8classics.com/
- Instagram: https://www.instagram.com/mdv8classics/
- Google Business Profile: https://maps.app.goo.gl/3k8A7bFV4z6zArog8
- Mercado principal: Ontario y comunidades reales del Inland Empire y áreas cercanas.
- Idioma público inicial: inglés estadounidense.
- Idioma de documentación interna: español.

El teléfono, email, horarios, disponibilidad de servicio móvil, políticas comerciales y texto legal permanecen sin confirmar. No los inventes. El teléfono `951-286-8559` pertenece al sitio anterior y debe validarse antes de publicarse.

## Alcance vigente

- Fase inicial: landing de marketing con experiencia premium, Instagram y merch.
- La tienda puede requerir rutas utilitarias (`/shop`, confirmación de compra y políticas), pero no se crearán todavía páginas SEO por ciudad o servicio.
- La arquitectura SEO por keywords queda aplazada hasta recibir investigación, volumen, intención, competidores y prioridades comerciales.
- No crear doorway pages, páginas de ciudades repetitivas ni bloques de keyword stuffing.

## Reglas de marca y experiencia

- La dirección es “Southern California classic garage meets 1970s rock editorial”.
- Debe sentirse auténtica, artesanal, contundente y cinematográfica; nunca como una plantilla genérica de taller.
- Prioriza fotos, videos, audio visual y proyectos reales del cliente.
- ImageGen puede producir conceptos, fondos o texturas, pero nunca vehículos falsos presentados como trabajos del taller.
- Evita clichés automotrices sin intención: llamas genéricas, calaveras decorativas, exceso de cromado, dashboards falsos o legibilidad sacrificada.
- Todo movimiento debe respetar `prefers-reduced-motion`, mantener navegación por teclado y proteger Core Web Vitals.
- Nunca activar audio automáticamente.

## SEO y contenido

- Mantén el NAP exactamente consistente con la dirección confirmada.
- Describe MDV8 como negocio ubicado en Ontario; San Bernardino sólo puede aparecer como área atendida si el cliente lo confirma.
- No inventes premios, certificaciones, años de experiencia, cifras de clientes, garantías, reviews o disponibilidad 24/7.
- Usa datos estructurados únicamente para contenido visible y verificable.
- Cada futura página orgánica tendrá una intención y primary keyword propia, definida en el futuro keyword map.
- Conserva redirects del sitio anterior cuando se conozca su inventario de URLs.

## Instagram

- Mostrar entre 10 y 15 publicaciones recientes, priorizando reels.
- Usar acceso autorizado, fetch de servidor, caché y fallback; no scraping frágil.
- Cargar posters optimizados y reproducir video bajo interacción. No descargar ni republicar contenido sin autorización.
- Si falla la API, la página debe conservar su estructura y mostrar un CTA al perfil.

## Comercio y seguridad

- Proveedor provisional: Stripe Checkout.
- No almacenar datos de tarjetas en la aplicación.
- Crear sesiones en servidor, validar precios del lado servidor y verificar firmas de webhooks.
- Usar modo de pruebas hasta autorización expresa de lanzamiento.
- Nunca guardar secretos en Git; documentar nombres en `.env.example` cuando exista código.
- No inventar precios, tallas, inventario, envíos, impuestos ni políticas de devolución.

## Flujo para goals y loops

1. Lee el estado y define un resultado verificable.
2. Identifica hechos confirmados, supuestos y entradas faltantes.
3. Implementa el cambio más pequeño que produzca el resultado completo.
4. Ejecuta pruebas proporcionales al riesgo.
5. Realiza revisión visual en desktop y mobile cuando haya UI.
6. Actualiza `PROJECT_STATE.md` y el log de decisiones si cambió el proyecto.
7. No marques el goal como completo mientras queden fallos obligatorios o verificaciones pendientes.

## Quality gate mínimo

- Typecheck, lint, tests y build exitosos.
- Navegación y formularios probados por teclado.
- Sin errores críticos de accesibilidad automatizada.
- Sin desbordamiento horizontal en viewports móviles.
- SEO técnico, metadata, canonical, robots, sitemap y JSON-LD inspeccionados.
- Imágenes y videos optimizados; lazy loading fuera del contenido principal.
- Enlaces, teléfono, email, dirección y CTAs verificados.
- Checkout y webhook probados en sandbox cuando exista comercio.
- Estados loading, vacío, error y fallback verificados para integraciones externas.

## Uso de agentes y skills

- Usa `.agents/skills/mdv8-creative-guardian` para creación o revisión visual.
- Usa `.agents/skills/mdv8-release-gate` antes de declarar una entrega lista.
- La futura skill de SEO local sólo se creará después del keyword research.
- Los agentes revisores son temporales: dirección creativa/UX, SEO local y QA; el agente principal integra resultados y resuelve contradicciones.


<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [arielomesias/mdv8classics](https://github.com/arielomesias/mdv8classics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
