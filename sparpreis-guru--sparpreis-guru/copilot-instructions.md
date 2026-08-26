## sparpreis-guru

> Diese Datei beschreibt den verlässlichen Arbeits- und Prüfablauf für dieses Repository.

# Codex-Projektleitfaden

Diese Datei beschreibt den verlässlichen Arbeits- und Prüfablauf für dieses Repository.

## Paketmanager

- Die im Feld `packageManager` der `package.json` festgelegte pnpm-Version ist verbindlich und wird über Corepack verwendet.
- Unter Windows nicht `pnpm` oder `pnpm.cmd` direkt aufrufen. Der globale Wrapper unter `%LOCALAPPDATA%\pnpm\bin` kann in dieser Umgebung hängen.
- Stattdessen immer `corepack pnpm <befehl>` verwenden.
- Beispiele:
  - Entwicklung: `corepack pnpm dev`
  - Abhängigkeiten installieren: `corepack pnpm install`
  - Produktions-Build: `corepack pnpm build`

## Verifikation

Die Prüfung soll proportional zur Änderung erfolgen. Solange aktiv an einem Feature gearbeitet wird, müssen nach Zwischenständen nicht automatisch Typprüfung, Git-Prüfung oder Browser-Test ausgeführt werden. Erst wenn das Feature erkennbar fertig entwickelt ist oder der Nutzer ausdrücklich eine Prüfung verlangt, gelten die folgenden Prüfungen. Bei einem Auftrag zur Commit- oder Release-Vorbereitung sind sie immer auszuführen.

1. Nach abgeschlossenen TypeScript-/TSX-Änderungen zuerst die schnelle Typprüfung ausführen:

   `node .\node_modules\typescript\bin\tsc --noEmit --pretty false`

   Alternativ funktioniert `corepack pnpm exec tsc --noEmit --pretty false`.

2. Nach kleinen CSS- oder Textänderungen zusätzlich ausführen:

   `git diff --check`

3. Nach größeren, routing-, build- oder release-relevanten Änderungen den vollständigen Build ausführen:

   `corepack pnpm build`

4. Richtwerte für Timeouts:
   - Typprüfung: 30 Sekunden
   - Produktions-Build: 120 Sekunden

5. Wenn ein Befehl das Zeitlimit überschreitet:
   - Nicht denselben Befehl mehrfach parallel starten.
   - Prüfen, ob versehentlich der globale `pnpm`-Wrapper verwendet wurde.
   - Den direkten TypeScript-Befehl oben als Gegenprobe verwenden.
   - Nur Prozesse beenden, die nachweislich durch die aktuelle Codex-Aufgabe gestartet wurden.

## Browser und visuelle Prüfung

- Der integrierte Browser ist in der Codex-Erweiterung für VS Code und in der Codex CLI nicht verfügbar. Das ist eine Einschränkung der Oberfläche und kein lokaler Konfigurationsfehler.
- In einer VS-Code-Sitzung nicht wiederholt versuchen, den integrierten Browser zu initialisieren.
- Für einen einfachen lokalen Erreichbarkeitstest darf bei laufendem Dev-Server beispielsweise `Invoke-WebRequest http://localhost:3000/<route>` verwendet werden. Das ersetzt keine visuelle Prüfung.
- In der Regel hat der Nutzer einen dev-server auf Port 3000 laufen, davon ist erst einmal auszugehen.
- Für echte Viewport-, Interaktions- oder Screenshot-Prüfungen das Projekt bzw. den Chat in der ChatGPT-Desktop-App öffnen und dort den integrierten Browser mit der lokalen URL verwenden.
- Browser- und Sichtprüfungen nur ausführen, wenn der Nutzer sie ausdrücklich anfordert; nicht jede Layoutkorrektur automatisch im Browser öffnen.
- Die fehlende Browser-Verfügbarkeit nur erwähnen, wenn visuelle Prüfung für das Ergebnis relevant ist. Keine wiederholten Standardhinweise bei reinem Code- oder Layout-Editing.
- Keine Browser-Test-Abhängigkeit wie Playwright hinzufügen, sofern der Benutzer nicht ausdrücklich eine projektlokale Browser-Testumgebung wünscht.

Offizielle Hinweise:

- Browser: https://learn.chatgpt.com/docs/browser

## Arbeitsverzeichnis

