## acaxeemx-chatbot

> Este archivo es cargado automáticamente por Claude Code al abrir este repo. Sirve como índice del contexto versionado del proyecto. Para detalles, sigue los enlaces.

# Proyecto Acaxee — Contexto para agentes de IA

Este archivo es cargado automáticamente por Claude Code al abrir este repo. Sirve como índice del contexto versionado del proyecto. Para detalles, sigue los enlaces.

## Arquitectura del proyecto

- [app/](app/) — Código del **chatbot Acaxee** (Flask + Telegram + Firebase RTDB + GitHub Models / Anthropic).
- [app/prompts/system_prompt.py](app/prompts/system_prompt.py) — Prompt principal del chatbot. Fuente de verdad operativa.
- [prompt_engineering/](prompt_engineering/) — Nivel superior de arquitectura: diseño y revisión de prompts para todos los agentes del ecosistema Acaxee. **No es parte del runtime del chatbot.**

## Agente Ingeniero de Prompts (AIP)

El proyecto cuenta con un rol de manager de prompts definido en [prompt_engineering/agent_prompt_engineer.md](prompt_engineering/agent_prompt_engineer.md). Cualquier tarea de diseño, revisión o mejora de prompts (del chatbot o de agentes futuros) debe pasar por ese rol y apoyarse en la base de conocimiento que mantiene.

## Base de conocimiento del negocio

Vive en [prompt_engineering/knowledge/](prompt_engineering/knowledge/) y es la **fuente de verdad única** para que cualquier agente nuevo herede coherencia de marca, tono y reglas del negocio:

- [prompt_engineering/knowledge/brand_voice.md](prompt_engineering/knowledge/brand_voice.md) — Esencia, personalidad, tono, framework conversacional (Conectar → Resolver → Sugerir), triggers de venta, prohibiciones. Voz oficial de Acaxee.

- [prompt_engineering/knowledge/restaurant_layout.md](prompt_engineering/knowledge/restaurant_layout.md) — Distribución de mesas: 42 mesas, 7 zonas, 240 sillas. Fuente de verdad para asignación de mesas en reservas.

- [prompt_engineering/knowledge/restaurant_tables.json](prompt_engineering/knowledge/restaurant_tables.json) — JSON estructurado con mesas, zonas, capacidad, forma y reglas de ocupación mínima. Fuente para seed a Firebase (`/restaurant_config`). Se sube con `python -m scripts.seed_restaurant_config`.

A medida que crezca el ecosistema (CRM, marketing, reportería), se sumarán aquí fichas adicionales (hechos de negocio, registro de agentes, etc.).

## Reglas para agentes de IA que trabajen en este repo

1. **Voz de marca:** Cualquier prompt o texto de cara al cliente final debe respetar [prompt_engineering/knowledge/brand_voice.md](prompt_engineering/knowledge/brand_voice.md). Nada de "estimado cliente", nada de "el más exclusivo".
2. **Segmentación:** No mezclar código del chatbot ([app/](app/)) con artefactos de diseño de prompts ([prompt_engineering/](prompt_engineering/)). Son capas distintas de la arquitectura.
3. **Idioma:** Español natural. El negocio, el equipo y los clientes son hispanohablantes.
4. **Sitio web (acaxeemazatlan.com):** Referencia de *contenido* (platos, servicios, vibra del lugar), NO de *voz*. La voz se define en `brand_voice.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JulianHerreraG) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
