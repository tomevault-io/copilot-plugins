## nta-tracker

> - Kommunikation mit dem User auf Deutsch

# NTA Balloon Navigator - Projektregeln

## Sprache
- Kommunikation mit dem User auf Deutsch
- Code-Kommentare auf Deutsch

## Changelog
- Bei jeder Code-Änderung den Eintrag in `CHANGELOG.md` unter `## [Unveröffentlicht]` ergänzen
- Ausführlich beschreiben was geändert wurde – nicht nur "Feature X hinzugefügt", sondern auch was es tut und wie der User es benutzen kann
- Beispiel: statt "3DT-Tool hinzugefügt" → "Tools: 3DT-Rechner hinzugefügt – berechnet optimale Flugstrategie für 3D Tasks basierend auf Windschichten. Donut-Modus (max. Distanz in Ring-Zone) und Torte-Modus (gewichteter Score mit Multiplikatoren). Task-Auswahl übernimmt Ringe automatisch."

## Versionen
- Beim Release müssen 3 Stellen aktualisiert werden:
  1. `package.json` → `"version"`
  2. `src/renderer/App.tsx` → `const APP_VERSION`
  3. `src/renderer/components/LoginScreen.tsx` → `const APP_VERSION`

## Git
- Aktueller Branch: `dev` (Entwicklung)
- `main` = stabile Release-Version, nicht direkt bearbeiten
- Release-Workflow: siehe `GIT-WORKFLOW.md` auf dem Desktop

## Projekt
- Desktop App: Electron + React + TypeScript
- Lite App: Vite + React (Web für Crew)
- Backend: Supabase (Auth, DB, Realtime)
- State Management: Zustand
- Karten: Leaflet + Cesium

---
> Source: [Lukas5x5/nta-tracker](https://github.com/Lukas5x5/nta-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
