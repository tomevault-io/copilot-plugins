## truths-and-rights

> App ciudadana que informa derechos ante intervenciones policiales en Peru.

# Truths and Rights

App ciudadana que informa derechos ante intervenciones policiales en Peru.

## Estructura

```
data/PE/           → Datos legales (JSON): situaciones, derechos, acciones, fuentes, contactos, mitos, contextos
schema/            → Schema SQLite (12 tablas + junction tables + views)
scripts/           → build_db.py, scrape_official.py, generate_site.py, validate_sources.py, history.py, check_scraper_health.py
mobile/            → App React Native (Expo) - offline-first con SQLite embebido
site/              → Portal estatico generado (no versionado, se genera en CI)
build/             → DB SQLite generada (no versionado)
tests/             → pytest para validacion de datos
.github/workflows/ → CI (validate, deploy-site, check-emergency, check-updates, full-scrape)
docs/              → AUTOMATION.md, LEGAL_METHODOLOGY.md, DATA_FORMAT.md, ADDING_A_COUNTRY.md
```

## Comandos

```bash
make build-pe          # Genera DB SQLite desde los JSON
make validate-pe       # Valida integridad de datos
make test              # pytest
make site              # Genera portal HTML estatico
make serve             # Sirve portal en localhost:8000
make mobile-install    # npm install en mobile/
make mobile-copy-db    # Copia DB a mobile/assets/db/
make mobile-start      # Inicia Expo dev server
make mobile-build-apk  # Build APK con EAS
make mobile-test       # Jest tests de la app
```

## Principios

- Si no tiene fuente oficial verificable, no entra a la base de datos
- Los workflows NUNCA modifican datos legales automaticamente (solo crean Issues)
- La app funciona offline: toda la DB se embebe (~256KB)
- IDs en snake_case ingles, textos en espanol
- Fechas ISO 8601, JSON indent 2, UTF-8

## Stack

- **Datos:** JSON → SQLite via build_db.py
- **App movil:** React Native + Expo + expo-sqlite + Zustand + React Navigation
- **Portal web:** Python puro genera HTML estatico (sin frameworks)
- **CI:** GitHub Actions (validacion, deploy, verificacion automatica de fuentes)
- **Scraper:** requests + BeautifulSoup (TC, El Peruano, Congreso)

## Archivos clave para contexto

- `data/PE/metadata.json` — Config del pais, emergencias activas
- `data/PE/situations/01_id_check.json` — Ejemplo de situacion
- `schema/database_schema.sql` — Schema completo
- `mobile/src/database/queries.ts` — Queries SQL de la app
- `mobile/src/database/search.ts` — Motor de busqueda offline
- `docs/AUTOMATION.md` — Como funciona la automatizacion

## Estado actual (v0.4.0)

- 43 fuentes legales, 20 derechos, 9 situaciones, 17 acciones, 7 contactos, 7 mitos
- App movil completa: pantallas, navegacion, DB, busqueda, tests
- Portal publico en https://ellokoakrata.github.io/truths-and-rights/
- Automatizacion: check semanal de emergencia, quincenal de fuentes, mensual scrape
- Decreto DS N° 006-2026-PCM venció 2026-02-19 (marcado como expirado). Verificar si fue renovado en El Peruano.

## Skills disponibles

- `/truths-and-rights-context` — Contexto completo del proyecto
- `/plan-mobile-app` — Guia para la app movil

---
> Source: [ELLokoAkrata/truths-and-rights](https://github.com/ELLokoAkrata/truths-and-rights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
