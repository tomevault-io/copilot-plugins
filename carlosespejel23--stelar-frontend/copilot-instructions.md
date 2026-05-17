## stelar-frontend

> Frontend de Stellar, la plataforma SaaS edtech para docentes en México. Construido con React + TypeScript + MUI, basado en un paquete de componentes comercial que incluye dashboard, páginas y componentes pre-construidos.

# CLAUDE.md — Stellar Frontend

## Identidad

Frontend de Stellar, la plataforma SaaS edtech para docentes en México. Construido con React + TypeScript + MUI, basado en un paquete de componentes comercial que incluye dashboard, páginas y componentes pre-construidos.

---

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Framework | React 18+ con TypeScript |
| UI Library | MUI (Material UI) v5/v6 |
| Routing | React Router v6 |
| HTTP Client | Axios (ya configurado en el paquete) |
| State Management | React Context |
| Formularios | React Hook Form + Yup/Zod (para validaciones) |
| Idioma de la UI | Español (México) |

---

## Arquitectura del frontend

### Estructura de carpetas

```
frontend/src/
├── app.tsx                         # Componente principal de la app
├── main.tsx                        # Entry point (ReactDOM.render)
├── global-config.ts                # Configuración global (API URL, etc.)
├── global.css                      # Estilos globales
├── vite-env.d.ts                   # Tipos de Vite
│
├── _mock/                          # Datos mock para desarrollo
│   ├── _user.ts, _overview.ts...   # Mock data por dominio
│   └── index.ts
│
├── assets/                         # Assets estáticos
│   ├── data/                       # Datos estáticos (countries.ts, etc.)
│   ├── icons/                      # Iconos SVG como componentes React
│   └── illustrations/              # Ilustraciones SVG (error pages, onboarding, etc.)
│
├── auth/                           # Módulo de autenticación (contexto, guards, vistas)
│   ├── components/                 # Componentes de UI de auth (form-head, form-divider, etc.)
│   ├── context/
│   │   ├── auth-context.tsx        # Context base de autenticación
│   │   └── jwt/                    # Implementación JWT (provider, actions, utils)
│   │       ├── auth-provider.tsx   # JWT AuthProvider principal
│   │       ├── action.ts           # Acciones de auth (login, register, logout)
│   │       └── utils.ts            # Utilidades JWT (decode, isExpired, etc.)
│   ├── guard/                      # Guards de ruta
│   │   ├── auth-guard.tsx          # Redirige a login si no autenticado
│   │   ├── guest-guard.tsx         # Redirige a dashboard si ya autenticado
│   │   └── role-based-guard.tsx    # Verifica roles/permisos
│   ├── hooks/                      # Hooks de auth
│   │   ├── use-auth-context.ts     # Hook para acceder al AuthContext
│   │   └── use-mocked-user.ts     # Hook de usuario mock (dev)
│   ├── types.ts                    # Tipos de autenticación
│   ├── utils/
│   │   └── error-message.ts        # Formateo de errores de auth
│   └── view/
│       └── jwt/                    # Vistas de auth JWT
│           ├── jwt-sign-in-view.tsx
│           └── jwt-sign-up-view.tsx
│
├── components/                     # Componentes reutilizables del paquete
│   ├── animate/                    # Animaciones (Framer Motion)
│   │   ├── motion-container.tsx, motion-viewport.tsx, motion-lazy.tsx
│   │   ├── scroll-progress/       # Barra de progreso de scroll
│   │   └── variants/              # Variantes de animación (fade, slide, zoom, etc.)
│   ├── custom-popover/             # Popover personalizado
│   ├── file-thumbnail/             # Preview de archivos
│   ├── flag-icon/                  # Iconos de banderas
│   ├── hook-form/                  # Integración React Hook Form + MUI
│   │   ├── form-provider.tsx       # FormProvider wrapper
│   │   ├── rhf-text-field.tsx      # TextField controlado por RHF
│   │   ├── rhf-select.tsx          # Select controlado por RHF
│   │   ├── rhf-checkbox.tsx        # Checkbox controlado por RHF
│   │   ├── rhf-autocomplete.tsx    # Autocomplete controlado por RHF
│   │   ├── rhf-date-picker.tsx     # DatePicker controlado por RHF
│   │   ├── rhf-radio-group.tsx     # RadioGroup controlado por RHF
│   │   ├── rhf-switch.tsx          # Switch controlado por RHF
│   │   ├── rhf-slider.tsx          # Slider controlado por RHF
│   │   ├── rhf-rating.tsx          # Rating controlado por RHF
│   │   └── schema-utils.ts        # Utilidades de validación (Zod)
│   ├── iconify/                    # Sistema de iconos (Iconify)
│   ├── label/                      # Componente Label (badges/chips estilizados)
│   ├── loading-screen/             # Pantallas de carga y splash
│   ├── logo/                       # Logo de la app
│   ├── nav-section/                # Componentes de navegación
│   │   ├── components/             # Subcomponentes (collapse, dropdown, subheader)
│   │   ├── horizontal/             # Navegación horizontal
│   │   ├── mini/                   # Navegación mini/colapsada
│   │   ├── vertical/              # Navegación vertical (sidebar)
│   │   └── styles/                 # Estilos y CSS vars de nav
│   ├── progress-bar/               # Barra de progreso (top of page)
│   ├── scrollbar/                  # Scrollbar personalizado
│   ├── search-not-found/           # Estado vacío de búsqueda
│   ├── settings/                   # Panel de configuración visual
│   │   ├── context/                # Context de settings (tema, layout, etc.)
│   │   └── drawer/                 # Drawer de configuración visual
│   └── svg-color/                  # Wrapper para colorear SVGs
│
├── layouts/                        # Layouts de página
│   ├── auth-split/                 # Layout de auth (pantalla dividida)
│   │   ├── layout.tsx              # Layout principal
│   │   ├── content.tsx             # Contenido del formulario
│   │   └── section.tsx             # Sección decorativa lateral
│   ├── dashboard/                  # Layout del dashboard (sidebar + header)
│   │   ├── layout.tsx              # Layout principal
│   │   ├── content.tsx             # Área de contenido
│   │   ├── nav-vertical.tsx        # Sidebar vertical
│   │   ├── nav-horizontal.tsx      # Nav horizontal (alternativa)
│   │   └── nav-mobile.tsx          # Nav móvil (drawer)
│   ├── simple/                     # Layout simple (sin sidebar)
│   │   └── layout.tsx
│   ├── core/                       # Estructura base de layouts
│   │   ├── header-section.tsx      # Header base
│   │   ├── main-section.tsx        # Main content area
│   │   └── layout-section.tsx      # Layout wrapper
│   ├── components/                 # Componentes compartidos de layouts
│   │   ├── account-button.tsx      # Botón de cuenta
│   │   ├── account-drawer.tsx      # Drawer de cuenta
│   │   ├── notifications-drawer/   # Drawer de notificaciones
│   │   ├── searchbar/              # Barra de búsqueda
│   │   ├── sign-out-button.tsx     # Botón de cerrar sesión
│   │   ├── workspaces-popover.tsx  # Selector de workspace/tenant
│   │   └── ...                     # Otros (language, contacts, settings, etc.)
│   ├── nav-config-dashboard.tsx    # Configuración de ítems del nav del dashboard
│   ├── nav-config-account.tsx      # Configuración de nav de cuenta
│   └── nav-config-workspace.tsx    # Configuración de nav de workspace
│
├── lib/                            # Librerías/integraciones
│   └── axios.ts                    # Instancia de Axios (interceptors, base URL)
│
├── pages/                          # Páginas (una por ruta, lazy-loaded)
│   ├── auth/
│   │   └── jwt/
│   │       ├── sign-in.tsx         # Página de login
│   │       └── sign-up.tsx         # Página de registro
│   ├── dashboard/                  # Páginas del dashboard
│   │   ├── one.tsx ... six.tsx     # Páginas placeholder (por renombrar)
│   └── error/
│       └── 404.tsx                 # Página 404
│
├── routes/                         # Configuración de rutas
│   ├── paths.ts                    # Constantes de rutas (/dashboard, /auth/login, etc.)
│   ├── sections/                   # Definición de rutas por sección
│   │   ├── auth.tsx                # Rutas de auth
│   │   ├── dashboard.tsx           # Rutas del dashboard
│   │   └── index.tsx               # Composición de todas las rutas
│   ├── components/                 # Componentes de routing
│   │   ├── router-link.tsx         # Link wrapper
│   │   └── error-boundary.tsx      # Error boundary para rutas
│   └── hooks/                      # Hooks de routing
│       ├── use-router.ts           # Hook de navegación
│       ├── use-pathname.ts         # Hook de pathname actual
│       ├── use-params.ts           # Hook de parámetros de ruta
│       └── use-search-params.ts    # Hook de query params
│
├── sections/                       # Secciones de contenido (vistas/features)
│   ├── blank/
│   │   └── view.tsx                # Vista en blanco (placeholder)
│   └── error/
│       └── not-found-view.tsx      # Vista de 404
│
├── theme/                          # Sistema de temas MUI
│   ├── index.ts                    # Export principal
│   ├── create-theme.ts             # Creación del tema
│   ├── theme-provider.tsx          # ThemeProvider
│   ├── theme-config.ts             # Configuración del tema
│   ├── theme-overrides.ts          # Overrides globales
│   ├── types.ts                    # Tipos del tema
│   ├── core/                       # Núcleo del tema
│   │   ├── palette.ts              # Paleta de colores
│   │   ├── typography.ts           # Tipografía
│   │   ├── shadows.ts              # Sombras
│   │   ├── custom-shadows.ts       # Sombras personalizadas
│   │   ├── mixins/                 # Mixins reutilizables (background, border, text)
│   │   └── components/             # Overrides de componentes MUI (40+ archivos)
│   │       ├── button.tsx, card.tsx, table.tsx, dialog.tsx, ...
│   │       └── mui-x-data-grid.tsx, mui-x-date-picker.tsx, ...
│   └── with-settings/              # Tema dinámico basado en settings
│       ├── color-presets.ts        # Presets de colores
│       ├── right-to-left.tsx       # Soporte RTL
│       ├── update-components.ts    # Actualización dinámica de componentes
│       └── update-core.ts          # Actualización dinámica del core
│
└── utils/                          # Utilidades
    └── format-time.ts              # Formateo de fechas/tiempo
```

