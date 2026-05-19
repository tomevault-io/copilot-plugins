## drawfrommemory

> **Merke und Male** — Gedächtnistraining-App für Kinder (React Native / Expo).

# CLAUDE.md — DrawFromMemory

## Projekt

**Merke und Male** — Gedächtnistraining-App für Kinder (React Native / Expo).
Spieler sehen ein Bild kurz, zeichnen es aus dem Gedächtnis, vergleichen das Ergebnis.

- **Aktuell: v1.3.4** — Play-Store-ready, keine offenen Blocker
- **In staging (seit 2026-05-08):** PR #164 — i18n-Fix: hardcodierte App-Titel in HomeScreen & GameScreen durch `t('app.name')` ersetzt
- **Live Demo:** https://s540d.github.io/DrawFromMemory/
- **Repo:** https://github.com/S540d/DrawFromMemory

---

## Tech Stack

| Bereich | Technologie |
|---|---|
| Framework | React Native + Expo (SDK 55) |
| Navigation | Expo Router (file-based) |
| Native Drawing | `@shopify/react-native-skia` |
| Web Drawing | Canvas API (`DrawingCanvas.web.tsx`) |
| State | React Hooks + AsyncStorage |
| i18n | custom `services/i18n.ts` (de/en) |
| Tests | Jest + jest-expo (241 Tests) |
| CI | GitHub Actions (`ci-cd.yml`) |
| Crash Reporting | Sentry (optional via `EXPO_PUBLIC_SENTRY_DSN`) |

---

## Wichtige Dateien

```
app/
  game.tsx              # Haupt-Spielschirm (Merken → Zeichnen → Ergebnis)
  gallery.tsx           # Galerie gespeicherter Zeichnungen
  levels.tsx            # Level-Auswahl
  settings.tsx          # Einstellungen

components/
  DrawingCanvas.tsx          # Web-Implementierung
  DrawingCanvas.native.tsx   # Native Skia-Implementierung (Flood-Fill via Rect-Spans)
  DrawingCanvas.web.tsx      # Web Canvas-Implementierung
  LevelImageDisplay.tsx      # SVG-Bild mit schrittweisem Aufdecken
  ParentalGate.tsx           # Eltern-Sperre für Einstellungen

services/
  FloodFillService.ts        # Flood-Fill-Algorithmus (Scanline, 1-Bit-Palette)
  SoftwareRasterizer.ts      # CPU-Rasterizer für native Fill-Grenzenerkennung
  NativeFillLayerService.ts  # Konvertiert DrawingPath[] → native Skia Rect-Spans
  LevelManager.ts            # Level-Definitionen und -Verwaltung
  StorageManager.ts          # AsyncStorage-Wrapper
  useGamePhase.ts            # Spielphasen-Hook (Merken/Zeichnen/Ergebnis)
  useTimer.ts                # Timer-Hook mit Pause/Resume
```

---

## Entwicklungs-Workflow

```bash
npm start          # Expo Dev Server
npm test           # Jest (--runInBand für stabile Ergebnisse)
npm run lint       # ESLint
npm run build:web  # Web-Build (gh-pages)
npx tsc --noEmit   # TypeScript-Check
```

**Branch-Strategie:** Feature-Branches → PR → `testing` (QA) → `staging` (Pre-Production) → `main` (Produktion). Alle drei Branches (`main`, `staging`, `testing`) müssen immer existieren und dürfen nie gelöscht werden.

---

## CI/CD (`.github/workflows/ci-cd.yml`)

Jobs:
1. **Code Quality & Linting** — ESLint
2. **Unit Tests & Coverage** — Jest
3. **Build Web** — Expo Web Build
4. **Security Audit** — `npm audit --audit-level=high` (blockiert bei high/critical)
5. **Credential Scan** — Keystore & sensible Dateien
6. **Docs Privacy Check** — prüft `docs/private/` auf versehentliche Commits
7. **Platform Compatibility** — React Native Kompatibilitätsprüfung

---

## Flood-Fill Architektur (Native)

Das größte technische Thema der letzten Entwicklungsphase war die Flood-Fill auf alten Android-Geräten (Nexus 6 / Adreno 420). Die GPU-Pipeline (`readPixels` / `MakeImage`) ist dort unzuverlässig.

**Aktueller Ansatz (v1.3.4):**
- CPU-Flood-Fill via `floodFillPixels()` (Scanline-Algorithmus, 1-Bit-Palette)
- Ergebnis wird als horizontale Rect-Spans (`FloodFillSpan[]`) ausgegeben
- Native rendering via Skia `<Rect>` Elemente (kein Image-Roundtrip)
- Strokes werden IMMER als `<Path>` Komponenten gerendert — kein Mode-Switching
- `NativeFillLayerService` konvertiert `DrawingPath[]` → renderfähige Layers

---

## Security

- `npm audit --audit-level=high` in CI — Pipeline blockiert bei high/critical
- Verbleibende 5 low-Findings: `@tootallnate/once` via jest-expo-Chain — fix erfordert breaking jest-expo Major-Upgrade, intentionally excluded
- Alle high/critical Vulnerabilities zuletzt gefixt: 2026-04-21 via PR #144

---

## Offene Issues / Bekannte Einschränkungen

- **jest-expo → @tootallnate/once** (low severity): Fix erfordert `jest-expo@47` — breaking change, noch nicht gemacht
- **Nexus 6 Flood-Fill**: Implementierung fertig, aber kein Gerät für manuelle Tests vorhanden
- **iOS**: Nicht primär getestet (Fokus auf Web + Android)

---
> Source: [S540d/DrawFromMemory](https://github.com/S540d/DrawFromMemory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
