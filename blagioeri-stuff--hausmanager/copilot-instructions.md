## hausmanager

> Next.js 14 App Router-App für Schweizer Hausbesitzer: Rücklagenplanung, Gartenmanagement, Dokumentenverwaltung, KI-Assistenz. SQLite via Prisma. Tailwind CSS. Deployed auf Chromebook/Crostini.

# Hausmanager — Claude Code Kontext

## Was ist das?
Next.js 14 App Router-App für Schweizer Hausbesitzer: Rücklagenplanung, Gartenmanagement, Dokumentenverwaltung, KI-Assistenz. SQLite via Prisma. Tailwind CSS. Deployed auf Chromebook/Crostini.

## Branch
`claude/home-maintenance-app-qoaBs` — immer hier entwickeln und pushen.

## Technik-Stack
- Next.js 14 App Router, React 18, TypeScript 5
- Prisma 5 + SQLite (`prisma/dev.db`)
- Tailwind CSS 3, Zod, React Hook Form
- `@anthropic-ai/sdk` für Claude-Aufrufe
- Vitest für Unit-Tests (`npm test`)

## Wichtige Befehle
```bash
npm run dev          # Dev-Server auf 0.0.0.0:3000
npm test             # 29 Vitest-Tests
npm run db:migrate   # Prisma-Migration ausführen
npx tsc --noEmit     # TypeScript-Check
git push -u origin claude/home-maintenance-app-qoaBs
```

## Architektur — Schlüsseldateien
| Datei | Zweck |
|---|---|
| `src/lib/db.ts` | Prisma-Client (immer `@/lib/db` importieren, nicht `@/lib/prisma`) |
| `src/lib/calculations.ts` | `enrichComponent()`, `buildReserveProjection()` — Kernlogik |
| `src/lib/component-types.ts` | COMPONENT_TYPES, Lebensdauern, Standardkosten |
| `src/lib/garden-types.ts` | PLANT_TYPES, GARDEN_ELEMENT_TYPES, MONTHS_DE |
| `src/lib/log.ts` | `writeLog(level, category, message, detail?)` — nie werfend |
| `src/lib/utils.ts` | `normalizeUrl()` |
| `src/types/index.ts` | RawComponent, EnrichedComponent, RawGardenPlant, etc. |
| `src/components/layout/Sidebar.tsx` | Navigation — navItems-Array + einstellungenItem separat unten |
| `src/components/ui/` | Button, Input, Textarea, Select, Card — alle mit forwardRef |
| `src/components/forms/ComponentForm.tsx` | RHF-Formular für Hauskomponenten |
| `src/components/garden/GardenPlantDetailClient.tsx` | Pflanzdetail inkl. Companion Planting |
| `src/components/garden/WetterWidget.tsx` | Client-Widget, fetcht `/api/garten/wetter` |

## Seitenstruktur
```
/dashboard              Server-Komponente, force-dynamic
/dashboard-ki           KI-Wartungshinweise
/komponenten            Hauskomponenten-Übersicht
/kosten                 Kostenverwaltung
/garten                 Gartenmanager-Übersicht (WetterWidget eingebettet)
/garten/pflanzen        Client-Komponente, Liste/Kachel-Toggle
/garten/elemente        Client-Komponente, Liste/Kachel-Toggle
/garten/aufgaben        KI-Vorschläge, Todos
/garten/karte           Drag&Drop-Karte, Klick-Panel
/garten/ki-analyse      Foto-Upload → Claude Vision
/dokumentation          HouseDocuments, Grid/Liste-Toggle
/links                  URLs, normalizeUrl()
/bericht                PDF-Export
/importieren            KI-Dokument-Import
/log                    Systemprotokoll (SystemLog-Tabelle)
/einstellungen          Einstellungen inkl. gartenStandort
```

