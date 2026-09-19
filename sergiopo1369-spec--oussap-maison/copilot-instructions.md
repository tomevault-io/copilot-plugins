## oussap-maison

> > Copia este prompt completo y pégalo en el agente que vaya a construir el proyecto (Claude Code, Cursor, etc.).

# SÚPER PROMPT — "Bissap Maison": web para vender productos naturales de hibisco (Reims / Châlons)

> Copia este prompt completo y pégalo en el agente que vaya a construir el proyecto (Claude Code, Cursor, etc.).
> Está escrito para ejecutarse en fases y es auto-contenido. Objetivo: **algo simple pero profesional**, listo para vender por WhatsApp.

---

## 0. Contexto del negocio

- Producto artesanal y **fait maison** a base de **hibisco** (*Hibiscus sabdariffa*, la flor con la que se hace el **bissap**).
- Venta **local**: zona de **Reims** y **Châlons-en-Champagne** (Marne, Francia).
- Canal de pedidos: **WhatsApp** (sin carrito, sin pasarela de pago en la v1).
- Mercado principal: **francófono** → la web se redacta en **francés**. (Opcional futuro: versión ES/EN.)
- Tono: natural, cercano, honesto, hecho a mano. Nada de promesas médicas.

---

## 1. ROL

Actúas como un **equipo combinado**:

1. **Diseñador/desarrollador web senior** especializado en *landing pages* de producto artesanal: rápidas, limpias, mobile-first, alta conversión.
2. **Redactor de marca y SEO local** con experiencia en pequeño comercio / alimentación artesanal en Francia, que conoce las reglas de **alegaciones de salud (Reglamento UE 1924/2006)** y redacta copy evocador sin infringirlas.

Trabajas de forma modular, documentas cada decisión y **no inventas propiedades del producto**: todo beneficio se expresa en lenguaje sensorial/tradicional, no como afirmación de salud verificable.

---

## 2. OBJETIVO

Construir **"Bissap Maison"** (nombre provisional, confirmar con el cliente): una web de **una sola página** (one-page, con secciones ancladas) que:

- Presenta la **gama de productos de hibisco** y sus **sabores**.
- Explica que el producto es **versátil**: se puede transformar de líquido a otros formatos.
- Recoge los **pedidos por WhatsApp** con un botón que abre una conversación con mensaje pre-rellenado.
- Muestra la **zona de reparto / punto de recogida** con **Google Maps** integrado (Reims y Châlons).
- Incluye la información **legal mínima** (mentions légales, RGPD, alérgenos, conservación).
- Carga rápido, se ve perfecta en móvil y transmite confianza.

---

## 3. GAMA DE PRODUCTO (confirmar precios y formatos con el cliente)

**Base:** infusión concentrada de flor de hibisco (bissap), elaborada a mano.

**Sabores / variantes:**

| Sabor | FR | Nota de cata sugerida |
|---|---|---|
| Menta | Menthe | fresco, herbáceo, final limpio |
| Jengibre | Gingembre | picante suave, cálido, tónico |
| Vainilla | Vanille | dulce, redondo, envolvente |
| Natural | Nature | ácido, floral, afrutado (base sin aromatizar) |

**Formatos / usos del mismo producto (versatilidad):**

- **Jus / boisson** — listo para beber (diluir al gusto).
- **Sirop / concentré** — para diluir en agua fría o caliente.
- **Infusion chaude** — servir caliente como tisana.
- **Confiture de bissap** — versión mermelada para tostadas, quesos, repostería.
- **En pâtisserie** — sirope o confitura para bizcochos, glaseados, postres ("de líquido a pastel").

> El texto debe dejar claro que **es un solo producto natural** que el cliente puede usar de varias formas, no cinco productos distintos (salvo la confitura, que se vende como tal).

---

## 4. PROPUESTA DE VALOR Y BENEFICIOS — REGLAS DE REDACCIÓN

El cliente quiere comunicar ideas como: *sentirse bien, boost de energía, concentración, fuerza, momento zen*.

**Cómo se redacta (permitido, lenguaje sensorial / de experiencia):**

- « Un moment pour souffler. » / « La pause hibiscus. »
- « Une boisson vive et réconfortante, à partager. »
- « Le rituel bissap : on se pose, on savoure. »
- « Fait main, à Reims, avec des fleurs d'hibiscus et rien d'inutile. »
- « Sans colorant, sans arôme artificiel, sans conservateur ajouté. » *(solo si es cierto — confirmar con el cliente)*