### Convenciones de nombrado de archivos

- **kebab-case** para todos los archivos: `auth-guard.tsx`, `jwt-sign-in-view.tsx`, `format-time.ts`
- **Componentes React**: archivos `.tsx`
- **Utilidades/tipos/constantes**: archivos `.ts`
- Cada carpeta tiene su `index.ts` para re-exports

---

## Principios de desarrollo

### 1. Componentes del paquete primero

SIEMPRE busca primero en el paquete comprado (`.reference/`) antes de crear un componente nuevo. El paquete ya tiene: tablas con paginación, formularios, cards de estadísticas, gráficos, modales, sidebars, layouts, páginas de auth, y mucho más.

**Flujo para construir una vista:**
1. Consulta `COMPONENT_CATALOG.md` para ver qué componentes existen.
2. Busca en `.reference/full-demo/` una vista similar a la que necesitas.
3. Copia la estructura y adáptala (cambia datos mock por llamadas a la API real).
4. Si no existe nada similar, construye usando componentes base de MUI.

### 2. Patrón Page → Section (View)

El paquete Minimal UI separa las páginas en dos capas:
- **`pages/`**: wrappers delgados que solo definen metadata (título) e importan la vista.
- **`sections/`**: contienen la lógica real de la vista (layout, datos, interacción).

