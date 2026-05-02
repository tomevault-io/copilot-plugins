## autocontrolapp

> - **Tests:** `npm test` (Usa el runner nativo de Node `node --test`, NO Vitest/Jest).

# Guía para Agentes - Auto Gasto PRO

## Comandos Críticos
- **Tests:** `npm test` (Usa el runner nativo de Node `node --test`, NO Vitest/Jest).
- **Test Individual:** `node --test path/to/file.test.js`.
- **Despliegue:** `npm run deploy` (Limpia, buildea y sube a `gh-pages` sobre la carpeta `dist`).
- **Validación:** `npm run lint` antes de cualquier commit (estricto: 0 warnings).

## Arquitectura y Convenciones
- **Stack:** Vite + React (Hooks/Context) + Firebase (Auth/Firestore).
- **Estructura de Carpetas:**
  - `src/features/`: Componentes lógicos de negocio (ej. `ExpenseForm`, `Reports`).
  - `src/pages/`: Vistas de alto nivel conectadas al router.
  - `src/context/`: Estado global (Auth, Gastos).
- **Tests:** Se ubican junto al archivo que testean (ej. `src/utils/dateUtils.js` -> `src/utils/dateUtils.test.js`).
- **Imports:** Usa módulos de JS (`type: module` en `package.json`).

## Gotchas y Configuración
- **Firebase:** La config está en `src/config/firebase.js`. Requiere variables de entorno de Firebase para funcionar localmente.
- **Gráficos:** Usa `recharts`. Cualquier cambio en reportes debe verificar compatibilidad con este stack.
- **Estilos:** CSS puro modularizado por componente (archivos `.css` junto a `.jsx`).

## Flujo de Trabajo
1. Investigar lógica en `src/features` antes de tocar `src/pages`.
2. Verificar utilitarios en `src/utils` para evitar duplicar lógica de fechas o monedas.
3. Correr `npm test` para asegurar que los cambios no rompan cálculos de combustible o fechas.

---
> Source: [wally720/autocontrolapp](https://github.com/wally720/autocontrolapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