**Cómo NO se redacta (alegación de salud no autorizada — prohibido):**

- ❌ "aumenta la concentración", "da energía", "refuerza el sistema inmunitario", "reduce la tensión", "quema grasa", "desintoxica", "cura / previene …".
- ❌ Cualquier referencia a enfermedades o a efectos fisiológicos concretos.

**Si el cliente insiste en el ángulo "bienestar":** usar la fórmula de **uso tradicional** con matiz claro, p. ej.:
« En Afrique de l'Ouest, le bissap est la boisson conviviale par excellence, servie lors des fêtes et des retrouvailles. » — describe cultura, no promete efectos.

Añadir siempre, discreto, un aviso: « Produit alimentaire artisanal. Ne se substitue pas à une alimentation variée et équilibrée. »

---

## 5. STACK TÉCNICO (simple y profesional)

- **Astro** (recomendado) o **HTML + Tailwind** plano si se quiere aún más simple. Sin framework pesado, sin base de datos.
- **Tailwind CSS** para estilos.
- **Sin backend en la v1**: los pedidos van por WhatsApp (`https://wa.me/…` con `?text=` pre-rellenado).
- **Imágenes**: `<picture>` / formato WebP, `loading="lazy"`, `alt` descriptivo en francés. Comprimir a < 200 KB cada una.
- **Fuentes**: 1 tipografía con carácter para títulos + 1 sans legible para texto (self-hosted o `font-display: swap`).
- **Despliegue**: **Netlify** o **Vercel** (deploy desde Git, HTTPS y dominio gratis para empezar).
- **Analítica**: Plausible o nada en la v1 (no bloquear el lanzamiento por esto). Si se añade GA4, incluir banner de cookies.
- **Rendimiento objetivo**: Lighthouse ≥ 95 en móvil, LCP < 2 s.

Si el repo ya tiene un stack definido, respétalo y adapta esta sección.

---

## 6. ESTRUCTURA DE CARPETAS (ejemplo con Astro)

```
bissap-maison/
├── src/
│   ├── pages/
│   │   └── index.astro            # One-page con todas las secciones
│   ├── components/
│   │   ├── Header.astro           # Logo + nav ancla + botón WhatsApp
│   │   ├── Hero.astro             # Claim + foto producto + CTA
│   │   ├── Products.astro         # Gama y sabores (tarjetas)
│   │   ├── Uses.astro             # "De liquide à gâteau" — usos/formatos
│   │   ├── Story.astro            # Fait maison, hibiscus, artesana/o
│   │   ├── OrderWhatsApp.astro    # Cómo pedir + botón wa.me
│   │   ├── MapSection.astro       # Google Maps Reims / Châlons + zona
│   │   ├── FAQ.astro              # Conservación, alérgenos, plazos
│   │   ├── Footer.astro           # Mentions légales, RGPD, redes
│   │   └── WhatsAppButton.astro   # Botón flotante reutilizable
│   ├── layouts/
│   │   └── Base.astro             # <head>, SEO, Open Graph, JSON-LD
│   ├── data/
│   │   └── site.ts                # Config: teléfono, zonas, sabores, precios
│   └── styles/
│       └── global.css
├── public/
│   ├── images/
│   ├── favicon.svg
│   └── og-image.jpg
├── astro.config.mjs
└── README.md
```

Centralizar en `src/data/site.ts` todo lo editable sin tocar componentes:
número de WhatsApp, texto del mensaje pre-rellenado, lista de sabores, precios, dirección/coordenadas del mapa, enlaces de redes, email de contacto.

---

## 7. SECCIONES DE LA WEB (contenido, en francés)

1. **Header fijo**: logo "Bissap Maison", enlaces ancla (Produits · Utilisations · Commander · Nous trouver), botón **« Commander sur WhatsApp »**.
2. **Hero**: foto del producto + claim (« Le bissap fait maison, à Reims. Fleur d'hibiscus, rien d'inutile. ») + CTA WhatsApp + subclaim con sabores.
3. **Produits**: 4 tarjetas (Menthe, Gingembre, Vanille, Nature) con nota de cata, formato y precio. Tarjeta aparte para **Confiture de bissap**.
4. **Utilisations** — « De liquide à gâteau »: bloque visual con los usos (jus, sirop, infusion chaude, confiture, pâtisserie). Una frase por uso.
5. **Notre histoire**: quién lo hace, hecho a mano, la flor de hibisco (*Hibiscus sabdariffa*), origen del bissap, ingredientes. Refuerza confianza (E-E-A-T local).
6. **Commander**: pasos claros — 1) Escríbenos por WhatsApp, 2) Dinos sabores y cantidad, 3) Acordamos recogida o entrega en Reims / Châlons, 4) Pago en efectivo o transferencia a la entrega. Botón grande wa.me.
7. **Nous trouver**: mapa de Google Maps embebido + texto de la **zona cubierta** (Reims, Châlons-en-Champagne y alrededores) + días/horarios de entrega si aplica.
8. **FAQ**: conservación (nevera, X días / caducidad), alérgenos, ¿lleva azúcar?, ¿es sin alcohol? (sí), plazos de preparación, pedidos para eventos.
9. **Footer**: mentions légales, política de privacidad (RGPD), contacto, Instagram/Facebook, aviso alimentario del punto 4.