```typescript
// pages/dashboard/students.tsx — thin wrapper
import { CONFIG } from 'src/global-config';
import { StudentListView } from 'src/sections/students/view';

const metadata = { title: `Estudiantes | ${CONFIG.appName}` };

export default function Page() {
  return (
    <>
      <title>{metadata.title}</title>
      <StudentListView />
    </>
  );
}

// sections/students/view.tsx — lógica real
import { DashboardContent } from 'src/layouts/dashboard';

export function StudentListView() {
  // Aquí va la lógica: fetch de datos, tabla, filtros, etc.
  return (
    <DashboardContent maxWidth="xl">
      {/* contenido */}
    </DashboardContent>
  );
}
```

### 3. Capa HTTP: `lib/axios.ts`

La instancia de Axios está en `src/lib/axios.ts`. Usa `CONFIG.serverUrl` (variable `VITE_SERVER_URL`).

```typescript
// lib/axios.ts — ya configurado
import axios from 'axios';
import { CONFIG } from 'src/global-config';

const axiosInstance = axios.create({
  baseURL: CONFIG.serverUrl,
  headers: { 'Content-Type': 'application/json' },
});

export default axiosInstance;

// También exporta un fetcher genérico y un objeto `endpoints` con las rutas de la API.
```

Los endpoints se definen en el mismo archivo `lib/axios.ts` en el objeto `endpoints`. Conforme se agreguen módulos de Stellar, se deben agregar aquí:

