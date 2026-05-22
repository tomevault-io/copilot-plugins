## ratsi-melle

> - `src/` enthaelt die Python-Pakete fuer Fetching, Parsing, Analyse und Interfaces.

# Repository Guidelines

## Projektstruktur & Modulorganisation

- `src/` enthaelt die Python-Pakete fuer Fetching, Parsing, Analyse und Interfaces.
- `scripts/` enthaelt CLI-Einstiegspunkte fuer Datenaufbau, Exporte und Hilfsjobs.
- `tests/` enthaelt pytest-Tests und Fixtures.
- `data/` enthaelt Laufzeitdaten und lokale Datenbanken.
- `docs/` enthaelt Projektdokumentation; Detailregeln zur Ablage stehen in `docs/repository_guidelines.md`.

## Build-, Test- und Entwicklungsbefehle

- `pip install -r requirements.txt` installiert die Abhaengigkeiten.
- `python scripts/build_local_index.py` baut eine SQLite-DB unter `data/db/local_index.sqlite` aus `data/raw/`.
- `python -m pytest` fuehrt die Tests in `tests/` aus.
- Weitere workflow-spezifische Kommandos stehen in `README.md` und den passenden Dateien unter `docs/`.

## Coding-Style & Namenskonventionen

- Python-Code nutzt 4 Leerzeichen Einrueckung und `snake_case.py` fuer Module.
- Klassennamen folgen `UpperCamelCase`; oeffentliche Funktionen sollen kurze Docstrings haben.
- Verzeichnisse nutzen `snake_case`.
- Datenordner folgen `YYYY-MM-DD_Gremium_ID/`, Tagesordnungspunkte `NN_kurztitel/`.
- Bevorzuge strukturiertes Logging und halte Laufzeitlogs unter `logs/`.

## Test-Richtlinien

- Tests liegen in `tests/` und heissen `test_*.py`.
- Nutze Fixtures aus `tests/fixtures/` beim Parsen von HTML.
- Fuege Tests fuer neue Parsing- oder Download-Edge-Cases hinzu; vor PRs `python -m pytest` laufen lassen.

## Commit- & Pull-Request-Richtlinien

- Commit-Messages sind kurz, im Imperativ und in Satzform; Englisch und Deutsch kommen vor.
- Agenten sollen regelmaessig kleine, zusammenhaengende Zwischen-Commits erstellen statt grosse Sammel-Commits am Ende.
- Spaetestens vor riskanteren Refactorings, groesseren Dateiedits oder laengeren Arbeitspaketen ist ein sauberer Zwischen-Commit anzulegen.
- PRs sollten die Aenderung, die Testschritte und Auswirkungen auf Daten/Schemata beschreiben.
- Verlinke zugehoerige Issues/Tasks, wenn verfuegbar, und nenne neue Skripte oder Konfigurationsupdates.

## Daten, Sicherheit & Konfiguration

- Keine Secrets, API-Keys oder grosse Download-Daten einchecken; stattdessen `*.template`-Beispiele.
- Rohdaten bleiben unveraendert unter `data/raw/`, abgeleitete Daten unter `data/processed/`.
- Aktualisiere `docs/`, wenn sich Datenformate oder Crawl-Verhalten aendern.
- Nach Aenderungen an GUI, Workflows, Exporten, Datenformaten oder Nutzerfuehrung muessen die betroffenen Dokumente geprueft und bei Bedarf aktualisiert werden, insbesondere `README.md`, aktive Dateien unter `docs/` sowie bei Legacy-Pfaden die passenden Dateien unter `docs/archive/`.
- Zeilenenden werden ueber `.gitattributes` und `.editorconfig` vereinheitlicht:
  - Quell- und Konfigurationsdateien (u. a. `*.py`, `*.json`, `*.yml`) werden im Repository mit `LF` gefuehrt.
  - Windows-Skripte (`*.bat`, `*.cmd`, `*.ps1`) werden mit `CRLF` gefuehrt.
  - Keine manuellen Massenumstellungen von Zeilenenden ohne expliziten Anlass.

