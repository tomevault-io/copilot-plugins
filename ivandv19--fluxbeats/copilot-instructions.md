## fluxbeats

> Landing page pública para exhibir y vender instrumentales musicales (beats lofi/chillhop). Portafolio profesional con reproductor de audio, formulario de contacto protegido contra spam y soporte multilingüe (es/en).

# AGENTS.md — Fluxbeats Landing Page

## Descripción del Proyecto

Landing page pública para exhibir y vender instrumentales musicales (beats lofi/chillhop). Portafolio profesional con reproductor de audio, formulario de contacto protegido contra spam y soporte multilingüe (es/en).

## Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Framework | Vite 7 + React 19 (SPA) |
| UI | Tailwind CSS 4 |
| Backend API | Hono (Cloudflare Pages Functions) |
| Contacto | Resend + Cloudflare Turnstile |
| Runtime | Bun 1.x |
| Deploy | Cloudflare Pages (Wrangler) |
| Lint/Format | Biome 2 |
| Tests | Vitest + jsdom + @testing-library/react |
| i18n | React Context (es/en) |

## Estructura del Código

```
src/
├── assets/           # SVG estáticos
├── components/       # Componentes React (.tsx)
│   ├── Beats/        # Catálogo de beats + MiniPlayer
│   ├── Contacto/     # Formulario de contacto + validación
│   ├── Footer/       # Footer global
│   ├── Header/       # Header + navegación responsive
│   ├── Hero/         # Hero section
│   ├── Licencias/    # Precios y licencias
│   ├── SobreMi/      # About section
│   ├── Testimonios/  # Reseñas de clientes
│   └── common/       # Componentes reutilizables
├── context/          # React Contexts (Theme, Language)
├── data/             # Datos estáticos (beats)
├── hooks/            # Custom hooks (useAudioPlayer, useScrollAnimation)
├── i18n/             # Traducciones es/en
├── pages/            # Páginas (Home, legal)
├── test/             # Setup de testing
└── utils/            # Utilidades (time formatting)

functions/
└── api/[[route]].ts  # API Hono (contact form endpoint)

public/
├── _headers          # Security & cache headers
├── robots.txt
├── sitemap.xml
├── favicon.png
└── og-image.jpg
```

## Comandos Disponibles

| Comando | Descripción |
|---|---|
| `bun run dev` | Servidor de desarrollo (Vite) |
| `bun run build` | TypeScript check + build de producción |
| `bun run preview` | Preview del build |
| `bun run lint` | Biome lint (read-only) |
| `bun run format` | Biome format (auto-fix) |
| `bun run check` | Biome check + format (auto-fix) |
| `bun run test` | Vitest (run mode) |

## Convenciones de Código

- **Indentación**: Tabs (configurado en Biome)
- **Comillas**: Dobles (`"`) en JS/TS
- **Imports**: Organizados automáticamente por Biome
- **Tipado**: TypeScript estricto (`strict: true`, `noUnusedLocals`, `noUnusedParameters`)
- **Estilos**: Tailwind CSS 4 utility classes
- **Nombres**: camelCase para hooks/utils, PascalCase para componentes

## Variables de Entorno

| Variable | Uso |
|---|---|
| `RESEND_API_KEY` | API key de Resend para envío de emails |
| `CONTACT_EMAIL` | Email destino del formulario de contacto |
| `TURNSTILE_SECRET_KEY` | Secret de Cloudflare Turnstile |
| `VITE_TURNSTILE_SITE_KEY` | Site key de Turnstile (cliente) |

## API Endpoints

- `POST /api/contact` — Enviar formulario de contacto (validado con Zod + Turnstile)
- `GET /api/health` — Health check

## Reglas para el Agente

1. Siempre correr `bun run lint` y `bun run test` después de cambios
2. Mantener i18n consistente (agregar claves en `es.ts` Y `en.ts`)
3. Usar Biome para formato y lint (tabs + double quotes)
4. No agregar emojis en commits ni código (salvo que el usuario lo pida)
5. No alterar la estructura de componentes sin confirmar

---
> Source: [Ivandv19/fluxbeats](https://github.com/Ivandv19/fluxbeats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
