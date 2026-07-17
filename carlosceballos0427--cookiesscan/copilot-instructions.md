## cookiesscan

> Aplicación web (en etapa inicial) construida con Astro + React. El propósito del proyecto

# CookiesScan

Aplicación web (en etapa inicial) construida con Astro + React. El propósito del proyecto
es el análisis/escaneo de cookies de sitios web. Por ahora existe una sola página
(`src/pages/index.astro`).

## Stack

- **Astro** + **TypeScript**
- **React** (vía `@astrojs/react`) para componentes interactivos
- **Tailwind CSS v4** (vía `@tailwindcss/vite`)

## Comandos (en WSL/Ubuntu)

```bash
npm run dev       # servidor de desarrollo (puerto 4321)
npm run build     # build de producción a dist/
npm run preview   # previsualizar el build
```

El preview de Claude usa `.claude/launch.json`.

## Estructura

- `src/pages/` — páginas (ruteo por archivos de Astro)
- `src/components/` — componentes (usar `.astro` para estático, `.tsx` para interactivo)
- `src/layouts/` — layouts base
- `src/assets/`, `src/styles/` — recursos y estilos globales
- `public/` — assets estáticos

## Reglas de estilo de código (OBLIGATORIAS)

Todo el código debe quedar documentado en español:

- **Cada componente/función** → comentario de encabezado con qué hace y qué props recibe.
- **Cada bucle** (`.map()`, `for`) → comentario de una línea: qué recorre y para qué.
- **Cada expresión compleja** (regex, encadenamientos, hooks) → comentario explicativo.
- **Bloques de HTML/CSS/JS** → comentario de sección describiendo su propósito.

## Contexto adicional

- Idioma del proyecto: español (respuestas, comentarios y documentación en español).

---
> Source: [carlosceballos0427/cookiesscan](https://github.com/carlosceballos0427/cookiesscan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