## API-Routen
```
/api/components         GET/POST HomeComponent
/api/components/[id]    GET/PUT/DELETE
/api/garten/pflanzen    GET/POST
/api/garten/pflanzen/[id]  GET/PUT/DELETE
/api/garten/elemente    GET/POST
/api/garten/elemente/[id]  GET/PUT/DELETE
/api/garten/todos       GET (filter: plantId, done, month) / POST
/api/garten/todos/[id]  PUT/DELETE
/api/garten/aufgaben/ki GET → Claude generiert saisonale Aufgaben
/api/garten/pflanzen/companion  POST { plantId } → Companion-Vorschläge
/api/garten/wetter      GET → Nominatim+Open-Meteo, 1h-Cache
/api/garten/karte/background  GET/POST/DELETE Satellitenfoto
/api/garten/analyze     POST → Claude Vision Foto-Analyse
/api/log                GET (filter: level, category, days) / DELETE (olderThan)
/api/settings           GET/POST key-value-Store
/api/backup             GET list / POST { action: create|restore }
```

## Datenbank-Modelle (Prisma)
`HomeComponent`, `MaintenanceEntry`, `Document`, `Setting`, `Cost`, `HouseDocument`, `Link`, `GardenPlant`, `GardenElement`, `GardenPhoto`, `GardenTodo`, `SystemLog`

## Konventionen
- Server-Komponenten nutzen `prisma` direkt, Client-Komponenten fetchen via API
- Datumsfelder werden vor JSON-Serialisierung zu `.toISOString()` konvertiert
- `writeLog()` in allen KI- und Backup-Routen aufrufen
- Neue Sidebar-Einträge ins `navItems`-Array in `Sidebar.tsx` einfügen
- Tests: `src/lib/__tests__/*.test.ts` mit Vitest, `globals: true`

## Deployment auf Synology NAS
- Das Projekt ist Docker-ready mit `Dockerfile` und `docker-compose.yml`.
- SQLite-Datenbank liegt im Container unter `/app/data/hausmanager.db` und wird über das Volume `hausmanager_data` persistiert.
- Uploads werden in `/app/uploads` gespeichert und per Volume `hausmanager_uploads` gesichert.
- Die Container-Umgebung setzt `DATABASE_URL=file:/app/data/hausmanager.db` und `UPLOAD_DIR=/app/uploads`.
- Beim Start führt der Container `npx prisma migrate deploy` aus, um Migrationen automatisch anzuwenden.

### Synology NAS Deployment via SSH
1. Auf dem NAS einen Zielordner anlegen:
```bash
ssh claude-deploy@192.168.178.189 'mkdir -p ~/hausmanager && cd ~/hausmanager'
```
2. Projektdateien übertragen (beispielsweise per `scp` oder `rsync`):
```bash
rsync -av --exclude node_modules --exclude .git ./hausmanager/ claude-deploy@192.168.178.189:~/hausmanager/
```
3. Auf dem NAS in den Projektordner wechseln:
```bash
ssh claude-deploy@192.168.178.189 'cd ~/hausmanager && docker compose pull && docker compose build && docker compose up -d'
```
4. Alternative ohne Pull, wenn der Quellcode lokal gebaut werden soll:
```bash
ssh claude-deploy@192.168.178.189 'cd ~/hausmanager && docker compose build && docker compose up -d'
```
5. Logs prüfen:
```bash
ssh claude-deploy@192.168.178.189 'cd ~/hausmanager && docker compose logs -f hausmanager'
```

### Wichtige Hinweise
- Synology verwendet in der Regel `docker compose`; `docker-compose` kann bei älteren DSM-Versionen anders heißen.
- Setze sensible Umgebungsvariablen wie `CLAUDE_API_KEY` nur auf dem NAS, nicht im Repo.
- Wenn du ein NAS-spezifisches Backup-Verzeichnis brauchst, ergänze `BACKUP_DIR` in einer `.env` oder in `docker-compose.yml`.

## Aktueller Stand (v1.1.0)
Alle Features aus Phase 9, G2, G3, G4 implementiert. Letzter Commit: `b6d7497`.
Offene Punkte: keine bekannten Bugs.

---
> Source: [blagioeri-stuff/hausmanager](https://github.com/blagioeri-stuff/hausmanager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
