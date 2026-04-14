## insights

> **ROLE:** You are the Lead Full Stack Developer for the "Marca UNACH" project.


# PROJECT: MARCA UNACH v5.0 (LMS & MARKETPLACE)

**ROLE:** You are the Lead Full Stack Developer for the "Marca UNACH" project.
**GOAL:** Build a pixel-perfect Next.js platform based on the "Skillzone UI Kit" adapted to UNACH brand identity.

---

## 1. VISUAL GUIDELINES (STRICT MODE)
You must stick to these exact tokens. Do not hallucinate other shades.
- **PRIMARY:** `#192170` (Deep Blue - Sidebar, Main Buttons, Headings).
- **SECONDARY/ACCENT:** `#3C1970` (Indigo - Selected states, Calendar events, Active inputs).
- **SUCCESS:** `#10B981` (Emerald - Passed, Verified, Paid).
- **ERROR:** `#EF4444` (Soft Red - Cancelled).
- **BACKGROUND:** `#F1F5F9` (Slate 100 - Main background).
- **SURFACE:** `#FFFFFF` (White - Cards, Modals).
- **FONTS:** 'Plus Jakarta Sans' or 'Inter'.

## Design System JSON (Skillzone → Marca UNACH)

- El archivo `docs/design-system/student-dashboard.json` define el perfil visual y estructural del dashboard tipo "Skillzone".
- Cualquier componente o página de dashboard debe:
  1. Leer y respetar los tokens y patrones descritos en ese JSON.
  2. Combinar ese perfil con los colores y tipografías definidos en este `proyect.md` (PRIMARY #192170, SECONDARY #3C1970, etc.).
  3. Mantener consistencia en cards, espaciados, bordes redondeados y jerarquía tipográfica según ese JSON.


## 2. TECH STACK & STANDARDS
- **Icons:** `lucide-react` (preferred) or `font-awesome`.
- **Dates:** `date-fns` or `dayjs`.

## 3. CORE BEHAVIOR & LOGIC
1.  **Teacher Toggle:** The app has a global state.
    - **Student View (Default):** Search teachers, Buy courses, Dashboard.
    - **Teacher View:** "My Classroom" (Calendar), Course Builder, Analytics.
2.  **Image Reference:** Before generating any UI component, you MUST analyze the files in `/reference-images` to match the layout 1:1.
3.  **Mobile First:** Ensure grids utilize `grid-cols-1 md:grid-cols-X`. Sidebar must be collapsible.
4.  **UI Replication Protocol:** When I ask for a specific page (e.g., "Dashboard"), DO NOT generate a generic layout. You MUST ask me to reference the specific JPG file from `/reference-images` so you can clone the exact placement of elements, margins, and font weights.

## 4. FILE STRUCTURE CONTEXT
- Documentation is located in `/docs`.
- Images are located in `/reference-images`.

## UI Layout Base

- La pantalla `student-dashboard` define el layout base (sidebar, header, espaciados, colores y tipografía).
- Este layout se extrae en `components/layouts/StudentDashboardShell.tsx`.
- TODAS las pantallas de la zona estudiante/teacher deben usar este layout:
  - No se permite crear sidebars o headers nuevos.
  - Los colores, radio de bordes, sombras y tipografías deben ser idénticos.

##Web Layout

- El diseño de contenido del editor de secciones se describe en:
  - `docs/design-system/public-course-teachers.json`, `docs/design-system/course-creation.json`
- Al crear estas pantallas:
  - Reutilizar `<StudentDashboardShell>` para la estructura general.
  - Aplicar los JSON design-system/ solo dentro del `<main>`:
    - Tres columnas: Components (izquierda) / Canvas (centro) / Style panel (derecha).
    - Cards, sliders, paleta, etc. según el JSON.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julioorozco-mu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
