## domu-frontend

> Evitar respuestas del tipo "voy a ubicar..." y comenzar directamente en los archivos correctos del proyecto.

# AGENTS.md

## Objetivo
Evitar respuestas del tipo "voy a ubicar..." y comenzar directamente en los archivos correctos del proyecto.

## Reglas para la IA
1. Si el tema es "Crear mi comunidad" o "carga de archivos", no hagas exploración inicial: usa el mapa de este archivo.
2. Antes de cambiar código, confirma en una línea qué archivo/función vas a editar.
3. Después de cambiar código, reporta: archivo, función, endpoint y validaciones tocadas.
4. Prohibido usar emojis en UI o contenido. Usar solo iconografía SVG (componente `Icon` o archivos `.svg`).

## Flujo "Crear mi comunidad" (fuente de verdad)
- UI, estado, validación y submit: `src/pages/Home.jsx`
- Estilos del modal y formulario: `src/pages/Home.scss`
- Mapa/selección de ubicación: `src/components/LocationPicker.jsx`
- Registro local y estadísticas de comunidades: `src/services/communityMaps.js`
- Envío al backend: `src/services/api.js` en `buildings.createRequest`
- Endpoint backend: `POST /buildings/requests`

### Puntos exactos dentro de `src/pages/Home.jsx`
- Apertura del flujo: `handleCreateCommunity` y botones "Crear mi comunidad".
- Modal de 3 pasos: render condicionado por `showCommunityModal` y estado `step`.
- Carga de archivo de acreditación: `handleProofFile`.
- Envío de solicitud: `handleCommunitySubmit`.
- Persistencia local: `COMMUNITY_FORM_STORAGE_KEY` y `COMMUNITY_DOC_NAME_KEY`.

## Carga de archivos por módulo
- Solicitud de comunidad (PDF/JPG/JPEG/PNG, máx. 50MB):
  - UI: `src/pages/Home.jsx` (`handleProofFile`)
  - API: `src/services/api.js` -> `buildings.createRequest` (FormData campo `document`)
- Biblioteca (solo PDF, máx. 30MB):
  - UI: `src/pages/ResidentLibrary.jsx` (`handleFileChange`, `handleUpload`)
  - API: `src/services/api.js` -> `library.upload` (FormData campo `file`)
- Perfil (avatar y avatar de privacidad):
  - UI: `src/pages/ResidentProfile.jsx` (`handleAvatarUpload`)
  - API: `src/services/api.js` -> `users.updateAvatar` y `users.updatePrivacyAvatar` (FormData campo `avatar`)
- Marketplace (múltiples imágenes):
  - UI: `src/pages/ResidentMarketplaceCreate.jsx` (`handleImageChange`, submit)
  - API: `src/services/api.js` -> `market.createItem` y `market.updateItem` (FormData campo `images`)
- Boletas de cargos (admin):
  - UI: `src/pages/AdminCommonExpenses.jsx` (`handleUploadReceipt`)
  - API: `src/services/api.js` -> `finance.uploadChargeReceipt` (FormData campo `document`)

## Rutas clave relacionadas
- Definiciones centralizadas: `src/constants/routes.js`
- Declaración de rutas: `src/App.jsx`
- Home pública: `/`
- Biblioteca: `/resident/biblioteca`
- Perfil: `/resident/profile`
- Marketplace crear: `/resident/marketplace/nuevo`
- Gastos comunes admin: `/dashboard/charges`

## Comportamiento esperado de la IA en este repo
1. No usar frases de exploración como "voy a ubicar...".
2. Ir directo al archivo indicado por este mapa.
3. Si hay ambigüedad entre varios uploads, preguntar solo: "¿Comunidad, Biblioteca, Perfil, Marketplace o Boletas?".

---
> Source: [Wasoner/domu-frontend](https://github.com/Wasoner/domu-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
