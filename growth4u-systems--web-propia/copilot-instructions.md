## web-propia

> Landing page de Growth4U, especialistas en Growth Marketing para empresas tech B2B y B2C. La web incluye blog, lead magnets, casos de éxito, páginas legales y panel admin.

# CLAUDE.md - Growth4U Landing

## Descripción del Proyecto
Landing page de Growth4U, especialistas en Growth Marketing para empresas tech B2B y B2C. La web incluye blog, lead magnets, casos de éxito, páginas legales y panel admin.

## Stack Técnico (Producción)
- **Framework**: Astro 5 con React islands (`client:only="react"`)
- **Estilos**: Tailwind CSS
- **Base de datos**: Firebase Firestore
- **Hosting**: Netlify
- **Admin**: SPA React embebida en `/admin/`

## Estructura del Proyecto
```
growth4u-landing/
├── astro-app/                  # ✅ PROYECTO PRINCIPAL (producción)
│   ├── src/
│   │   ├── pages/              # Rutas estáticas
│   │   │   ├── index.astro     # Home
│   │   │   ├── blog/           # Blog y posts
│   │   │   ├── recursos/       # Lead magnets
│   │   │   │   ├── [slug].astro          # Dinámico (Firebase)
│   │   │   │   ├── cac-sostenible.astro
│   │   │   │   ├── meseta-de-crecimiento.astro
│   │   │   │   ├── sistema-de-growth.astro
│   │   │   │   ├── david-vs-goliat.astro
│   │   │   │   ├── kit-de-liberacion.astro
│   │   │   │   └── dashboard-de-attribution.astro
│   │   │   └── admin/          # Panel admin (React SPA)
│   │   ├── react/              # Componentes React (islands)
│   │   │   ├── LeadMagnetGate.tsx        # Gate con fetch Firebase
│   │   │   ├── StaticLeadMagnetGate.tsx  # Gate con contenido inline
│   │   │   └── admin/pages/    # Páginas del admin
│   │   ├── layouts/
│   │   └── lib/
│   │       ├── firebase-client.ts  # SDK cliente
│   │       └── firebase-fetch.ts   # REST API (build-time)
│   └── dist/                   # Build output
├── netlify.toml                # Config Netlify (base = "astro-app")
├── next-app/                   # Next.js (en desarrollo, NO en producción)
├── scripts/
│   └── notion_to_blog.py       # Sync Notion → Blog automático
└── src/                        # Vite antiguo (deprecated)
```

## Comandos Útiles
```bash
# Desarrollo
cd astro-app && npm run dev

# Build local (puede fallar con Node v24 — usar Netlify)
cd astro-app && npm run build

# Sync Notion → Blog
python scripts/notion_to_blog.py
```

## Configuración Netlify
- **Base directory**: `astro-app`
- **Build command**: `npm run build`
- **Publish directory**: `dist`
- **Production branch**: `main`

## Colores de Marca
- Primary: `#6351d5` (purple)
- Dark navy: `#032149`
- Cyan: `#45b6f7`
- Teal: `#0faec1`

## Firebase
- **Project ID**: `landing-growth4u`
- **App ID**: `growth4u-public-app`
- **Colecciones**: `blog_posts`, `lead_magnets`, `lead_magnet_leads`, `articles`, `leads`, `feedback`, `casos_de_exito`
- **Path base**: `artifacts/growth4u-public-app/public/data/`

## Integraciones
- **Cloudinary**: cloud `dsc0jsbkz`, upload preset `blog_uploads`
- **Netlify build hook**: `https://api.netlify.com/build_hooks/69738cc3fc679a8f858929cd`
- **Calendly**: `https://calendly.com/growth4u/consulta-estrategica`

## Variables de Entorno
```
NOTION_TOKEN=
NOTION_DB_ID=
ANTHROPIC_API_KEY=
NEXT_PUBLIC_FEEDBACK_WEBHOOK_URL=
```

## URLs Importantes
- Producción: https://growth4u.io
- Blog: https://growth4u.io/blog/
- Recursos: https://growth4u.io/recursos/
- Admin: https://growth4u.io/admin/

---

## Skill: Creador de Blogs GEO

Cuando el usuario pase un borrador, post de LinkedIn o tema de blog, conviértelo
en un artículo completo optimizado para GEO (Generative Engine Optimization),
estructurado para ser citado por ChatGPT, Perplexity y otros LLMs.

### Estructura obligatoria (Markdown)

```markdown
## Respuesta directa
[2-3 frases que responden directamente la pregunta principal]

## [Sección 1 — contexto o problema]
[Párrafos de desarrollo]

## [Sección 2 — desarrollo con subsecciones]
### Punto clave 1
### Punto clave 2
### Punto clave 3

## [Sección 3 — datos / comparación]
| Columna 1 | Columna 2 | Columna 3 |
|-----------|-----------|-----------|
| Dato      | Dato      | Dato      |

## Preguntas frecuentes
**¿Pregunta 1 relevante del tema?**
Respuesta concisa de 2-3 frases.

**¿Pregunta 2 relevante del tema?**
Respuesta concisa de 2-3 frases.

**¿Pregunta 3 relevante del tema?**
Respuesta concisa de 2-3 frases.
```

### Reglas del blog
- **Idioma**: Español siempre
- **Longitud**: 800-1200 palabras
- **Empieza siempre** con `## Respuesta directa` — es lo más importante para GEO
- **Mínimo 1 tabla** de comparación o datos por artículo
- **Mínimo 3 preguntas frecuentes** al final, formuladas como preguntas reales
- **Solo Markdown puro** — sin explicaciones, sin comentarios adicionales
- No uses `#` (H1) dentro del artículo — el título ya lo pone la web
- Negrita para términos clave y datos importantes
- Usa ejemplos reales o números concretos cuando sea posible

### Metadatos a generar junto al artículo
```
Título: [título SEO claro, 50-60 caracteres]
Categoría: Estrategia | Marketing | Growth
Excerpt: [primera frase o resumen, máx. 200 caracteres]
Slug: [kebab-case del título, sin acentos]
```

### Flujo de trabajo
1. Usuario pasa borrador, post LinkedIn o tema
2. Generas el artículo completo en la estructura GEO
3. Devuelves primero los metadatos, luego el Markdown
4. Sin preguntas innecesarias — si el tema está claro, escribe directamente

---
> Source: [Growth4U-systems/Web-Propia](https://github.com/Growth4U-systems/Web-Propia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
