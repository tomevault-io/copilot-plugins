## webapporganizat

> Guía para mantener la calidad y consistencia del código.

# Estándares de Desarrollo

Guía para mantener la calidad y consistencia del código.

## Separación de Responsabilidades (SoC)
Evitar "God Components". Dividir lógica y vista.

### 1. Custom Hooks (`src/hooks/`)
- Toda la lógica de estado (`useState`, `useEffect`), llamadas a API y handlers debe ir aquí.
- **Ejemplo:** `useTaskDetail.ts` maneja la carga, edición y borrado de tareas.

### 2. Componentes UI (`src/components/`)
- Componentes "tontos" (presentacionales) que reciben props.
- Deben ser reutilizables y pequeños.
- **Modales:** Preferir modales unificados para Crear/Editar (ej. `TagModal`) controlados por props (`initialData`).
- **Listas:** Usar componentes específicos para items (ej. `TaskCard`) y contenedores con espaciado (`space-y-4`).

### 3. Páginas (`src/app/`)
- Actúan como "Controladores".
- Llaman a servicios/hooks para obtener datos.
- Manejan el routing (`useRouter`) y Feature Flags globales.
- **Suspense:** Envolver componentes que usen `useSearchParams` en `<Suspense>` para evitar errores de build en componentes cliente.

### 4. Servicios y API (`src/services/`)
- **Server-side Filtering:** Preferir filtrar, ordenar y paginar en el backend.
- Enviar parámetros de filtro como query params (ej. `?tag_ids=1&sort_by=date`).
- Mantener interfaces de TypeScript (`Task`, `Tag`, `TaskFilters`) sincronizadas con la respuesta del backend.

## Convenciones
- **Archivos:** `PascalCase` para componentes, `camelCase` para hooks/utils.
- **Imports:** Usar alias `@/` para rutas absolutas.
- **Iconos:** Usar `lucide-react`.
- **Estilos:** Tailwind CSS con diseño "soft" (`rounded-3xl`, `bg-gray-50`).
- **Modo Oscuro:** Usar prefijo `dark:` (ej. `bg-white dark:bg-gray-900`). Mantener contraste accesible.
- **Markdown:** Usar `ReactMarkdown` con `remark-gfm` para renderizado seguro.
  - Personalizar componentes (ej. `strong`, `ul`) para soportar estilos Tailwind y Dark Mode.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndresOrtega-tech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
