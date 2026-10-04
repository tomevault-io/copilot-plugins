## claude-project-template

> Este es un proyecto fullstack (Java + Angular + Postgres) con un módulo inicial de registro de usuarios.

# Contexto del Proyecto
Este es un proyecto fullstack (Java + Angular + Postgres) con un módulo inicial de registro de usuarios.

## Arquitectura y Tecnologías
- **Backend**: Spring Boot 3, Java 17+, JPA (en la carpeta `/backend`).
- **Frontend**: Angular 17+ con Standalone Components (en la carpeta `/frontend`).
- **Base de Datos**: PostgreSQL 18 ejecutándose vía Docker.

## Reglas de Desarrollo (AI Harness)
1. Escribe código limpio, aplicando principios SOLID.
2. En Java, usa inyección de dependencias por constructor, no uses `@Autowired` en campos.
3. En Angular, prioriza el uso de la nueva sintaxis de control de flujo (`@if`, `@for`).
4. Antes de ejecutar comandos de instalación (npm install o mvn clean install), pídeme confirmación.

## Gestión de Proyecto (Jira)
- El proyecto de Jira asociado a esta carpeta es **"Claude project test"** (key: `SCRUM`), en el sitio `cagiraldo88.atlassian.net`.
- Usa este proyecto como referencia para crear, buscar o actualizar issues relacionados con este código.

## Skills y Commands Disponibles
No intentes adivinar flujos complejos. Si te pido ejecutar una acción, primero lee la especificación en la carpeta correspondiente:

- Si te pido **"Sincronizar Jira"** o **"Iniciar ticket"**, lee la lógica en `.claude/skills/jira-sync.md`.
- Si te pido **"Hacer Code Review"**, aplica los criterios de `.claude/skills/code-review.md`.
- Para levantar infraestructura, inspecciona la carpeta `.claude/commands/`.

---
> Source: [camiloGiraldoR/claude-project-template](https://github.com/camiloGiraldoR/claude-project-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
