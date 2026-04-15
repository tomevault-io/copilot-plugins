## tilevisu-bewohner-status-kachel

> *Version 1.1 • 19 Mai 2025*


# Windsurf.io – Regeln für IP‑Symcon‑PHP‑Module

*Version 1.1 • 19 Mai 2025*

> **Ziel:** Schnell nachschlagbares Regelwerk (≤ 6 000 Zeichen) für alle Windsurf‑Entwickler.

## 1 · Grundprinzipien

* **SDK‑Pflicht** Alle Module halten sich strikt an die offizielle [IP‑Symcon PHP‑SDK‑Doku](https://symcon.de/…/sdk-php/).
* **Open Source** Code liegt unter MIT‑Lizenz auf GitHub; jeder Commit braucht eine aussagekräftige Nachricht (DE/EN).
* **SemVer** `MAJOR.MINOR.PATCH` + Auto‑Build.

## 2 · Systemvoraussetzungen

* IP‑Symcon ≥ 4.0, interne PHP‑Version (aktuell 8.2).

## 3 · Projektstruktur

```text
Bibliothek/
  library.json  README.md
  <Modul>/
    module.php  module.json  form.json  locale.json
  libs/  docs/  imgs/  tests/  actions/
```

Klassennamen ≙ Ordnernamen; Punkt‑Ordner (z. B. .github) sind erlaubt.

## 4 · library.json (Pflichtfelder)

`id`, `author:"Windsurf.io"`, `name`, `version`, optional: `url`, `build`, `date`, `compatibility`.

## 5 · module.json (Pflichtfelder)

`id`, `name`, `type`, `prefix`, optional: `vendor:"Windsurf.io"`, `url`, `parentRequirements`, `childRequirements`, `implemented`, `aliases`.

## 6 · Klassenrichtlinien

1. `class <Name> extends IPSModule`
2. **Create()** → RegisterProperty\*, RegisterVariable\*, Timer, Buffer
3. **ApplyChanges()** → Schnittstellen binden, Status setzen
4. parent::Create/ApplyChanges **nie** löschen
5. Statuscodes (102 OK, 104 Fehler …) konsequent.

## 7 · IP‑Symcon‑APIs zuerst

* Keine eigenen Listener/Server, solange Symcon etwas anbietet.
* **WebSocket:** `ws(s)://<host>:3777/api/` + Auth‑Header statt Eigenlösung.
* Fehlende Features dürfen nur nach Review + Dokuhinweis selbst implementiert werden.

## 8 · Coding‑Standards

* `declare(strict_types=1);`, PSR‑12, PSR‑4 (Composer).
* Namensräume: `Windsurf\IPSymcon\<Lib>`
* Exceptions statt `echo/die`; Magic Numbers → `const`/`enum`.
* Tools: phpcs, phpstan L8.

## 9 · Internationalisierung

* **locale.json Pflicht** – *alle* im Backend/Frontend sichtbaren Texte **zweisprachig** (DE & EN).
* Keine hard‑codierten Strings in PHP oder form.json.

## 10 · Konfigurationsformular

`form.json` nutzt Panels, Expander, Actions (`onClick`, `onChange`). Dynamische Änderungen über `ReloadForm()`, `UpdateFormField()`.

## 11 · Logging & Debugging

* Entwicklungs‑Logs → `SendDebug()` (Schaltbar).
* Fehler → `LogMessage(..., IPS_LOG_ERR)`; sensible Daten maskieren.

## 12 · Tests & CI

* **tests/** → PHPUnit 10, Integration via IP‑Symcon‑Headless (Docker).
* GitHub Actions: lint, phpcs, phpstan, phpunit, semantic‑release.

## 13 · Versionierung & Release

* Releases via GitHub + ZIP für IP‑Symcon‑Store; ChangeLog automatisiert.

## 14 · Dokumentation

* `README.md` (Funktions‑Überblick, Installation, Beispiele, Statuscodes).
* Zusätzliche Handbücher optional in **docs/**.

## 15 · Qualitätssicherung

* Pull‑Requests & Code‑Review **Pflicht**.
* CI muss grün sein; Test‑Coverage ≥ 90 %, neue Komponenten 100 %.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/da8ter) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