```typescript
export const endpoints = {
  auth: {
    me: '/api/v1/auth/me',
    signIn: '/api/v1/auth/login',
    signUp: '/api/v1/auth/register',
  },
  students: {
    list: '/api/v1/students',
    details: (id: string) => `/api/v1/students/${id}`,
  },
  // ... agregar por módulo
};
```

### 4. Configuración global: `global-config.ts`

La configuración de la app se centraliza en `src/global-config.ts` con la constante `CONFIG`:

```typescript
CONFIG.appName      // Nombre de la app (para títulos de página)
CONFIG.appVersion   // Versión del package.json
CONFIG.serverUrl    // URL del backend (VITE_SERVER_URL)
CONFIG.assetsDir    // Directorio de assets (VITE_ASSETS_DIR)
CONFIG.auth.method  // Método de auth: 'jwt' (el que usamos)
CONFIG.auth.skip    // Skip auth para desarrollo
CONFIG.auth.redirectPath  // Ruta después de login
```

### 5. Imports con path alias `src/`

El proyecto usa `src/` como alias de importación (configurado en Vite). SIEMPRE usa este alias:

```typescript
// ✅ Correcto
import { DashboardContent } from 'src/layouts/dashboard';
import axiosInstance from 'src/lib/axios';
import { CONFIG } from 'src/global-config';

// ❌ Incorrecto
import { DashboardContent } from '../../layouts/dashboard';
import axiosInstance from '../lib/axios';
```

### 6. TypeScript types

Los tipos se definen donde se usan. Conforme el proyecto crezca, se organizarán en archivos dedicados dentro de cada módulo en `sections/` o en una carpeta `types/` si son compartidos.

Tipos clave que deben reflejar las respuestas del backend:

```typescript
// Respuesta paginada del backend (Spring Boot Page<T>)
export interface PagedResponse<T> {
  content: T[];
  page: number;
  size: number;
  totalElements: number;
  totalPages: number;
  last: boolean;
}

// Respuesta de error del backend
export interface ErrorResponse {
  code: string;
  message: string;       // Ya viene en español del backend
  fieldErrors?: Record<string, string>;
  timestamp: string;
}

export type RiskLevel = 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
```

### 7. Autenticación (módulo `auth/`)

La autenticación ya está estructurada en `src/auth/`:
- **`context/jwt/auth-provider.tsx`**: Provider principal, maneja estado del usuario y tokens.
- **`context/jwt/action.ts`**: Funciones de login, register, logout.
- **`guard/auth-guard.tsx`**: Protege rutas que requieren autenticación.
- **`guard/guest-guard.tsx`**: Protege rutas de auth (redirige si ya logueado).
- **`guard/role-based-guard.tsx`**: Protege por rol/permiso.
- **`hooks/use-auth-context.ts`**: Hook para acceder al estado de auth.
- **`view/jwt/`**: Vistas de sign-in y sign-up.

### 8. Rutas: `routes/paths.ts`

Las constantes de rutas se definen en `src/routes/paths.ts`. Usa este archivo para agregar nuevas rutas:

```typescript
const ROOTS = {
  AUTH: '/auth',
  DASHBOARD: '/dashboard',
};

export const paths = {
  auth: {
    jwt: {
      signIn: `${ROOTS.AUTH}/jwt/sign-in`,
      signUp: `${ROOTS.AUTH}/jwt/sign-up`,
    },
  },
  dashboard: {
    root: ROOTS.DASHBOARD,
    students: `${ROOTS.DASHBOARD}/students`,
    // ... agregar nuevas rutas aquí
  },
};
```

Las definiciones de rutas (lazy loading) están en `src/routes/sections/`.

### 9. Navegación del sidebar: `layouts/nav-config-dashboard.tsx`

Los ítems del menú lateral se configuran en `src/layouts/nav-config-dashboard.tsx` con el array `navData`. Cada ítem tiene `title`, `path`, `icon`, y opcionalmente `children`.

### 10. Idioma de la UI: español

