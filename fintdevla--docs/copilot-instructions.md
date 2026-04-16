## docs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fint documentation site built with [Mintlify](https://mintlify.com). Fint is a payment and administration platform for educational institutions (schools, universities, academies). Documentation is in Spanish.

## Development Commands

```bash
# Install Mintlify CLI globally
npm i -g mintlify

# Start local development server
mintlify dev

# Re-install dependencies (if dev server fails)
mintlify install

# Regenerate API reference from OpenAPI spec
npx @mintlify/scraping@latest openapi-file https://dev.api.fint.app/api/v1/api-docs -o api-reference
```

## Architecture

### Tabs Structure
The docs are organized in 3 tabs (defined in `mint.json`):

| Tab | Path | Purpose |
|-----|------|---------|
| Documentación | `docs/` | User guides for platform features |
| Desarrolladores | `developers/` | Technical docs for API integrations |
| API Reference | `api-reference/` | Auto-generated endpoint documentation |

### Content Structure

```
docs/
├── introduction.mdx          # Landing page for users
└── guides/
    ├── dashboard.mdx         # Panel de Control
    ├── clientes.mdx          # Gestión de clientes
    ├── portal-contacto.mdx   # Portal de autoservicio de contacto
    ├── items-descuentos.mdx  # Configuración de cuotas
    ├── solicitudes-pago.mdx  # Liquidaciones/facturas
    ├── pagos.mdx             # Historial de pagos
    ├── links-pago.mdx        # Venta online
    ├── suscripciones.mdx     # Pagos recurrentes automáticos
    ├── eventos.mdx           # Tickets para eventos
    ├── mensajeria.mdx        # Comunicaciones
    └── configuracion.mdx     # Settings

developers/
├── introduction.mdx          # Getting started for devs
├── contact/                  # Contact model documentation
└── webhooks/                 # Webhook integration guide

api-reference/                # Auto-generated from OpenAPI
```

### Configuration
- `mint.json` - Navigation, tabs, colors, OpenAPI URL
- OpenAPI spec: `https://dev.api.fint.app/api/v1/api-docs`

### Content Format
All pages use MDX with Mintlify components:
- `<Card>`, `<CardGroup>` - Navigation cards
- `<Steps>`, `<Step>` - Step-by-step guides
- `<Info>`, `<Warning>` - Callouts
- `<CodeGroup>` - Multi-language code examples
- `<iframe>` - Loom video embeds (each guide has a tutorial video)
- `mermaid` - Diagrams for state machines, flows and transitions (especially useful in developer docs)

### Writing Style
The documentation is written for someone who is new to the platform and possibly evaluating it. Follow these principles:

- **Vary the components**: Don't use the same component (especially tables) over and over. Mix `<Steps>` for workflows, `<Cards>` for feature highlights, `<Accordion>` for detailed/advanced config, bullet lists for quick references, and prose for explanations.
- **Tables only when truly tabular**: Use tables only for real tabular data (listado columns, comparisons). Never use a table just to list field names and descriptions — prefer bullet lists or prose instead.
- **Friendly and guiding tone**: Write like you're walking someone through the platform, not writing a technical spec. Use "vos" form (Argentine Spanish).
- **Think like a new user**: Every section should be easy to scan. If it feels dense or boring to read, restructure it.
- **No unnecessary images**: Only include images when they add real value (eg: statistics dashboards, complex UIs). Obvious UI elements don't need screenshots.

### Conventions
- Each guide includes a Loom video tutorial at the top
- "Próximos Pasos" section at the bottom links to related guides using `<CardGroup cols={3}>`
- Links use absolute paths: `/docs/guides/clientes` (not relative)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fintdevla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
