## asoban-lector-notis

> description: Despliegue: Fly.io y Sprites API

---
description: Despliegue: Fly.io y Sprites API
globs: **/fly.toml,**/Dockerfile,docs/**/*.md,docker-compose.yml
alwaysApply: false
---

# Despliegue

- **Fly.io**: ver `docs/despliegue-fly.md`. Backend soporta `DATABASE_URL` (inyectada por Fly Postgres attach). Frontend usa `REACT_APP_API_URL` en build.
- **Sprites (sprites.dev)**: se usa para despliegues. API en https://sprites.dev/api — provisionar sandboxes Linux (Sprites), ejecutar comandos (Exec), servicios en segundo plano (Services), checkpoints y proxy TCP. Autenticación: `Authorization: Bearer $SPRITES_TOKEN`. No confundir con docker-compose; los despliegues con Sprites se orquestan vía API (crear sprite, exec, servicios, etc.).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/engard0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
