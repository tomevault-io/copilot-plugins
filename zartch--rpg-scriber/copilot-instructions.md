## rpg-scriber

> Guía de referencia para Claude Code al trabajar en este proyecto.

# CLAUDE.md — RPG Scribe

Guía de referencia para Claude Code al trabajar en este proyecto.

## Instrucciones Generales

- **NO hacer commits** a menos que el usuario lo pida explícitamente. El usuario probará antes de hacer commit.

## Descripción del Proyecto

RPG Scribe escucha partidas de rol en tiempo real (vía Discord), transcribe quién dice qué y genera un resumen narrativo vivo de la sesión usando IA. Distingue diálogo in-game de meta-rol. Incluye Web UI para monitorizar sesiones, editar campaña/NPCs y revisar historial.

## Comandos Esenciales

```bash
pip install -e ".[dev]"            # dependencias desarrollo
pip install -e ".[local]"          # + faster-whisper local

rpg-scribe --campaign config/campaigns/example.toml
rpg-scribe                         # modo genérico (sin campaña)

pytest                             # toda la suite
pytest -k test_nombre              # test específico
ruff check src/ tests/             # linter
ruff format src/ tests/            # formatear
```

## Arquitectura

Patrón event-driven async. Flujo principal:

```
Listener → [AudioChunkEvent] → Transcriber → [TranscriptionEvent] → Summarizer → [SummaryUpdateEvent]
```

Capas backend: **Routers** → **Services** → **Repositories** → **Database** (SQLite/aiosqlite).

Frontend: ES modules browser-nativos en `web/static/js/`, CSS modular en `web/static/css/`.

Ver detalles completos: [`docs/architecture.md`](docs/architecture.md)

### Documentación de Dominio

- Summarizer y prompts: [`docs/summarizer-context.md`](docs/summarizer-context.md)
- Web UI features: [`docs/web-ui-features.md`](docs/web-ui-features.md)
- Entidades, locations, relationships: [`docs/web-ui-entities.md`](docs/web-ui-entities.md)
- REST API endpoints: [`docs/api-endpoints.md`](docs/api-endpoints.md)
- Loading states y UX: [`docs/loading-states-ux.md`](docs/loading-states-ux.md)
- Audio chunks: [`docs/audio-chunks-design.md`](docs/audio-chunks-design.md)
- Discord voice y DAVE E2EE: [`docs/discord-voice.md`](docs/discord-voice.md) — incluye **bug abierto**: degradación silenciosa del audio en sesiones largas (sin resolver)
- Campaign summaries: [`docs/campaign-summaries.md`](docs/campaign-summaries.md)
- TTS narración: [`docs/tts-narration.md`](docs/tts-narration.md)
- Grafo canónico de entidades/relaciones: [`docs/canonical-graph-model.md`](docs/canonical-graph-model.md)
- Módulo RAG de manuales: [`docs/rag-lib.md`](docs/rag-lib.md)
- Bot de reglas (RAG) y framework de bots por voz: [`docs/rules-bot.md`](docs/rules-bot.md)

## Convenciones de Código

- Python 3.10+, todo async (`async/await`)
- Dataclasses para modelos y eventos (`frozen=True` para eventos)
- ABC para interfaces (`BaseListener`, `BaseTranscriber`, `BaseSummarizer`)
- `from __future__ import annotations` en todos los módulos
- Linter: ruff | Tests: pytest + pytest-asyncio (`asyncio_mode = "auto"`)

## Planes de Implementación

- Organizar en **fases numeradas**
- Indicar explícitamente qué fases son paralelizables y cuáles secuenciales
- Formato: `Fase N (paralelizable con Fase M)` o `Fase N (requiere Fase M)`

## Variables de Entorno

| Variable | Descripción | Requerida |
|---|---|---|
| `DISCORD_BOT_TOKEN` | Token del bot de Discord | Sí |
| `OPENAI_API_KEY` | API key de OpenAI (transcripción) | Sí |
| `ANTHROPIC_API_KEY` | API key de Anthropic (resumen) | Sí |
| `RPG_SCRIBE_HOST` | Host del Web UI (default: 127.0.0.1) | No |
| `RPG_SCRIBE_PORT` | Puerto del Web UI (default: 8000) | No |
| `RPG_SCRIBE_DB` | Ruta de la base de datos (default: rpg_scribe.db) | No |
| `DISCORD_SUMMARY_CHANNEL_ID` | Canal de Discord para resúmenes | No |

## Testing

- 15 archivos de test, ~405 tests (~404 pass, 1 fallo pre-existente)
- **Fallos pre-existentes conocidos**:
  - `test_tts_config_from_toml`: el `default.toml` tiene TTS habilitado, el test espera `False`
  - `test_defaults_from_toml_override_dataclass_defaults`: falla si `RPG_SCRIBE_HOST` está en env vars
  - `test_generate_toml_is_valid_toml`: requiere `tomllib` (Python 3.11+), no disponible en 3.10
  - `test_half_open_failure_reopens` en `test_resilience.py`: timing sensible

---
> Source: [Zartch/Rpg-Scriber](https://github.com/Zartch/Rpg-Scriber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