Todos los textos visibles al usuario están en español mexicano. Los labels, placeholders, mensajes, títulos de página, tooltips, y botones deben estar en español.

```typescript
// Ejemplos de texto en la UI
"Iniciar sesión"           // no "Log in"
"Registrar estudiante"     // no "Register student"
"Calificaciones"           // no "Grades"
"Pase de lista"            // no "Attendance"
"Alumnos en riesgo"        // no "At-risk students"
"Promedio ponderado"       // no "Weighted average"
"Guardar cambios"          // no "Save changes"
```

---

## Rutas de la aplicación

```typescript
// Rutas actuales (implementadas)
/auth/jwt/sign-in                   // Login
/auth/jwt/sign-up                   // Registro

/dashboard                          // Dashboard principal (placeholder)
/dashboard/two ... /dashboard/six   // Páginas placeholder

// Rutas objetivo (por implementar bajo /dashboard)
/dashboard/students                         // Lista de estudiantes
/dashboard/students/new                     // Registrar estudiante
/dashboard/students/:id                     // Detalle/dashboard del estudiante
/dashboard/students/:id/edit                // Editar estudiante
/dashboard/groups                           // Lista de grupos
/dashboard/groups/new                       // Crear grupo
/dashboard/groups/:id                       // Detalle de grupo
/dashboard/subjects                         // Lista de materias
/dashboard/subjects/new                     // Crear materia
/dashboard/grades                           // Registro de calificaciones
/dashboard/grades/:groupId/:subjectId       // Registro batch de calificaciones
/dashboard/attendance                       // Pase de lista
/dashboard/attendance/:groupId/:subjectId   // Pase de lista batch (API: /assistance/batch)
/dashboard/attendance/history               // Historial de asistencia (API: /assistance/...)
/dashboard/alerts                           // Alumnos en riesgo
/dashboard/alerts/:id                       // Detalle de alerta
/dashboard/reports                          // Reportes
/dashboard/settings                         // Configuración general del tenant
/dashboard/settings/academic                // Períodos académicos
/dashboard/settings/risk-weights            // Pesos del algoritmo de riesgo
/dashboard/settings/notifications           // Preferencias de notificaciones
/dashboard/settings/users                   // Gestión de usuarios
/dashboard/settings/roles                   // Gestión de roles
```

---

## Vistas prioritarias (orden de implementación)

### Fase 1: Auth y estructura base
1. LoginPage — conectar con `POST /api/v1/auth/login` y `POST /api/v1/auth/validate`
2. RegisterPage — conectar con `POST /api/v1/auth/register`
3. ForgotPasswordPage, ResetPasswordPage, VerifyEmailPage
4. DashboardLayout con sidebar, header, y badge de notificaciones
5. AuthGuard, GuestGuard

### Fase 2: CRUD académico
6. StudentListPage — tabla paginada con búsqueda
7. StudentFormPage — crear/editar con validación
8. GroupListPage, GroupFormPage
9. SubjectListPage, SubjectFormPage
10. Enrollments (dentro de StudentDetailPage o GroupDetailPage)

### Fase 3: Calificaciones y asistencia
11. GradeEntryPage — la vista más importante para el docente: seleccionar grupo + materia + período, ver lista de alumnos, ingresar calificaciones en batch
12. AttendancePage — pase de lista: seleccionar grupo + materia + fecha, marcar presente/ausente/retardo/justificado para cada alumno
13. AttendanceHistoryPage — filtrar por grupo, materia, estudiante, rango de fechas

### Fase 4: Analytics y dashboard
14. DashboardPage — resumen con cards de estadísticas, alertas recientes, gráfico de asistencia
15. StudentDetailPage — dashboard individual con calificaciones, asistencia, nivel de riesgo, gráficos de tendencia
16. AlertListPage — tabla de alumnos en riesgo con filtros por nivel
17. AlertDetailPage — desglose de factores de riesgo con visualización
18. RiskWeightsPage — sliders para ajustar los pesos del algoritmo

### Fase 5: Reportes y configuración
19. ReportsPage — solicitar y descargar reportes
20. TenantSettingsPage, AcademicConfigPage
21. NotificationPrefsPage
22. UsersPage, RolesPage

---

## Cómo usar los componentes del paquete

### Referencia

Los dos proyectos del paquete están en:
- `.reference/full-demo/` — versión completa con todos los módulos demo (ecommerce, jobs, etc.)
- `.reference/starter-kit/` — versión base limpia