---

## 8. PEDIDOS POR WHATSAPP (implementación)

- Enlace: `https://wa.me/33XXXXXXXXX?text=<mensaje URL-encoded>` (número francés en formato internacional, sin `+` ni espacios: `33` + 9 dígitos).
- Guardar número y plantilla en `site.ts`. Ejemplo de mensaje pre-rellenado:

  > `Bonjour Bissap Maison ! Je souhaite commander : [sabor + cantidad]. Je suis à [Reims / Châlons]. Merci !`

- **Botón flotante** visible en móvil (esquina inferior derecha), con `aria-label` y contraste AA.
- Todos los CTA de pedido apuntan al mismo enlace.
- No usar formularios que envíen email en la v1 (menos mantenimiento, menos RGPD). Si más adelante se quiere lista de correo, añadir con proveedor detrás de env var.
- `rel="noopener"` y `target="_blank"` en los enlaces externos.

---

## 9. GOOGLE MAPS (implementación)

- Sección **« Nous trouver »** con un `<iframe>` de Google Maps (modo *embed*, sin API key) centrado en el punto de recogida o en la zona Reims–Châlons.
- `loading="lazy"`, `title` descriptivo en francés, `width/height` responsivos (`aspect-ratio`), sin romper el layout móvil.
- Si no hay dirección pública (venta a domicilio), centrar el mapa en **Reims centre** y describir el radio de reparto en texto; añadir un pin aproximado.
- Enlace « Ouvrir dans Google Maps » debajo del mapa.
- Coordenadas y URL del embed en `site.ts`.
- Nota RGPD: el iframe de Google carga recursos de terceros → mencionarlo en la política de privacidad (o cargar el mapa solo tras clic si se quiere ser estricto).

---

## 10. IDENTIDAD VISUAL

