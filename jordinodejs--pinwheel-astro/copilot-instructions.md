## pinwheel-astro

> Estas reglas guían a la AI de Cursor en este repositorio `pinwheel-astro` para producir código consistente, seguro y alineado con las mejores prácticas de Astro 5, Tailwind CSS 4 y el stack moderno de desarrollo web. Mantén estas prácticas en cada cambio.

# Reglas de Cursor para Pinwheel Astro

Estas reglas guían a la AI de Cursor en este repositorio `pinwheel-astro` para producir código consistente, seguro y alineado con las mejores prácticas de Astro 5, Tailwind CSS 4 y el stack moderno de desarrollo web. Mantén estas prácticas en cada cambio.

## Stack Tecnológico y Versiones

### Framework Principal
- **Astro**: 5.x (Vite 7) con integraciones `@astrojs/react`, `@astrojs/mdx`, `@astrojs/sitemap` y `astro-auto-import`
- **UI**: Tailwind CSS 4 con plugin `@tailwindcss/vite` y Bootstrap Grid System personalizado
- **Tipado**: TypeScript estricto (`extends: "astro/tsconfigs/strict"`)
- **Contenido**: `astro:content` v5 (Content Layer API) con colecciones en `src/content` y esquemas en `src/content.config.ts`
- **Gestión de paquetes**: PNPM (requerido) - NO uses npm o yarn
- **Fuentes**: `astro-font` para optimización automática de fuentes
- **Analytics**: `@digi4care/astro-google-tagmanager` controlado por `src/config/config.json`

### Arquitectura de Renderizado
- **Hosting**: Sitio completamente estático (SSG) - puede subirse a hosting básico como Hostinger
- **Componentes React**: Se hidratan en cliente con directivas apropiadas (`client:*`)
- **Sin SSR**: No usa `output: 'server'` ni adaptadores de servidor

## Estructura y Convenciones de Archivos

### Organización de Directorios
```
src/
├── pages/           # Rutas de Astro (file-based routing)
├── layouts/         # Layouts base y componentes de layout
│   ├── components/  # Componentes Astro reutilizables
│   ├── partials/    # Header, Footer, CTA
│   └── shortcodes/  # Componentes MDX auto-importados
├── content/         # Colecciones de contenido (blog, careers, etc.)
├── lib/             # Utilidades y helpers
├── styles/          # CSS layers y utilidades personalizadas
└── config/          # Archivos de configuración JSON
```

### Alias de Importación (tsconfig.json)
SIEMPRE usa estos alias en lugar de rutas relativas:
- `@/components/*` → `src/layouts/components/*`
- `@/shortcodes/*` → `src/layouts/shortcodes/*`
- `@/partials/*` → `src/layouts/partials/*`
- `@/*` → `src/*`

### Convención de Nomenclatura
- **Archivos**: PascalCase para componentes (`Banner.astro`), kebab-case para utilidades (`text-converter.ts`)
- **Componentes**: Nombres descriptivos y específicos (`BlogPostCard`, `ServiceFeatures`)
- **Variables/funciones**: camelCase con nombres explícitos, evita abreviaturas
- **Constantes**: UPPER_SNAKE_CASE
- **Contenido**: `-index.md(x)` para páginas índice de colección

## Reglas de Codificación TypeScript

### Tipado Estricto
- **NO uses `any`** - usa tipos específicos o `unknown` si es necesario
- **Props en .astro**: Define `interface Props` y desestructura con valores por defecto
- **Content Collections**: Usa `CollectionEntry<"collection">` para props de entrada
- **Funciones exportadas**: Siempre con tipos de retorno explícitos

```astro
---
interface Props {
  title: string;
  description?: string;
  featured?: boolean;
}

const { title, description = "Default description", featured = false } = Astro.props;
---
```

### Importaciones y Módulos
- **Extensiones de archivo**: Incluye `.js` para archivos TypeScript importados
- **Importaciones**: Agrupa por tipo (tipos, utilidades, componentes)
- **Re-exportaciones**: Evita exportaciones masivas (`export *`)

```ts
// Correcto
import { markdownify, plainify } from "@/lib/utils/textConverter.js";
import type { CollectionEntry } from "astro:content";
```

## Astro 5 - Mejores Prácticas

### Content Collections (Content Layer API)
- **Ubicación**: Define colecciones en `src/content.config.ts`
- **Loaders**: Usa `glob()` y `file()` loaders nativos
- **Schemas**: Siempre define schemas con Zod para validación
- **Consultas**: Usa `getCollection()` y `getEntry()` con tipos apropiados

```ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: "**/*.md", base: "./src/content/blog" }),
  schema: z.object({
    title: z.string(),
    description: z.string().optional(),
    pubDate: z.coerce.date(),
    featured: z.boolean().default(false),
  })
});
```

### Componentes y Layouts
- **Props tipadas**: Siempre usa interfaces TypeScript para props
- **Layout base**: Usa `src/layouts/Base.astro` como wrapper principal
- **Metadatos**: Pasa `title`, `meta_title`, `description`, `image` cuando sea necesario
- **Slots**: Usa slots nombrados para contenido estructurado

### Islas de React
- **Principio**: Astro renderiza HTML por defecto, usa React SOLO para interactividad
- **Directivas de hidratación**:
  - `client:load` - Para interacción inmediata (formularios críticos)
  - `client:idle` - Para componentes diferibles (widgets)
  - `client:visible` - Para contenido que entra en viewport (carruseles)
- **Importaciones**: Asegúrate que componentes React estén en `layouts/function-components/`

```astro
---
import InteractiveForm from "@/function-components/InteractiveForm.jsx";
---

<InteractiveForm client:load />
```

## Tailwind CSS 4 - Estándares

### Metodología CSS
- **Layers**: Respeta la estructura de `main.css` (@layer base, @layer components)
- **Utilidades**: Prioriza clases utilitarias sobre CSS personalizado
- **Componentes**: Usa `@apply` SOLO en `src/styles/components.css`
- **Variables**: Usa CSS custom properties definidas en `tw-theme.js`
- **Grid**: Sistema híbrido Bootstrap Grid + Tailwind Flex

### Patrones de Clases
```astro
<!-- Contenedor estándar -->
<div class="container">
  <div class="row items-center">
    <div class="lg:col-6">
      <!-- Contenido -->
    </div>
  </div>
</div>

<!-- Botones usando clases componente -->
<a class="btn btn-primary" href="/contact">Contact Us</a>
<a class="btn btn-white btn-sm" href="/learn-more">Learn More</a>
```

### Responsividad y Accesibilidad
- **Mobile-first**: Usa breakpoints progresivos (`sm:`, `md:`, `lg:`, `xl:`)
- **Estados focus**: Siempre incluye estados focus visibles
- **Contraste**: Asegura contraste suficiente (AA mínimo)
- **Semántica**: Usa elementos HTML semánticos apropiados

## Gestión de Contenido y Assets

### Assets e Imágenes
- **Astro Assets**: Usa `astro:assets` (`<Image />`) para imágenes procesables
- **Public assets**: Enlaces absolutos desde `/` para archivos en `public/`
- **Optimización**: Siempre define `width`, `height` y `alt` descriptivo
- **Fuentes**: NO añadas manualmente `<link>` a Google Fonts - usa `AstroFont`

```astro
---
import { Image } from "astro:assets";
---

<Image
  src="/images/hero-banner.png"
  alt="Descripción significativa de la imagen"
  width={800}
  height={400}
  loading="lazy"
/>
```

### Content Collections - Patrones
- **Frontmatter**: Mantén coherencia con el schema definido
- **Referencias**: Usa `reference()` para vincular entre colecciones
- **Filtrado**: Ordena manualmente las colecciones (`getCollection` no garantiza orden)

```astro
---
import { getCollection } from 'astro:content';

const posts = (await getCollection('blog'))
  .filter(post => !post.data.draft)
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---
```

## Configuración y Personalización

### Archivos de Configuración
- **Centralizada**: Toda la configuración en `src/config/*.json`
- **Tipado**: Define interfaces TypeScript para objetos de configuración
- **Ambiente**: Usa variables de entorno para datos sensibles
- **Cache**: Reutiliza configuraciones, evita lecturas múltiples

### Utilidades y Helpers
- **Reutilización**: Usa utilidades existentes antes de crear nuevas:
  - `markdownify()` - Convierte markdown a HTML inline
  - `plainify()` - Elimina HTML y entities
  - `humanize()` - Formatea strings para mostrar
  - `slugify()` - Genera slugs URL-safe
- **Ubicación**: Nuevas utilidades en `src/lib/utils/`

## Rendimiento y Optimización

