## video-recap

> Guía para trabajar en este repo con Claude Code.

# CLAUDE.md

Guía para trabajar en este repo con Claude Code.

## Qué es

Pipeline que genera un borrador de newsletter semanal a partir del VOD de un programa
de streaming: encuentra el video, descarga el audio, lo transcribe, redacta el mail con
Claude y lo envía por Resend para revisión humana. Corre en GitHub Actions (cron semanal)
o a mano en local. Ver `README.md` para el uso.

## Estructura

```
src/recap/
  __main__.py      # entrypoint: `python -m recap [url]`
  config.py        # Settings (pydantic) desde env vars; carga .env con dotenv
  models.py        # modelos de dominio (VideoMeta, Draft, etc.)
  find_vod.py      # elige el VOD del canal por fecha/duración o por TITLE_KEYWORD
  download_audio.py# descarga audio 16kHz mono wav con yt-dlp
  transcribe.py    # transcribe con Groq whisper, con chunking por tamaño
  write_email.py   # redacta el mail con Claude (structured output) + linkify timestamps
  send_draft.py    # envía HTML + Markdown por Resend
  pipeline.py      # orquesta las etapas y maneja errores/notificaciones
  retry.py         # helper de reintentos

tests/             # un test por módulo, con fixtures en tests/fixtures/
```

## Convenciones

- **Etapas independientes:** cada módulo del pipeline se puede correr y testear solo.
  Los clientes externos (Groq, Anthropic, Resend, yt-dlp) se inyectan como parámetros
  para poder mockearlos en tests.
- **Configuración por env vars** vía `config.load_settings()`. No hardcodear secretos.
  En local se leen de `.env` (gitignoreado); en CI, de GitHub Secrets.
- **Tests sin red:** ningún test llama a servicios externos. Usar los fixtures de
  `tests/fixtures/` y mockear los clientes.
- **TDD:** escribir el test antes que la implementación. Correr `.venv/bin/pytest`.

## Correr

```bash
python -m venv .venv && .venv/bin/pip install -e ".[dev]"
.venv/bin/pytest              # tests
.venv/bin/python -m recap     # pipeline (requiere .env y ffmpeg)
```

## Gaps conocidos

- `find_vod_by_title` / `select_vod_by_title` (filtro por `TITLE_KEYWORD`) todavía no
  tienen tests dedicados.

---
> Source: [SLAtv/video-recap](https://github.com/SLAtv/video-recap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
