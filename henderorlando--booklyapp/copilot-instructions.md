## booklyapp

> Reglas y estándares para la creación, actualización y mantenimiento de la documentación en Bookly (Frontend, Backend y Global)


# Estándares de Documentación Bookly

Lineamientos obligatorios para crear, modificar y organizar documentación en Bookly.

## 1. Estructura de Directorios (Single Source of Truth)

### `docs/` (Global)

- `business-requirements/` - PDFs oficiales: requerimientos, HU, flujos, casos de uso, catálogo de errores.
- `api-alignment/` - Políticas de alineación frontend-backend, mapeo de endpoints, auditorías.
  - `inventories/` - Inventario de endpoints por servicio (.md y .csv).
  - `missing-definitions/` - Definiciones faltantes (MD-002 a MD-011).
- `project-management/` - Progreso, seguridad, planes activos (i18n, waitlist).
- `qa/` - Smoke tests, auditorías de funciones, mejoras de dashboard, planes E2E.
- `workflows-guide/` - Guías rápidas y definiciones de workflows de Windsurf.
- `archive/` - Documentación obsoleta (rules-review, workflows-old, fixes anteriores).

### `bookly-mock/docs/` (Backend)

- `adr/` - Architecture Decision Records (ADR-001 a ADR-003).
- `api/` - Estándar de respuestas, Swagger, AsyncAPI, ejemplos de uso.
- `architecture/` - Modelo C4, configuración ESM, MongoDB, RabbitMQ.
- `deployment/` - Guías de despliegue (Local, Docker, GH Actions, Pulumi).
- `development/` - Contribución, debug, ejecución, seeds, guías de uso, template CSV.
- `implementation/` - Idempotencia, logging, cache, WebSocket, integración.
- `operations/` - KPIs operativos.
- `templates/` - Plantillas estándar (requirement, endpoints, seeds, architecture, database, event bus).
- `testing/` - Estado de testing y auditoría de dashboard.
- `archive/` - Planes completados, migraciones, refactorings, verificaciones, rules-review.

### `bookly-mock-frontend/docs/` (Frontend)

- `architecture-and-standards/` - ARCHITECTURE, BEST_PRACTICES, PERFORMANCE, TESTING.
- `api-integration/` - Guías de integración UI-API por servicio (01-06), configuración backend, estructura de respuesta.
- `project-management/` - Backlog técnico (PENDIENTES.md).
- `reports/` - Auditorías UX y accesibilidad.
- `archive/` - Trabajo histórico organizado en subdirectorios:
  - `sprints/`, `fixes/`, `plans/`, `implementations/`, `migrations/`, `refactoring/`, `design-system/`, `translations/`, `sessions/`.

### Regla de Oro

- NO crear documentación técnica del backend en la carpeta global ni en el frontend.
- NO mezclar reportes de progreso con requerimientos de negocio.
- NO crear archivos sueltos en la raíz de ningún directorio `docs/` (solo INDEX o DIRECTORIO).

## 2. Formato y Estilo (Markdown)

1. **Título Principal (H1):** Todo archivo debe comenzar con un único `# Título Descriptivo`.
2. **Jerarquía de Encabezados:** Usar secuencialmente `##`, `###`, `####`. No saltar niveles.
3. **Listas y Espaciado:**
   - Dejar una línea en blanco antes y después de cada encabezado y cada lista o bloque de código.
   - Usar sangría de 2 o 4 espacios para listas anidadas de forma consistente.
   - Usar `-` (dash) para listas desordenadas, nunca `*`.
4. **Enlaces Relativos:** Siempre usar `./ruta/archivo.md` o `../ruta/archivo.md`. NUNCA rutas absolutas.

## 3. Actualización de Índices

Cualquier adición, movimiento o eliminación de un archivo de documentación obliga a actualizar:

- `docs/` → `docs/DIRECTORIO_DOCUMENTACION.md`
- `bookly-mock/docs/` → `bookly-mock/docs/INDEX.md`
- `bookly-mock-frontend/docs/` → `bookly-mock-frontend/docs/INDEX.md`

## 4. Archivo de Documentación Obsoleta (Archive)

**NUNCA ELIMINAR** documentación histórica. Mover a `archive/` correspondiente:

- Planes completados → `archive/plans/`
- Fixes resueltos → `archive/fixes/`
- Migraciones completadas → `archive/migrations/`
- Refactorings terminados → `archive/refactoring/`
- Sprints/fases completadas → `archive/sprints/`
- Implementaciones terminadas → `archive/implementations/`

## 5. Plantillas Estándar

- **Features frontend:** Estructura `Feature: [Nombre]` → Requerimientos → Implementación → Tests.
- **Fixes frontend:** Estructura `Fix: [Nombre]` → Problema → Causa Raíz → Solución → Prevención.
- **Backend:** Usar plantillas de `bookly-mock/docs/templates/` (REQUIREMENT, ENDPOINTS, SEEDS, ARCHITECTURE, DATABASE, EVENT_BUS).

## 6. Lenguaje y Tono

- **Idioma:** Español neutro.
- **Tono:** Técnico, directo y conciso.
- **Formato:** Preferir bullet points, tablas y diagramas Mermaid.js. Evitar textos largos sin estructura.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HenderOrlando) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
