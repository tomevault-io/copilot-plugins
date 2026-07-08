## ab-generator-codexcopy

> `package.json` → `"version"` ist die einzige Quelle der Wahrheit. `vite.config.ts` liest

# Versionierung

`package.json` → `"version"` ist die einzige Quelle der Wahrheit. `vite.config.ts` liest
sie in `__APP_VERSION__` ein, `AppFooter.tsx` zeigt sie in der Fußzeile an. Bei jeder
Änderung dort automatisch mit.

**Regel: Bei jedem Push auf `main` wird die Version erhöht — nach Größe der Änderung:**

| Änderungsgröße | Beispiel | Bump |
|---|---|---|
| Klein (Fix) | Bugfix, kleine UI-Korrektur, Copy-Änderung, Absturz-Fix | Patch: `x.y.Z` → `x.y.(Z+1)` |
| Mittel (Feature) | neues Feature, spürbare UX-Erweiterung, nicht-brechende Verhaltensänderung | Minor: `x.Y.z` → `x.(Y+1).0` |
| Groß (Breaking) | Breaking Change, große Architektur-Umbauten, grundlegendes Redesign | Major: `X.y.z` → `(X+1).0.0` |

Vorgehen pro Push:
1. Änderungsgröße einschätzen (siehe Tabelle oben).
2. `"version"` in `package.json` entsprechend erhöhen.
3. Eintrag in `CHANGELOG.md` ergänzen (neuester Eintrag oben, Datum + Kurzbeschreibung).
4. Versions-Bump gehört in denselben Commit wie die fachliche Änderung (bei mehreren
   Commits pro Push: in den letzten Commit vor dem Push).

Bei Unsicherheit zwischen zwei Stufen: lieber die kleinere wählen und im Zweifel
nachfragen, statt die Versionsnummer voreilig hochzuziehen — Rücksprünge sind mit Semver
nicht vorgesehen.

---
> Source: [TaughtMe/AB-Generator-CodexCopy](https://github.com/TaughtMe/AB-Generator-CodexCopy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
