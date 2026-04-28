## muva-chat

> Guidance for Claude Code when working with this repository.

# CLAUDE.md

Guidance for Claude Code when working with this repository.

---

## Project Context

**MUVA Chat** — Multi-Tenant Tourism Platform (Next.js 15.5.9, TypeScript, PostgreSQL + PostgREST, Claude AI)

| Chat | API | Engine |
|------|-----|--------|
| `/with-me` (tenant) | `/api/dev/chat` | `dev-chat-engine.ts` |
| `/my-stay` (guest) | `/api/guest/chat` | `conversational-chat-engine.ts` |
| `muva.chat` (global) | `/api/public/chat` | `public-chat-engine.ts` |

---

## Deployment & VPS

**Rama:** Siempre `dev`. `tst` y `prd` solo reciben merges via GitHub API.

**SSH:** `ssh -i ~/.ssh/muva-vps -p 2244 root@195.200.6.216 "comando"`

**Deploy:** Commit → push → `gh pr create --base tst --head dev --fill && gh pr merge --merge` → repetir para prd → `curl -sL https://muva.chat/api/health | jq`

| Ambiente | Directorio | PM2 | Dominio |
|----------|------------|-----|---------|
| TST | /var/www/muva-chat-tst | muva-chat-tst | tst.muva.chat |
| PRD | /var/www/muva-chat-prd | muva-chat-prd | muva.chat |

**VPS emergencia:** `gh workflow run vps-exec.yml -f environment=tst -f command="pm2 status"` (requiere autorización)

**Cron (en VPS):** Sync cada 15min (`scripts/cron/sync-unified.sh`), cleanup fotos 2am, TRA disabled. Config: `/etc/cron.d/muva*` + `crontab -l`. Auth: `Bearer {CRON_SECRET}`.

---

## Database

**PostgreSQL self-hosted en VPS** — Credenciales en `.env.claude` (gitignored). Requiere SSH tunnel (`~/ssh-tunnel-muva.sh`).

| Ambiente | Host | Puerto | DB | Usuario | PostgREST |
|----------|------|--------|----|---------|-----------|
| DEV | 195.200.6.216 | 46101 | muva_dev | muva_dev_user | 3002 |
| TST | 127.0.0.1 | 46101 | muva_tst | muva_tst_user | 3003 |
| PRD | 127.0.0.1 | 46101 | muva_prd | muva_prd_user | 3004 |

### Queries (MCP preferido)
```
mcp__postgres-dev__query  → DEV (auto-allowed)
mcp__postgres-tst__query  → TST (requiere confirmación)
mcp__postgres-prd__query  → PRD (EXTREMA PRECAUCIÓN)
```
**CLI fallback DEV:** `DEV_DATABASE_URL="..." node .claude/db-query.js "SELECT ..."`

### Migraciones
```bash
touch migrations/$(date +%Y%m%d%H%M%S)_descripcion.sql   # Crear (SQL idempotente)
DEV_DATABASE_URL="..." node .claude/db-query.js "$(cat migrations/FILE.sql)"  # Aplicar DEV
```
**TST/PRD:** Los usuarios de app no tienen permisos de owner. Usar SSH + postgres superuser:
```bash
MIGRATION_SQL=$(cat migrations/ARCHIVO.sql | base64)
ssh -i ~/.ssh/muva-vps -p 2244 root@195.200.6.216 \
  "echo '$MIGRATION_SQL' | base64 -d | sudo -u postgres psql -d muva_tst"
```
Verificar siempre con MCP query después. Pre-commit hook valida idempotencia.

### Schema Reference
**OBLIGATORIO antes de queries:** Leer `.claude/db-schema-reference.md` (tabla→columnas). **NUNCA adivinar columnas.**
Un hook detecta errores "column does not exist" y retorna columnas correctas. Regenerar: `node .claude/hooks/generate-schema-ref.cjs`

### Multi-Tenant Data Model
`tenant_registry` → `hotels` → `accommodation_units` → `guest_reservations` (todas con `tenant_id` directo).
- **NO** `tenants` → usar `tenant_registry`
- **NO** `tenant_registry.hotel_id` → la relación es inversa: `hotels.tenant_id`

**Accommodation Manuals:** Ver `docs/accommodation-manuals-architecture.md`

---

## Behavioral Guidelines

1. **PRIORIZAR sugerencias del usuario** — Investigar PRIMERO antes de proponer alternativas
2. **NO work-arounds facilistas** — Investigar causa real → Proponer solución
3. **Autonomía de ejecución** — NUNCA pedir al usuario hacer tareas que yo puedo hacer
4. **COMMITS requieren autorización explícita** — NUNCA commitear sin que usuario lo pida
5. **TypeScript changes** — Buscar TODOS los archivos, agregar campos A LA VEZ, `pnpm run build` ANTES de commit
6. **Preguntas con opciones** — SIEMPRE usar `AskUserQuestion` tool (NO texto esperando respuesta)
7. **TST/PRD** — SIEMPRE consultar usuario antes de modificar
8. **VPS** — NUNCA modificar código via SSH, usar workflows

---

## Key Patterns

**Vector Search:** Send FULL chunks to LLM — NUNCA truncar (`.substring()`)

**SIRE Compliance:** USAR `src/lib/sire/sire-catalogs.ts` (USA=249, NOT 840). NUNCA ISO 3166-1 → 100% RECHAZADO.

**MUVA Embeddings:** `node scripts/database/populate-embeddings.js --all` (o archivo individual). NUNCA crear scripts alternativos.

**Google Maps URLs:** Solo `https://maps.google.com/?cid=XXX` o `maps.app.goo.gl/xxx`. NUNCA `?q=lat,lng`. Enrichment: `node scripts/migration/enrich-google-place-ids.cjs`

---

## Project Roadmap

| # | Proyecto | Status | Docs |
|---|----------|--------|------|
| 1 | ICS Calendar Integration | Pendiente | `docs/agency-tenants/` |
| 2 | Payments (Bancolombia) | MVP Wompi ✅ | `docs/payments/` |
| 3 | Landmark Photo Matching | Pendiente | `docs/chat-engagement/TODO.md` |
| 4 | Content Anchors + Semantic Chunking | Auditado | `docs/content-management/` |
| 5 | MUVA Operations | En progreso | `docs/muva-operations/` |
| 6 | WhatsApp Business | Setup ✅ | `docs/whatsapp-business-integration/` |
| 7 | Maps Integration | Pendiente | `docs/maps-integration/` |
| 8 | MUVA Ecosystem | Pendiente | `docs/muva-ecosystem/` |
| 9 | Task Management Pro | Planificado | `docs/task-management-pro/` |

**Completados:** Check-in v3 ✅ | Camera + Smart OCR ✅ | Agency Tenants FASES 1-11 ✅ | Accommodation Consolidation ✅

---

**Last Updated:** April 8, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/toneill57) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