### Proceso para reutilizar un componente

1. **Buscar en COMPONENT_CATALOG.md** — verifica si el componente ya está catalogado.
2. **Si no está catalogado**, busca en `.reference/full-demo/src/` por nombre o funcionalidad.
3. **Copiar al proyecto**:
   - Componentes reutilizables → `frontend/src/components/`
   - Vistas/features completas → `frontend/src/sections/{modulo}/`
   - Páginas (thin wrappers) → `frontend/src/pages/dashboard/`
4. **Adaptar**:
   - Reemplaza datos mock/hardcodeados por props o llamadas API.
   - Cambia textos al español.
   - Ajusta al tema de Stellar si es necesario.
   - Usa imports con alias `src/` (no rutas relativas).
5. **NO modifiques nada en .reference/** — es solo lectura.

### Convenciones del paquete que debes respetar

- Usa los componentes MUI del paquete (pueden tener wrappers custom). Revisa si el paquete tiene componentes envolventes antes de usar MUI directamente.
- Respeta el sistema de temas del paquete. Las personalizaciones van en `theme/index.ts`.
- Usa los layouts del paquete como base (sidebar, header, etc.).
- Si el paquete tiene un sistema de notificaciones (toast/snackbar), úsalo en lugar de instalar otro.

---

## Conexión con el backend

### Endpoints del backend (resumen por módulo)

**Auth**: validate (paso 1), login (paso 2 con tenant), register, refresh, switch-tenant, logout, verify-email, forgot-password, reset-password, me, me/permissions, me/tenants.
**Invitations**: external, internal, accept, revoke, list, register.
**Users**: CRUD de usuarios del tenant.
**Tenants**: obtener y actualizar tenant actual.
**Roles**: CRUD de roles + permisos.
**Groups**: CRUD de grupos.
**Subjects**: CRUD de materias.
**Students**: CRUD + enrollments + dashboard individual.
**Grades**: registro individual, batch, consultas por estudiante/materia/grupo, promedio.
**Attendance**: registro individual, batch, historial por estudiante/materia/grupo.
**Alerts**: listar, resumen, descartar, evaluar/re-evaluar.
**Risk weights**: consultar y actualizar pesos.
**Notifications**: listar, marcar como leída, preferencias.
**Reports**: generar, listar, descargar.
**Stats**: dashboard general, estadísticas por grupo/materia.

Para el detalle completo de cada endpoint, consulta `backend/CLAUDE.md` o la documentación OpenAPI en `http://localhost:8080/swagger-ui.html`.

---

## Reglas para el asistente

- **Lee COMPONENT_CATALOG.md antes de construir cualquier vista.**
- **Busca en .reference/ antes de crear un componente desde cero.**
- **Genera código TypeScript estricto** — no uses `any`. Define types para todo.
- **Todos los textos de UI en español** — labels, placeholders, títulos, mensajes, botones.
- **Nombres de archivos en kebab-case** — `student-list-view.tsx`, `grade-entry-view.tsx`, etc.
- **Nombres de componentes/funciones en PascalCase** — `StudentListView`, `GradeEntryView`.
- **Un archivo por componente/página** — no metas múltiples componentes en un archivo.
- **Usa imports con alias `src/`** — nunca rutas relativas (`../../`).
- **Sigue el patrón Page → Section**: las páginas en `pages/` son wrappers delgados; la lógica va en `sections/`.
- **Usa los patrones del paquete** — si el paquete tiene una forma de hacer tablas, formularios o layouts, sigue esa forma.
- **Endpoints en `lib/axios.ts`** — agrega nuevos endpoints al objeto `endpoints`.
- **Nuevas rutas**: agrega el path en `routes/paths.ts`, la definición lazy en `routes/sections/dashboard.tsx`, y el ítem de nav en `layouts/nav-config-dashboard.tsx`.
- **Cada página conecta con la API real** — no dejes datos mock. Si el endpoint del backend no existe aún, agrega la ruta en `endpoints` con un TODO.
- **Manejo de errores en cada llamada API** — loading states, error states, empty states.
- **Responsive** — las vistas deben funcionar en desktop y tablet (los docentes usan ambos).

---
> Source: [carlosespejel23/stelar_frontend](https://github.com/carlosespejel23/stelar_frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
