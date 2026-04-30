## sisgemec

> This document incorporates directives from:

# ui.mdc

```markdown
# UI Rules

This document incorporates directives from:
- **App Flow Document**: Clarifies user journeys (login, create ticket, list tickets, reporting/export).
- **Tech Stack & Frontend Guidelines**: React + TypeScript, TailwindCSS, and shadcn/ui aligned with OpenAPI/Supabase.
- **System Prompts for Cursor Context & Memory Bank**: Structured references for UI components, states, and contracts (`cursor-context.md`, `prompts-library.md`, `development-log.md`).
- **Cursorrules**: Consistent, modular design and normalized error handling.

---

## Framework & Library Guidelines

### Shadcn/ui
- Use **shadcn/ui** as the default design system (ref: *Frontend Guidelines*).
- **No direct overrides**: no editar `components/ui`. Extender vía wrappers del proyecto (`@components/ui/*`) para mantener capacidad de actualización.

### TailwindCSS
- Seguir el enfoque **utility-first** (ref: *Frontend Guidelines* y *Tech Stack*).
- Clases descriptivas y consistentes; preferir **tokens** (spacing/radius/typography) para consistencia visual y **accesibilidad AA**.

### Next.js + React
- **Next.js**: aplicar SSR/ISR de forma selectiva para páginas de listas/reportes (ref: *App Flow Document*).
- **Vite/SPA**: asegurar transiciones rápidas y **code-splitting** en vistas pesadas.
- Usar hooks de React (`useState`, `useEffect`, `useContext`) y **Context API** solo para estado realmente global (auth/sesión, toasts); mantener el resto **local por feature**.

---

## Component Guidelines

### Estructura & Organización de Archivos
- Componentes reutilizables en `src/components/`.
- UI por dominio en `src/features/<domain>/components/`.  
  *(ref: *Implementation Plan* & File Structure)*
- Cada componente debe incluir **docstrings/comentarios** concisos: props, estados y expectativas de accesibilidad (ref: *Cursorrules*).

### Responsivo & Accesible
- Diseño **mobile-first**; patrones `grid/flex` responsivos (ref: *Frontend Guidelines*).
- Garantizar **ARIA**, navegación por **teclado**, **focus management**, y **contraste** WCAG **AA**.
- Proveer estados de **loading / empty / error / success** en todas las vistas de datos.

---

## Performance Optimization
- Usar `React.memo()` y `useCallback()`/`useMemo()` en listas grandes o renders costosos (ref: *Tech Stack*).
- Preferir **dynamic imports** (lazy loading) para módulos pesados (tablas con virtualización, visores de reportes).

---

## Visual Consistency & Reporting Enhancements

- Confiar en **shadcn/ui** para spacing, tipografía, color y motion; evitar CSS ad hoc salvo que sea necesario.

### Reporting UI (SISGEMEC)
- **Panel Izquierdo (Sidebar): “Records & Filters”**
  - Filtros rápidos: **status**, **assignee**, **equipment** y **saved views**.
  - Incluir CTA **“+ Export”**.
- **Contenido Principal (Panel Derecho):**
  - Tablas/tarjetas paginadas para **Tickets**, **Equipos**, **Historial**.
  - Estados **empty/error** con acciones de recuperación.
- **Acciones Inferiores:**
  - **Generate Report** (server-side) y opciones para **Copy**, **Download current**, **Download all** (**CSV/XLSX/PDF**).

### Chatbot Integration (opcional)
- Superficial un asistente con OpenAI para *troubleshooting/FAQs*.
- Estados claros de **loading/error** y **guardrails** (sin PII en prompts/respuestas).

---

## Memory Bank & Collaboration

### Active Context Updates
- Ante cambios UI significativos, actualizar `cursor-context.md` (componentes, estados, contratos) y ajustar **snippets** en `prompts-library.md` (ref: *Memory Bank equivalent*).

### Code Reviews
- Validar en PRs: estados, accesibilidad, alineación de contratos, y **errores normalizados**.
- Usar checklist **LLM-Sec** cuando haya código UI generado por IA.

---

## Testing

- E2E con **Playwright** o **Cypress**. Flujos mínimos:
  - **login**, **create ticket**, **list/paginate/filter**, **export reports**, y (si aplica) **chatbot interactions**.
- Incluir aserciones de **accesibilidad** y **error boundaries**.

---

```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aosenuma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
