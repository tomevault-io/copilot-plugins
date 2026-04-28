## fb-konva-new

> Dateiorganisation


# Dateiorganisation

## Grundregeln
- **Root-Verzeichnis**: README.md, package.json, .gitignore, deploy_fb.sh
- **Dokumentation**: `docs/[features|implementation|testing|setup|architecture|plans]/`
- **Skripte**: `scripts/` oder `server/scripts/` oder `client/scripts/`
- **Reports/JSON**: `docs/implementation/` oder `shared/server/client/src/data/`
- **SQL**: `server/migrations/` oder `server/data/`

## Checkliste für neue Dateien
1. **Dokumentation?** → `docs/[kategorie]/`
2. **Skript?** → `scripts/` oder `server/scripts/`
3. **Report/JSON?** → `docs/implementation/` oder `data/`
4. **SQL?** → `server/migrations/`
5. **Konfiguration?** → Root oder `.github/` oder `deploy/`

## Details
Ausführliche Anleitung: `docs/maintainers/file-organization-guide.md`

## Beispiele
✅ `docs/implementation/report.md`, `scripts/utility.js`, `server/migrations/schema.sql`
❌ `report.md` (Root), `utility.js` (Root), `schema.sql` (Root)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pillepelle-123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
