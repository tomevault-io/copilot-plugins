## konnoprint-n8n

> Arbeitsprojekt für die n8n-Workflows der **konnoprint-Instanz**.

# Projekt n8n-KonnoPrint

Arbeitsprojekt für die n8n-Workflows der **konnoprint-Instanz**.

## Instanz

| | |
|---|---|
| URL | `https://n8n-konnoprint.rikoma.cloud` |
| Interne Bezeichnung | konnoprint |
| Hosting | Hostinger-VPS, Easypanel-Projekt `konnoprint` |
| Services (Docker Swarm) | `n8n-konnoprint` (n8n 2.29.8), `n8n-konnoprint-db` (Postgres 17, DB `konnoprint`), `n8n-konnoprint-runner` (Task Runner 2.29.8) |
| Kritikalität | hoch (CRM-Automationen, Call-Transkription) |

Es gibt insgesamt drei n8n-Instanzen (konnoprint, rikoma, Scheppler) — **in diesem Projekt wird ausschließlich an der konnoprint-Instanz gearbeitet.** VPS-, Easypanel- und n8n-Update-Themen gehören in `../Projekt-VPS-Hostinger/`.

## Zugänge

- **VPS:** SSH-Alias `vps-hostinger` (root@145.223.97.222, Key `~/.ssh/id_ed25519_vps`, passwortlos).
- **n8n-Datenbank (Postgres):** kein API-Key nötig, direkter Lesezugriff:
  ```bash
  ssh vps-hostinger 'DB=$(docker ps --filter name=konnoprint_n8n-konnoprint-db --format "{{.Names}}" | head -1); docker exec $DB psql -U postgres -d konnoprint -c "..."'
  ```
  Wichtige Tabellen: `workflow_entity` (nodes/connections als JSON), `execution_entity`, `execution_data`, `user_api_keys`.
- **n8n REST-API / MCP:** Es existiert **noch kein API-Key** (`user_api_keys` ist leer). Für n8n-mcp erst in der n8n-UI erstellen (*Settings → n8n API*) und in `.mcp.json` eintragen — siehe README.
- **Execution-Daten** in `execution_data.data` sind n8n-„flattened" JSON (Array mit String-Index-Pointern) — zum Lesen dereferenzieren.

- **Notion:** konnoprint ist ein **Teamspace im Rikoma-Workspace („Dein Business System")** — der normale claude.ai-Notion-Connector funktioniert, es ist **kein** separater Notion-MCP nötig (anders als bei Scheppler).

## Arbeitsweise

- **Nur lesend** auf DB und (künftig) MCP zugreifen. Keine schreibenden Operationen an Workflows — Änderungen nimmt der User manuell in der n8n-Oberfläche vor; Claude liefert exakte Node-Konfigurationen (immer Node-Typ + einzustellende Felder angeben) und prüft danach über die DB gegen.
- Vor Workflow-Arbeit `~/.claude/docs/n8n-regeln.md` lesen (Code-Node-Regeln, HTTP-Request-Eigenheiten, `$env`-Block).
- **Shell-Fallstrick:** Verschachtelte Quotes in `ssh vps-hostinger '...'` zerschießen SQL/Regex — lange Statements lokal als Skript schreiben und per `ssh vps-hostinger 'bash -s' < skript.sh` durchpipen.
- Services stoppen/starten nur über `docker service scale <name>=0|1` (Swarm), nie `docker stop` — und nur in Absprache, das ist Sache von `../Projekt-VPS-Hostinger/`.

---
> Source: [ri-ko-ma/KonnoPrint-n8n](https://github.com/ri-ko-ma/KonnoPrint-n8n) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
