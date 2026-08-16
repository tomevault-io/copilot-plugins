## vca-possystem-app

> Este archivo define cómo debe trabajar Claude Code en este repositorio. Léelo por completo antes de hacer cambios.

# CLAUDE.md — VCA POS System

Este archivo define cómo debe trabajar Claude Code en este repositorio. Léelo por completo antes de hacer cambios.

## Contexto del proyecto

Sistema de punto de venta (POS) compuesto por:
- `apps/backend` — API
- `apps/frontend` — Aplicación web
- `keycloak/` — Configuración de Keycloak (IAM / seguridad)
- `infra/` — Docker Compose, configuración de despliegue
- `docs/` — Documentación del proyecto

La seguridad (autenticación/autorización) se maneja con **Keycloak**, no se implementa login propio en el backend ni en el frontend.

---

## Estructura de ramas (obligatorio respetar)

Este repo usa **3 ramas principales**, cada una mapeada a un ambiente:

| Rama | Ambiente | Notas |
|------|----------|-------|
| `dev` | Desarrollo | Se puede romper temporalmente. Todo feature branch nace y regresa aquí. |
| `staging` | Pre-producción | Espejo casi exacto de prod. Se usa para QA antes de liberar. |
| `main` | Producción | Protegida. Nunca push directo. Solo vía PR desde `staging`. |

**Reglas para Claude Code:**
- Nunca hacer push directo a `main` o `staging`. Los cambios llegan ahí solo por PR desde la rama anterior en el flujo (`dev` → `staging` → `main`).
- Los feature branches se crean desde `dev` con el patrón `feature/nombre-corto` o `fix/nombre-corto`, y su PR apunta de vuelta a `dev`.
- Si se pide "desplegar a producción" o similar, recordar que `main` requiere PR + review, no es un push automático.

---

## Variables de entorno

- Nunca commitear archivos `.env`, `.env.dev`, `.env.staging`, `.env.prod`.
- El único archivo de entorno que se commitea es `.env.example`, con valores placeholder (`changeme`) y comentarios en inglés explicando cada variable.
- Si se agrega una variable de entorno nueva a cualquier `.env.*`, se debe reflejar también en `.env.example` en el mismo commit/PR.
- El patrón en `.gitignore` para esto es:
  ```
  .env*
  !.env.example
  ```

---

## Keycloak

### Estado actual (importante)
- **No se está usando import automático de realms todavía** (`--import-realm`). La configuración de realms, clients y roles se hace manualmente desde la consola admin de Keycloak en cada ambiente.
- No crear archivos en `keycloak/realm-export/` a menos que se pida explícitamente activar ese flujo.
- No agregar `--import-realm` ni volúmenes de import al `docker-compose.yml` sin que se solicite.

### Estructura actual de `keycloak/`
Por ahora esta carpeta solo debe contener documentación (`README.md`). No crear subcarpetas vacías (`themes/`, `providers/`, `scripts/`, `realm-export/`) de forma anticipada — se crean solo cuando haya contenido real que poner ahí.

### Comandos de arranque por ambiente
- `dev` → `KC_COMMAND=start-dev` (modo relajado, HTTP permitido, solo local)
- `staging` / `prod` → `KC_COMMAND=start --optimized` (modo producción real)

Nunca cambiar `start --optimized` por `start-dev` en staging o prod, ni al revés en dev, salvo pedido explícito.

### Base de datos de Keycloak
- La base de datos ya existe (gestionada por separado, no por Keycloak).
- Keycloak solo se conecta vía `KC_DB_URL` / variables `KC_DB_*`. No crear un servicio de base de datos dentro de `docker-compose.yml` para Keycloak salvo que se pida explícitamente.
- Cada ambiente tiene su propia base de datos aislada (`keycloak_dev`, `keycloak_staging`, `keycloak_prod`). Nunca compartir la misma base de datos ni el mismo realm entre ambientes.

---

## Docker Compose

- El archivo principal vive en `infra/docker-compose.yml` y debe funcionar para los 3 ambientes usando `--env-file`:
  ```bash
  docker compose --env-file .env.dev up -d
  docker compose --env-file .env.staging up -d
  docker compose --env-file .env.prod up -d
  ```
- No hardcodear valores de host, puertos, credenciales o nombres de contenedor directamente en el YAML — todo debe venir de variables de entorno (`${VARIABLE}`).
- No agregar `docker-compose.prod.yml` u otros archivos separados por ambiente salvo que se solicite; el patrón actual es un solo compose + distintos `.env`.

---

## Seguridad — reglas generales

- El backend nunca debe implementar su propio manejo de contraseñas o sesiones — toda autenticación pasa por Keycloak (validación de JWT).
- El client `vca-pos-backend` es **confidential** (usa client secret, corre en servidor).
- El client `vca-pos-frontend` es **public + PKCE** — nunca agregarle client secret, ya que el código corre en el navegador.
- No guardar tokens de acceso/refresh en `localStorage` del frontend salvo que se indique explícitamente lo contrario. Preferir cookies `httpOnly` o patrón BFF si se implementa.
- Nunca escribir secretos, contraseñas o client secrets directamente en código, YAML versionado, o archivos JSON de realm que se vayan a commitear.

---

## Estilo de trabajo esperado

- Antes de crear carpetas o archivos nuevos, revisar si ya existe una convención establecida en este documento o en `docs/`.
- Si una tarea implica una decisión de arquitectura no cubierta aquí (ej. nueva base de datos, nuevo servicio, cambio de flujo de auth), preguntar antes de implementar en lugar de asumir.
- Mantener consistencia entre `.env.example`, `docker-compose.yml` y cualquier documentación en `keycloak/README.md` o `docs/` — si se cambia una variable en un lado, actualizar los demás en el mismo cambio.

---
> Source: [diegoasp1023/vca-posSystem-app](https://github.com/diegoasp1023/vca-posSystem-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
