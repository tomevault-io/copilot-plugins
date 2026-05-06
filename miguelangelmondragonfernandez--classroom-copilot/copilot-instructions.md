## 00-global

> Reglas globales del proyecto Classroom Copilot — aplican siempre, sin excepción.


# Classroom Copilot — Reglas Globales

## Fuente de verdad
El archivo `Gemini.md` en la raíz del proyecto contiene la arquitectura, modelos de datos, stack y patrones del proyecto. **Consúltalo antes de implementar cualquier funcionalidad nueva**.

## Stack tecnológico
- **Frontend:** React 19, Vite, PrimeReact, PrimeFlex, React Router DOM v7, Firebase (auth), Vanilla CSS
- **Backend:** Node.js, Express, MySQL 8 (`mysql2`), JWT, Zod, `googleapis`
- **Sin Supabase.** Toda la persistencia y lógica de negocio va exclusivamente al backend Express + MySQL.

## Manejo de código
- Usar **`async/await`** siempre. Nunca cadenas `.then().catch()` extensas.
- Variables y funciones en **`camelCase`** en JS/JSX.
- Añadir comentarios `JSDoc` en servicios del backend y servicios de IA del frontend.
- Nunca exponer secretos en código fuente. Leer siempre desde `process.env` o `import.meta.env`.
- Preferir funciones pequeñas con una sola responsabilidad clara.

## Lo que NUNCA se debe hacer
- Escribir lógica de negocio directamente en rutas (`*.routes.js`).
- Hacer `fetch` directamente desde componentes React al backend.
- Alterar la base de datos sin un archivo de migración en `backend/database/migrations/`.
- Subir al repositorio archivos `.env` o credenciales reales.

---
> Source: [MiguelAngelMondragonFernandez/Classroom-copilot](https://github.com/MiguelAngelMondragonFernandez/Classroom-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