- **Paleta**: burdeos / rojo hibisco profundo (#8A1C3B aprox.) como color principal, crema/marfil de fondo, verde hoja o dorado suave como acento. Confirmar con el cliente.
- **Estilo**: natural y artesano — mucho espacio en blanco, fotografía real del producto (nada de stock genérico), detalles botánicos de la flor de hibisco.
- **Logo**: si no hay, proponer uno tipográfico simple + motivo de flor/cáliz de hibisco.
- **Accesibilidad**: contraste AA mínimo, tamaños de toque ≥ 44 px, foco visible, `prefers-reduced-motion` respetado.

---

## 11. SEO LOCAL

- `<title>` ≤ 60 car.: p. ej. « Bissap maison à Reims — hibiscus menthe, gingembre, vanille ».
- `meta description` ≤ 155 car. con gancho + zona + "commande WhatsApp".
- **Un solo H1**, H2 por sección, jerarquía correcta.
- **JSON-LD**: `LocalBusiness` (o `FoodEstablishment`) con `name`, `areaServed` (Reims, Châlons-en-Champagne), `telephone`, `sameAs` (redes), horarios si aplica. Añadir `Product` para cada sabor si hay precio estable.
- **Open Graph / Twitter Card**: título, descripción e imagen (`og-image.jpg`, foto del producto).
- `sitemap.xml` + `robots.txt` (rastreo completo).
- Nombre, palabras clave reales: *bissap Reims*, *jus d'hibiscus Reims*, *confiture bissap*, *boisson hibiscus Châlons*.
- Crear/relacionar con **Google Business Profile** (fuera de la web, pero recomendarlo en el README).
- Idioma: `<html lang="fr">`.

---

## 12. LEGAL Y ALIMENTARIO (Francia / UE — imprescindible antes de publicar)

- **Mentions légales**: identidad del vendedor (nombre o razón social, estatus — auto-entrepreneur/micro, SIRET si lo hay), email de contacto, alojamiento web.
- **RGPD / Politique de confidentialité**: qué datos se tratan (los de WhatsApp los gestiona Meta; si se añade analítica o formulario, detallarlo), base legal, derechos, contacto. Banner de cookies solo si hay trackers/no esenciales.
- **Información alimentaria (INCO / Reg. UE 1169/2011)**: lista de ingredientes, **alérgenos destacados**, condiciones de conservación ("À conserver au réfrigérateur, à consommer sous X jours"), fecha de consumo, cantidad neta. Puede ir en la etiqueta física + resumen en la FAQ.
- **Alegaciones**: aplicar estrictamente el punto 4 (nada de salud no autorizada).
- **Higiene**: recordar en el README que la venta de alimentos preparados en casa requiere declaración de actividad (déclaration en mairie / DDPP) y buenas prácticas de higiene — no es tarea de la web, pero se avisa.
- Si en el futuro se cobra online: habrá que añadir **CGV**, derecho de desistimiento (con excepción de productos perecederos) y pasarela conforme.

---

## 13. PLAN DE EJECUCIÓN POR FASES

1. **Fase 0 — Setup**: crear repo, Astro + Tailwind, estructura del punto 6, `site.ts` con datos placeholder.
2. **Fase 1 — Maqueta**: layout base, header, hero, footer, botón WhatsApp flotante, estilos y paleta.
3. **Fase 2 — Contenido**: redactar en francés todas las secciones del punto 7 siguiendo las reglas de copy del punto 4.
4. **Fase 3 — Pedidos + Mapa**: enlace `wa.me` con mensaje pre-rellenado, iframe de Google Maps, sección « Nous trouver ».
5. **Fase 4 — SEO técnico**: `<head>` completo, JSON-LD `LocalBusiness`, OG, sitemap, robots, favicon, `og-image`.
6. **Fase 5 — Legal**: páginas/bloques de mentions légales, confidentialité, info alimentaria y aviso del punto 4.
7. **Fase 6 — QA + deploy**: Lighthouse móvil ≥ 95, prueba del botón WhatsApp en móvil real, revisión de textos, deploy en Netlify/Vercel, conectar dominio.

---

## 14. CHECKLIST DE ACEPTACIÓN

- [ ] Web de una página, responsive, Lighthouse móvil ≥ 95 (Performance y Accessibility).
- [ ] Botón « Commander sur WhatsApp » abre WhatsApp con el mensaje pre-rellenado, en móvil y escritorio.
- [ ] Botón flotante de WhatsApp visible y accesible en todas las pantallas.
- [ ] Los 4 sabores (menthe, gingembre, vanille, nature) + la confiture están presentados con precio y formato.
- [ ] Sección « Utilisations » explica los usos (jus, sirop, infusion, confiture, pâtisserie) con el mensaje "de liquide à gâteau".
- [ ] Mapa de Google Maps embebido, con zona Reims / Châlons descrita y enlace « Ouvrir dans Google Maps ».
- [ ] Ningún texto contiene alegaciones de salud no autorizadas; incluye el aviso alimentario.
- [ ] Mentions légales, politique de confidentialité e información de alérgenos/conservación presentes.
- [ ] `<title>`, meta description, H1 único, JSON-LD `LocalBusiness`, Open Graph e imagen OG configurados.
- [ ] Todos los datos editables (teléfono, mensaje, sabores, precios, coordenadas, redes) centralizados en `site.ts`.
- [ ] Desplegada en producción con HTTPS y probada en un móvil real.

---

## 15. SUPUESTOS Y DECISIONES ABIERTAS (confirmar con el cliente)

- Nombre definitivo de la marca y dominio (se usa "Bissap Maison" como placeholder).
- Número de WhatsApp real y texto exacto del mensaje pre-rellenado.
- ¿Hay punto de recogida con dirección pública o solo entrega a domicilio? Coordenadas para el mapa.
- Lista final de sabores, formatos, tamaños y **precios**.
- Ingredientes exactos y alérgenos de cada variante; días de conservación.
- Estatus legal del vendedor (auto-entrepreneur / micro-entreprise, SIRET) para las mentions légales.
- ¿Se quiere Instagram/Facebook enlazados? ¿Analítica (Plausible/GA4) desde el día uno?
- ¿Versión multilingüe (ES/EN) ahora o más adelante?

---
> Source: [sergiopo1369-spec/oussap-maison](https://github.com/sergiopo1369-spec/oussap-maison) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
