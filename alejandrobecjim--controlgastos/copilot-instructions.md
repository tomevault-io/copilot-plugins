## controlgastos

> Monorepo de una app de control personal de gastos/ingresos (stack MERN). Este archivo aplica a todo el repositorio. Las subcarpetas (`backend/`, `frontend/`) tienen su propio `AGENTS.md` con convenciones específicas del stack que extienden, no reemplazan, este archivo.

# ControlGastos — Guía para Agentes

Monorepo de una app de control personal de gastos/ingresos (stack MERN). Este archivo aplica a todo el repositorio. Las subcarpetas (`backend/`, `frontend/`) tienen su propio `AGENTS.md` con convenciones específicas del stack que extienden, no reemplazan, este archivo.

## Fuente de Verdad

Todas las reglas innegociables viven en [docs/constitution.md](docs/constitution.md). Si alguna instrucción aquí entra en conflicto con la constitución, gana la constitución. No dupliques sus principios acá — este archivo solo agrega guía operativa a nivel de repositorio.

## Arquitectura

- `backend/` — API REST con Express + MongoDB/Mongoose.
- `frontend/` — SPA en React, consume el backend únicamente a través de la API REST (constitución #7).
- No hay carpeta de código compartido entre backend y frontend a menos que un futuro ADR lo justifique (constitución #1, Simplicidad del Stack).

## Convenciones

- **Idioma**: código, identificadores, comentarios y mensajes de commit en inglés. Texto de interfaz orientado al usuario en español (constitución #9).
- **Commits**: formato Conventional Commits (`feat:`, `fix:`, `chore:`, etc.). Nunca agregar trailers de "Co-Authored-By" o de atribución a IA.
- **Trazabilidad**: todo cambio no trivial referencia un ID de spec/tarea (constitución #2).
- **Tamaño**: mantener los cambios pequeños y revertibles de forma independiente (constitución #10).

## Build y Test

- `frontend/` — inicializado con Vite + React + TypeScript (`npm install`, `npm run dev`, `npm run build`, `npm run lint`). Ver comandos completos en [frontend/AGENTS.md](frontend/AGENTS.md).
- `backend/` — aún no inicializado, no existe `package.json`. Al hacer scaffolding, agregar sus comandos de build/test en [backend/AGENTS.md](backend/AGENTS.md).

Mantener los tests automatizados obligatorios para toda lógica de negocio (constitución #4).

---
> Source: [AlejandroBecJim/controlgastos](https://github.com/AlejandroBecJim/controlgastos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