- Bereits vorhandene oder parallel entstandene Änderungen gehören dem Benutzer und bleiben unangetastet.
- Für Git-Befehle in dieser Windows-Umgebung bei Bedarf verwenden:

  `git -c safe.directory='C:/Users/dudoo/Downloads/bahn vibe/bahn.vibe' <befehl>`

## Zusammenarbeit und Commits

- Visuelle Änderungen zunächst uncommitted zur Prüfung durch den Nutzer lassen. Keine Commits anlegen, solange der Nutzer sie nicht ausdrücklich freigibt.
- Wenn Commits gewünscht sind, thematisch trennen und Conventional Commits verwenden, da Release Please daraus Changelog und SemVer ableitet:
  - `feat:` für neue sichtbare Funktionalität,
  - `fix:` für Korrekturen bestehenden Verhaltens oder Layouts,
  - `docs:` und `chore:` für reine Dokumentation beziehungsweise Wartung,
  - `feat!:` oder ein `BREAKING CHANGE:`-Footer nur für echte inkompatible Änderungen.
- Vor dem Commit immer `git status`, den vollständigen Diff und `git diff --check` prüfen. Keine fremden Dateien versehentlich stagen.
- Nicht nach jeder kleinen Layoutiteration bauen oder committen. Erst die Nutzerprüfung abwarten und zusammengehörige Korrekturen anschließend sinnvoll bündeln.

## Cache-Datenbank und Dokumentation

- Die Regeln dieses Abschnitts gelten für die Cache-Datenbank `connection-cache.db`, nicht für die separat bereitgestellte Direktverbindungs-Datenbank.
- `docs/database-maintenance.md` ist die verbindliche Dokumentation für Aufbau, Datenbereiche, Aufbewahrungsregeln, Migrationen und Wartungsbefehle der Cache-Datenbank.
- Bei Änderungen am Schema, an Tabellen oder Indizes, an Aufbewahrungs- und Bereinigungsregeln, am dokumentierten Runtime-Verhalten oder an Wartungsbefehlen immer prüfen, ob `docs/database-maintenance.md` angepasst werden muss, und die notwendige Dokumentationsänderung in derselben Aufgabe umsetzen. Reine interne Refactorings ohne Änderung dieses Vertrags benötigen keine Dokumentationsänderung.
- Den Nutzer im Ergebnis ausdrücklich darüber informieren, welche Datenbankdokumentation angepasst wurde oder warum keine Anpassung erforderlich war.
- Änderungen am Datenbankformat, die mit vorhandenen Daten oder älteren Anwendungsversionen inkompatibel sein könnten, niemals still innerhalb der aktuellen Version umsetzen. Für `connection-cache.db` immer eine neue fortlaufende Migration unter `lib/database/migrations`, eine erhöhte `latestVersion` in `lib/database/database-schema.json` und die passende Dokumentation anlegen. Bereits veröffentlichte Migrationen bleiben unverändert.
- Datenbereinigung und Cache-Invalidierung sind keine Schema-Migrationen. Destruktive Wartungsbefehle müssen ihren exakten Datenbereich nennen, ohne `--yes` nur eine Vorschau liefern und dürfen `schema_migrations`, unbekannte Tabellen oder die jeweils andere Datenbank nicht berühren.

## Produkt-, Design- und Architekturquellen

- `PRODUCT.md` beschreibt die aktuelle fachliche Produktwahrheit; `DESIGN.md` beschreibt das bestehende visuelle System. Bestehende Implementierung und benachbarte Abläufe prüfen, bevor Verhalten oder Gestaltung geändert werden.
- Gemeinsame Komponenten wie `components/search/journey-result.tsx` und `components/search/journey-sort-controls.tsx` weiterverwenden, statt nahezu identische Ergebnisdarstellungen oder Sortieroberflächen lokal neu zu implementieren.
- Änderungen an der gemeinsamen Ergebnisdarstellung in Bestpreissuche, flexibler Hin-/Rückfahrt und Urlaubsfinder prüfen.
- Keine nicht belegten Produktversprechen, Testimonials, Kundenlogos, Auszeichnungen oder Leistungsangaben erfinden.

## Datum und Suchlogik

- Suchdatumsgrenzen immer mit den zentralen Helfern aus `lib/shared/berlin-date.ts` auf Basis von `Europe/Berlin` bestimmen. Keine Server-Lokalzeit oder unabhängige Datumslogik einführen.
- Regeln für zulässige Hin- und Rückfahrtdaten client- und serverseitig über die gemeinsamen Helfer in `lib/search/return-search-feasibility.ts` ableiten.

