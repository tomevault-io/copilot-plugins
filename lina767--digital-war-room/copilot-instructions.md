## frontend-react

> Frontend React/TypeScript – API, Hooks, Daten


# Frontend (React / TypeScript)

## API & Env
- Backend-URL über `import.meta.env.VITE_API_URL` mit Fallback (z. B. Production-URL); keine hart codierten `localhost` in Commit.
- Fetch zu `/api/analyze`, `/api/export/pdf` etc. mit dieser Basis-URL; Fehlerbehandlung (z. B. Toast) bei Netzwerkfehlern.

## Daten & Typen
- Analyse-Daten: Typ `ConflictData` in `useConflictWebSocket.ts` (oder gleichwertig); neue Felder (z. B. `sigint.conflict_reports`) dort ergänzen, wenn das Backend sie liefert.
- `key_findings` ist `string[]`; Anzeige als Liste/Karten je nach Komponente.

## Komponenten
- Funktionale Komponenten; wiederverwendbare Logik in Hooks (z. B. `useConflictWebSocket`, `useToast`).
- Keine unnötigen direkten Aufrufe von Backend-URLs in vielen Stellen; eine zentrale API-Basis nutzen.

---
> Source: [lina767/digital-war-room](https://github.com/lina767/digital-war-room) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
