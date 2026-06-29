## horacio-ramirez

> > Proyecto bajo el ecosistema Nexia. Lee primero `~/dev/CLAUDE.md` y

# CLAUDE.md — Horacio

> Proyecto bajo el ecosistema Nexia. Lee primero `~/dev/CLAUDE.md` y
> `~/dev/nexia-tools/NEXIA-OS.md`. Aquí solo lo específico de Horacio.

## Identidad del proyecto
Bot de Telegram para el Hora por Hora (HxH) de Mapartel (cliente consultoría).
Persona: **Horacio Ramírez** — cálido, breve, mexicano; "da antes de pedir";
nunca regaña ni compara líneas. Ver `Horacio - System Prompt (SN-04 v2).md`.

## Reglas específicas
- **Fuente de verdad = tablas `horacio.*`**, no las notas `.md` (esas son spec humana).
  El system prompt vive en `horacio.config` (key `system_prompt`, versionado).
- **Catálogos cerrados:** causas de paro y líneas solo se cambian en BD/seed, nunca
  inventadas por el bot. Línea sin estándar oficial → `estandar_status='no_estandar'`,
  NUNCA inventar meta.
- **HxH es append-only:** una corrección es un evento nuevo (`corrige_id`), no un UPDATE.
- **Prorrateo:** `plan = estándar × t_productivo_min / 60`. Comida/pausa reduce
  `t_productivo`, no cuenta como paro.
- **Gate R2-07 (anti-muerte-del-bot):** no se manda el primer ping al piso real hasta
  que los SLAs estén firmados (Daniel Nava paros, Nayeli faltantes) y los chat_id
  de las líderes dados de alta con consentimiento (RH/Ivonne). Hoy solo chat de prueba.
- **Privacidad:** resúmenes a Dirección llevan líneas, nunca nombres de operadoras.

## Reglas Nexia heredadas (recordatorio)
- DDL/DML vía `/pg/query` con SERVICE_ROLE — nunca puerto 5432, nunca dot-notation.
- Secretos (BOT_TOKEN, SERVICE_ROLE_KEY) solo en el nodo n8n — nunca en el repo.

## Arquitectura n8n
- Workflow `Horacio - Webhook` (`VKb215KJk5TdEsEY`): Webhook `/horacio-hxh` → nodo Code.
- El Code node hace `this.helpers.httpRequest` a `/pg/query` y a la API de Telegram.
- ⚠️ El Code node corre "Run Once for All Items": usar `$input.first().json`, NO `$json`.
- Botones generan callbacks estructurados (`hxh_si`, `hxh_no`, `pz_*`, `c_<CODIGO>`);
  el LLM no participa en el flujo HxH.

---
> Source: [jjrmrcly79/HORAcio_Ramirez](https://github.com/jjrmrcly79/HORAcio_Ramirez) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