### Estrategias de Carga
- **JavaScript mínimo**: Evita hidratar componentes innecesariamente
- **Lazy loading**: Usa `loading="lazy"` en imágenes
- **Critical CSS**: Mantén CSS crítico inline cuando sea necesario
- **Bundle splitting**: Separa dependencias pesadas del cliente

### SEO y Meta Tags
- **Layout Base**: Usa `Base.astro` para metadatos consistentes
- **Open Graph**: Completa metadatos OG/Twitter vía props del layout
- **Canonical URLs**: Define URLs canónicas para evitar contenido duplicado
- **Structured Data**: Añade JSON-LD cuando sea apropiado

## Workflow y Comandos de Desarrollo

### Scripts Disponibles
- **Desarrollo**: `pnpm dev` (puerto 4321)
- **Build**: `pnpm build` (genera carpeta `dist/` estática)
- **Type-check**: `pnpm check` (verifica errores TypeScript)
- **Formato**: `pnpm format` (Prettier con plugins de Astro y Tailwind)
- **Preview**: `pnpm preview` (servidor para probar build)

### Pre-commit Checklist
1. ✅ Ejecuta `pnpm check` - Sin errores TypeScript
2. ✅ Ejecuta `pnpm build` - Build exitoso
3. ✅ Ejecuta `pnpm format` - Código formateado
4. ✅ Verifica alias de importación válidos
5. ✅ Confirma ausencia de `any` no justificado

## Instrucciones Específicas para Cursor AI

### Comunicación
- **Idioma**: Responde SIEMPRE en español en comentarios y descripciones
- **Commits**: Mensajes de commit en inglés usando Conventional Commits
- **Documentación**: Comenta código complejo en español

### Metodología de Edición
- **Conservación**: NO reformatees código existente más allá del cambio necesario
- **Imports**: Usa alias `@/...` en lugar de rutas relativas
- **Reutilización**: Prefiere editar/extender componentes existentes antes que crear nuevos
- **Consistencia**: Mantén el estilo y convenciones del código circundante

### Patrones de Desarrollo
- **Componentes nuevos**: Sigue estructura existente en `layouts/components/`
- **Estilos**: Añade clases a `src/styles/components.css` si es necesario
- **Utilidades**: Extiende utilidades existentes antes de crear nuevas
- **Testing**: Prueba cambios en diferentes breakpoints y navegadores

## Antipatrones (EVITAR)

### Astro/React
- ❌ Hidratar componentes sin interactividad real
- ❌ Usar React para contenido estático
- ❌ Múltiples `client:load` en la misma página
- ❌ Importar librerías pesadas en el cliente

### TypeScript
- ❌ Usar `any` sin justificación documenta
- ❌ Suprimir errores TypeScript con `@ts-ignore`
- ❌ Props sin tipar en componentes
- ❌ Mutaciones directas de props de solo lectura

### CSS/Tailwind
- ❌ CSS inline para estilos complejos
- ❌ Duplicar utilidades existentes
- ❌ Clases !important innecesarias
- ❌ Estilos que rompen responsive design

### Arquitectura
- ❌ Lógica de negocio en componentes de presentación
- ❌ Rutas relativas largas en lugar de alias
- ❌ Configuración dispersa (usar `config/*.json`)
- ❌ Scripts de terceros fuera del control de configuración

## Referencias y Recursos

### Documentación Oficial
- [Astro 5 Docs](https://docs.astro.build) - Documentación oficial actualizada
- [Tailwind CSS 4](https://tailwindcss.com/docs) - Guía de utilidades y componentes
- [TypeScript Handbook](https://typescriptlang.org/docs) - Referencia completa TS

### Configuraciones del Proyecto
- `src/config/config.json` - Configuración general del sitio
- `src/config/theme.json` - Variables de tema y fuentes
- `src/config/menu.json` - Estructura de navegación
- `src/content.config.ts` - Schemas de Content Collections
- `astro.config.mjs` - Configuración de Astro e integraciones

### Comandos de Referencia Rápida
```bash
# Desarrollo
pnpm dev

# Verificaciones pre-commit
pnpm check && pnpm build && pnpm format

# Migración desde npm/yarn
./migrate-to-pnpm.sh  # Linux/Mac
./migrate-to-pnpm.ps1 # Windows
```

---

**Nota**: Mantén estas reglas como fuente de verdad para todas las contribuciones y automatizaciones de la AI. Actualiza este archivo cuando el stack tecnológico o las convenciones del proyecto cambien significativamente.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JordiNodeJS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
