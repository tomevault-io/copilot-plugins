## portfolio

> - **Usar sintaxis moderna de control flow**: `@for`, `@if`, `@switch`, `@else`

# Carlos Portfolio - Instrucciones para Claude

## Sintaxis Angular (17+)

- **Usar sintaxis moderna de control flow**: `@for`, `@if`, `@switch`, `@else`
- **NO usar directivas deprecadas**: `*ngFor`, `*ngIf`, `*ngSwitch` (deprecadas desde Angular 17)
- La sintaxis `@` no requiere importar `CommonModule`
- Ejemplo: `@for (item of items; track item.id) { }` en lugar de `*ngFor="let item of items"`

## Estructura del Proyecto

- Angular 21.2.0 standalone components
- PrimeNG 21.1.3 con PrimeIcons 7.0.0 y @primeuix/themes 2.0.3
- Tema personalizado en `src/app/mytheme.ts` basado en Aura de PrimeNG
- Sin archivos de test (.spec.ts) - configurado en angular.json
- Organización por features
- **SIEMPRE usar componentes de PrimeNG** para mantener coherencia en la UI
- **i18n con ngx-translate**: Archivos de traducción en `public/assets/i18n/`

## Internacionalización (i18n)

- Usa **ngx-translate** para traducciones dinámicas
- Idiomas soportados: Inglés (en) y Español (es)
- Idioma por defecto: Inglés (en)
- Archivos de traducción: `public/assets/i18n/en.json` y `es.json`
- **Todos los textos visibles deben usar el pipe translate**: `{{ 'key' | translate }}`
- Selector de idioma en el topbar (icono globe, alterna EN/ES)
- **Al añadir nuevo texto**: Agregar claves en ambos archivos JSON

## Estilos y CSS

### CSS Framework

- **NO usar Tailwind o Bootstrap** - No están instaladas
- **Solo CSS/SCSS puro**
- **NO usar clases utilitarias** tipo Tailwind (flex, items-center, gap-2, etc.)

### Gestión de Temas

- Tema personalizado: `src/app/mytheme.ts` (basado en Aura de PrimeNG)
- **Modo oscuro forzado**: La aplicación usa siempre modo oscuro (clase `dark-mode` en HTML)
- **Paleta de colores personalizada:**
  - Color primario (contraste/títulos): `#34d399` → `var(--p-primary-color)`
  - Fondo principal: `#1c1c1c` → `var(--p-surface-900)`
  - Cuadros de texto (timeline, about, work): `#FBFBFF` → `var(--p-surface-0)`
  - Texto sobre blanco: `var(--p-surface-900)`
  - Texto secundario sobre blanco: `var(--p-surface-500)`
- **IMPORTANTE: Siempre usar variables CSS del tema (`var(--p-*)`) en lugar de valores hardcodeados**
- **Estilos globales/genéricos → `src/styles.scss`**
  - Clases reutilizables: `.slide`, `.content-card`, `.section-title`
  - Variables CSS personalizadas (spacing, breakpoints)
- **Estilos específicos del componente → archivo .scss del componente**
  - Solo layouts y comportamientos únicos del componente

### Tipografías

- Body: **Roboto** (importada desde `/assets/fonts/`)
- Headings: **Space Grotesk** (importada desde `/assets/fonts/`)

### Diseño Responsive

- **SIEMPRE considerar mobile-first y responsive**
- Evitar tamaños fijos en píxeles cuando sea posible
- Usar unidades relativas (rem, em, %, vh/vw, calc())
- Usar variables CSS para valores reutilizables
- Considerar media queries para diferentes breakpoints

## Deploy

- Build output: carpeta `docs/` (para GitHub Pages)
- `baseHref`: `/portfolio/`
- El `TranslateLoader` carga i18n desde `/assets/i18n/` — los archivos deben estar en `public/assets/i18n/`
- Comando de build: `ng build`

## Rutas

- Única ruta implementada: `/home` → `HomePage`
- **Rutas pendientes de implementar** (están en la navbar pero redirigen a home):
  - `/portfolio`
  - `/hobbies`
  - `/bio`
  - `/contact`
- Al crear una nueva página: añadir componente en su carpeta feature y registrar la ruta en `src/app/app.routes.ts`

## Especificaciones del Proyecto

### Navbar (Topbar)

- Posición fija en la parte superior
- Ancho completo, sin bordes ni border-radius
- Altura: variable CSS `--navbar-height` (4rem desktop, 3.5rem mobile)
- Items del menú: Home, Portfolio, Hobbies, Bio, Contact, Download CV
- Mobile: ocultar labels de botones, mostrar solo iconos

### Home Page

- Layout basado en slides (secciones de viewport completo)
- Cada slide: `calc(100vh - var(--navbar-height))`
- Primer slide: hero section con nombre y título, centrado
- Botones sociales: fijos en bottom-right (GitHub, Itch.io, LinkedIn, Mail)

### Footer

- Siempre visible en todas las páginas (definido en app.html)
- Diseño en grid responsive (3 columnas desktop, 1 columna mobile)
- Secciones: Información personal, Quick Links, Redes sociales
- Copyright con año dinámico
- Border-top para separación visual

## Comunicación

- Usuario prefiere comunicación en español
- El contenido de la aplicación puede estar en inglés cuando sea apropiado

## Preferencias de Desarrollo

- Código limpio y mantenible
- Minimizar archivos innecesarios
- Organización de componentes por feature

---
> Source: [CespedDev/portfolio](https://github.com/CespedDev/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
