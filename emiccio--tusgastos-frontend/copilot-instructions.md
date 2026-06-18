## tusgastos-frontend

> Esta carpeta contiene el frontend de GestionAndo/TusGastos. Incluye la landing de marketing, los flujos autenticados y la capa cliente que consume la API del backend.

# AGENTS.md — Frontend

## Proposito

Esta carpeta contiene el frontend de GestionAndo/TusGastos. Incluye la landing de marketing, los flujos autenticados y la capa cliente que consume la API del backend.

Stack actual:
- Next.js 15
- React 19
- TypeScript
- Tailwind CSS
- Radix UI / componentes estilo shadcn

## Estructura de trabajo

- `src/app`: rutas App Router, layout global y secciones `(marketing)`, `(auth)` y `(app)`
- `src/components`: componentes de UI, layout, landing y charts
- `src/hooks`: estado compartido y flujos como autenticacion y hogares
- `src/lib`: utilidades, constantes y cliente API
- `src/types`: contratos de tipos del frontend
- `public`: assets estaticos e iconos

`src/lib/api.ts` es la frontera principal con el backend. No conviene cambiar firmas, payloads o manejo de errores ahi sin coordinar con los endpoints correspondientes.

## Comandos utiles

- `npm run dev`: levanta el frontend en desarrollo
- `npm run build`: compila la app de Next
- `npm run start`: sirve la build produccion
- `npm run lint`: corre lint del proyecto

## Reglas para cambios

- `docs/DESIGN.md` es la fuente de verdad para cambios visuales en marketing. Si una decision visual entra en conflicto con otra referencia, manda ese documento.
- Preservar tipografias, paleta, tono y patrones ya establecidos antes de introducir nuevas variantes.
- Reutilizar componentes de `src/components/ui` y patrones existentes antes de crear componentes nuevos.
- Mantener consistencia entre rutas `(marketing)`, `(auth)` y `(app)`; evitar mezclar decisiones visuales o de estado entre contextos distintos sin necesidad real.
- Si se modifica `useAuth`, `useHouseholds` o `src/lib/api.ts`, revisar flujos autenticados completos y no solo la pantalla tocada.
- No mezclar trabajo del producto con contenido de `ai/skills/impeccable` salvo pedido explicito. Ese arbol no forma parte del producto principal.

## Validacion esperada

- Correr `npm run build` si el cambio toca rutas, layout, configuracion o integraciones relevantes.
- Correr `npm run lint` si el cambio toca TypeScript, React, hooks o componentes.
- Verificar las vistas afectadas en desktop y mobile.
- Confirmar que los flujos autenticados sigan funcionando cuando se tocan login, household switching, formularios o consumo de API.

## Alcance y cuidado

- Ignorar artefactos generados o ajenos al cambio como `.next` y `node_modules`.
- Evitar editar `ai/skills/impeccable` salvo que la tarea lo pida de forma explicita.
- Al cambiar copy o UI de marketing, mantener el tono cercano y rioplatense que ya usa el producto.

---
> Source: [emiccio/TusGastos-Frontend](https://github.com/emiccio/TusGastos-Frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