## Voraussetzungen

- Python 3.11+ und pip sind fuer Skripte und Tests erforderlich.
- Git ist fuer Versionskontrolle und Zusammenarbeit erforderlich.
- Fuer optionale UI-Arbeiten wird Tkinter benoetigt; unter WSL `python3-tk` installieren.

## Versionspflege

- Das Projekt verwendet `Major.Minor.Patch`; die kanonische Versionsnummer liegt in `VERSION`.
- Solange das Projekt noch vor `1.0.0` liegt, gelten pragmatisch:
  - `0.x.0` fuer groessere Entwicklungsschritte oder inkompatiblere Meilensteine
  - `0.x.y` fuer Bugfixes, kleinere Erweiterungen und inkrementelle Verbesserungen
- Bei sichtbaren funktionalen Aenderungen, Meilensteinen oder Releases muss `VERSION` bewusst angepasst werden.

## Agent-spezifische Anweisungen (verpflichtend)

### Branch-Sicherheit

- NIEMALS auf Branch `main` arbeiten.
- Wenn der aktuelle Branch `main` ist, STOPPEN und zuerst einen neuen Branch erstellen.
- Das Branch-Praefix haengt vom ausfuehrenden Agent ab:
  - **Codex** (OpenAI Codex): Branch MUSS mit `codex/` beginnen.
  - **Claude** (Anthropic Claude Code): Branch MUSS mit `claude/` beginnen.
- Bevorzugte Muster fuer Codex:
  - codex/feature/*kurze-beschreibung*
  - codex/fix/*kurze-beschreibung*
  - codex/security/*kurze-beschreibung*
  - codex/refactor/*kurze-beschreibung*
  - codex/chore/*kurze-beschreibung*
- Bevorzugte Muster fuer Claude:
  - claude/feature/*kurze-beschreibung*
  - claude/fix/*kurze-beschreibung*
  - claude/security/*kurze-beschreibung*
  - claude/refactor/*kurze-beschreibung*
  - claude/chore/*kurze-beschreibung*
- Andere Coding-Agenten verwenden analog ihr eigenes Praefix (z. B. `gemini/`, `gpt/`), sodass immer erkennbar ist, welches System die Aenderungen vorgenommen hat.

### Aenderungsdisziplin

- Vermeide "nur-Format"-Commits (z. B. Zeilenenden/Whitespace), ausser auf Anfrage.
- Dateien unter `old/` nicht aendern, ausser explizit angefordert.

## Regeln zu Entfernen & Refactoring (verpflichtend)

- Python-Dateien (`*.py`) duerfen NICHT geloescht werden.
- Wenn eine Datei ungenutzt/obsolet wird oder beim Refactoring ersetzt wird:
  - Datei in `/old` verschieben statt loeschen.
  - Originaldateiname bleibt erhalten.
  - Inhalte der nach `/old` verschobenen Datei nicht aendern, ausser explizit angefordert.

- Typische Faelle:
  - Aufteilen einer grossen Datei in mehrere kleinere Module
  - Ersetzen einer Implementierung durch neue Architektur
  - Ablosen von Legacy-Logik

- Loeschen von Python-Dateien ist nur erlaubt, wenn der User explizit danach fragt.

## WSL-spezifische Python-Umgebung (optional)

- Wenn `python` fehlt, `python3` und `pip3` verwenden.
- Empfohlenes Setup:
  - Unter WSL die virtuelle Umgebung `.venv-wsl` verwenden.
  - Unter Windows die virtuelle Umgebung `.venv` verwenden.
  - WSL-Setup: `python3 -m venv .venv-wsl`
  - Aktivieren unter WSL: `source .venv-wsl/bin/activate`
  - `python -m pip install -r requirements.txt`
- Bei Paketinstallationen oder Testlaeufen unter WSL ggf. mit laengerem Timeout arbeiten.

---
> Source: [WehkingDiane/ratsi_melle](https://github.com/WehkingDiane/ratsi_melle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