## Direktverbindungs-Datenbank und Distribution

- Die kanonische Anwendung liegt unter `https://github.com/sparpreis-guru/sparpreis.guru`. Neue Links und Dokumentation dürfen nicht mehr den alten persönlichen GitHub-Namespace als Projektquelle verwenden.
- Die täglich erzeugte Direktverbindungs-Datenbank liegt als Rolling-Release-Asset im separaten Repository `sparpreis-guru/sparpreis.guru-direct-connections-data` unter dem festen Tag `direct-connections-data`. Sie wird nicht täglich in die Git-Historie dieses Repositories committed.
- Der Runtime-Updater liegt in `app/api/direct-connections/direct-connections-db.ts`. Er akzeptiert nur erwartete GitHub-API-/Asset-URLs, validiert Asset-Name, Größe, SHA-256-Digest und SQLite-Inhalt und ersetzt die Cache-Datei atomar mit Backup.
- Die aktive heruntergeladene Datei liegt unter `data/direct-connections.db`; `public/direct-connections.db` ist nur der gebündelte Fallback. Die zugehörige Release-Metadatei ist `data/direct-connections.release.json`. Fehlt sie bei einem alten Cache oder Fallback, wird einmalig aus dem neuen Release nachgeladen.
- Der Release-Check erfolgt pro laufender Instanz höchstens alle 12 Stunden; ein fehlgeschlagener Abruf wird nach fünf Minuten erneut versucht. Die Datenbank wird also nicht bei jedem Seitenaufruf erneut heruntergeladen.
- Keine Unterstützung für nie produktiv verwendete alte `YYYYMMDD`-Assetnamen ergänzen. Das aktuelle Assetformat enthält UTC-Zeitstempel und Digest-Präfix.
- Der Python-Builder `scripts/build-direct-connections.py` lädt bei jedem vollständigen Build bewusst frische GTFS-Feeds. Ein alter lokaler ZIP-Cache darf nicht unbemerkt als aktuelle Datenbasis dienen.
- Beim Komprimieren ist die große Übersichtsstruktur bewusst auf gzip-Stufe 1 und sind die vielen Detail-Blobs auf Stufe 6 eingestellt. Diese Aufteilung ist ein gemessener Kompromiss aus Buildzeit und Dateigröße; nicht pauschal alle Blobs auf dieselbe Stufe setzen. Die vorberechneten Zeitstrings und der `lru_cache` für Masken sind ebenfalls gezielte Builder-Optimierungen.
- Python ist nur für das Erzeugen der Datenbank erforderlich, nicht für normale Nutzer oder den Runtime-Download.
- Änderungen am Datenformat oder Builder immer mit dem Workflow des separaten Daten-Repositories abstimmen. Erzeuger, Assetname, SQLite-Schema und Runtime-Validierung bilden gemeinsam einen Vertrag.
- Die alte große Datenbank bleibt in der vorhandenen Git-Historie. Diese Historie nicht ohne ausdrückliche Entscheidung, Backup und koordinierte Force-Push-Migration mit `filter-repo` oder ähnlichen Werkzeugen umschreiben. Neue tägliche Datenstände gehören ausschließlich ins Rolling Release.

## Repository- und Container-Migration

- Release Please läuft auf `main` mit `release-type: node`. Zusätzliche deutsche Migrationshinweise können nach der automatischen Generierung in den Release-Text aufgenommen werden; Conventional-Commit-Texte ersetzen solche Hinweise nicht vollständig.
- Docker-Images werden vorerst absichtlich parallel nach Docker Hub (`butti/sparpreis-guru`), in den Organisations-GHCR-Namespace (`ghcr.io/sparpreis-guru/sparpreis-guru`) und zur Abwärtskompatibilität in den alten GHCR-Namespace gepusht. Den Legacy-Push und `LEGACY_GHCR_TOKEN` nicht ohne ausdrückliche Migrationsentscheidung entfernen.
- Für neue Dokumentation und Empfehlungen ist das Organisations-Image der primäre Pfad. Der alte persönliche Namespace existiert nur noch als Übergangskompatibilität.

---
> Source: [sparpreis-guru/sparpreis.guru](https://github.com/sparpreis-guru/sparpreis.guru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
