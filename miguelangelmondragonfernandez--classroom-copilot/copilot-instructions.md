## 02-frontend

> Reglas para el frontend React/Vite en /app/src — componentes, contextos y servicios.


# Frontend — Reglas de Arquitectura React

## Servicios — Centralización de HTTP
- Toda llamada al backend debe ir a través de los servicios en `app/src/services/`.
  - `api.js` — cliente base para el backend Express (usa `get`, `post`, `patch`, `del`).
  - `aiPlannerApi.js` — llamadas a la IA de planeación.
  - `activityGeneratorApi.js` — generación de actividades evaluables.
  - `gasApi.js` — llamadas a Google Apps Script (legacy, a deprecar gradualmente).
  - `googlePicker.js` — encapsula el Google Picker para Drive.
- **Un componente nunca hace `fetch` directamente.** Siempre delega a un servicio.

```jsx
// ✅ Correcto
import { post } from '../services/api';
const data = await post('/planning/ciclos', payload);

// ❌ Incorrecto
const res = await fetch('http://localhost:3001/planning/ciclos', { ... });
```

## Contextos
- `AuthContext` — datos de sesión y perfil del usuario autenticado. No agregar estado ajeno a auth.
- `CourseContext` — datos del curso activo. No agregar estado ajeno al curso.
- Si un módulo (ej. Planeación, Evaluaciones) necesita estado compartido entre componentes, crear su propio contexto cerrado (ej. `PlanningContext`) en vez de polucionar el contexto global.

## Componentes y UI
- Usar **PrimeReact** para todos los elementos interactivos (Botones, Dialogs, Toasts, DataTable, etc.).
- Usar **SweetAlert2** solo para confirmaciones de acciones destructivas (eliminar, sobreescribir).
- Usar **PrimeFlex** para los layouts (grid, flex utilities).
- No añadir librerías de UI adicionales sin consenso.

## Estilos
- Vanilla CSS global en `app/src/assets/index.css`.
- No usar estilos inline en componentes salvo posicionamiento dinámico calculado en JS.
- No agregar Tailwind CSS.

## Estructura de Carpetas de Módulos
Cada módulo visual sigue esta estructura dentro de su carpeta:
```
/planeacion
  ├── components/      # Componentes locales del módulo
  ├── PlaneacionPage.jsx  # Página principal
  └── (context opcional)
```

## Autenticación
- `firebase` se usa únicamente para el flujo de autenticación de Google OAuth.
- El token de sesión del backend (JWT) se guarda en `localStorage` con clave `session_token`.
- Nunca almacenar en `localStorage` ni `sessionStorage` datos sensibles más allá del JWT.

---
> Source: [MiguelAngelMondragonFernandez/Classroom-copilot](https://github.com/MiguelAngelMondragonFernandez/Classroom-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
