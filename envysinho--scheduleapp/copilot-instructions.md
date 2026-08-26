## scheduleapp

> Este archivo define cómo un agente debe operar en el proyecto `Schedule` para reducir el uso de tokens y ser eficiente en respuestas.

# AGENTS

## Objetivo
Este archivo define cómo un agente debe operar en el proyecto `Schedule` para reducir el uso de tokens y ser eficiente en respuestas.

## Contexto del proyecto
- `backend/`: Spring Boot con Maven, API Java en `src/main/java/com/example/schedule`.
- `frontend/`: React + Vite, código en `src/` y configuración en `package.json`.
- `desktop/`: Electron wrapper para el frontend.

## Reglas para el agente
1. Responder siempre en español, excepto cuando se citen nombres de archivos o comandos.
2. Priorizar respuestas breves y directas.
3. Para cambios de código, mostrar solo los archivos editados, los fragmentos mínimos y un resumen corto.
4. Evitar explicaciones largas si el usuario pide una acción concreta.
5. Cuando sea posible, usar instrucciones de patch o editar solo bloques específicos.

## Token savings
Ver guía completa: [`docs/ECONOMIA-TOKENS.md`](docs/ECONOMIA-TOKENS.md).

Resumen: no repetir contexto, grep antes de leer, patches pequeños, respuestas breves, sin builds/deps sin permiso.

## Flujo recomendado
- Leer y entender el archivo actual antes de modificarlo.
- Buscar archivos relevantes con `grep` o `find` antes de editar.
- Usar `replace_string_in_file` o `multi_replace_string_in_file` en vez de reescribir archivos completos.

## Avisos
- No crear dependencias nuevas sin permiso explícito.
- No ejecutar `npm install`, `mvn` o comandos de build sin que el usuario lo pida.
- Corregir solo el área concreta que el usuario solicita.

---
> Source: [envysinho/scheduleapp](https://github.com/envysinho/scheduleapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
