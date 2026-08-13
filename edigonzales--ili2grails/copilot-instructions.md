## ili2grails

> - Der Metadata-Reader liefert Metadaten für INTERLIS/ili2db.

# AGENTS.md – Arbeitsanweisungen für dieses Repo

## Projektkontext (kurz)
- Der Metadata-Reader liefert Metadaten für INTERLIS/ili2db.
- Der zentrale Architektur-Entscheid: **Hybrid-Ansatz (ili2db + ili2c)**.
  - *Warum:* ili2db gibt exakte Tabellen-/Spaltennamen und Mappings (ohne Modellkompilierung), ili2c liefert Semantik (Constraints, Doku, Labels, Units, Enums). Die Kombination erzeugt ein vollständiges, konsistentes Metamodell für spätere Code-Generatoren.
- Das **Metamodell ist framework-agnostisch** (Grails, Spring, etc.), um Wiederverwendung zu ermöglichen.
- Dokumentation wurde **in der README.md konsolidiert**, damit Entscheidungen, Nutzung und Architektur an einer Stelle gepflegt werden.

## Java Best Practices (bitte befolgen)
### Code-Qualität
- **Small, focused classes**: Klassen nur mit klarer Verantwortung (SRP).
- **Immutable wo möglich**: Felder `final` setzen, unveränderliche Collections nutzen.
- **Null-Sicherheit**: Eingaben validieren (`Objects.requireNonNull`), `Optional` sparsam und semantisch sinnvoll verwenden.
- **Defensive Programming**: Externe Daten (DB/Modelle) validieren, Fehlerfälle explizit behandeln.

### API-Design
- **Stabile öffentliche APIs**: Signaturen kompatibel halten, Breaking Changes vermeiden.
- **Sinnvolle Namen**: Klassen/Methoden sprechen die Domäne (INTERLIS/ili2db) und nicht die Implementierung.
- **Trennung von IO und Logik**: Parsing/Mapping von Datenquellen getrennt von Berechnung/Transformation.

### Fehlerbehandlung & Logging
- **Aussagekräftige Exceptions**: Fehlerursache und Kontext angeben (Modellname, Tabelle, Attribut).
- **Keine silent failures**: Fehler sollen nachvollziehbar sein.
- **Log-Level bewusst wählen**: INFO für Workflow, DEBUG für Details, WARN/ERROR für Probleme.

### Testing
- **Tests für Daten-Mapping**: besonders bei Typ-Inferenz und Beziehungen.
- **Reproduzierbare Testdaten**: Models/DB-Schema versionieren und dokumentieren.

### Dokumentation
- **README ist die Wahrheit**: Architekturentscheidungen, Nutzeranleitung, Beispiele dort pflegen.
- **Keine Duplikate**: Andere Markdown-Dateien nur als Kurzreferenz.

## Wenn du Änderungen machst
- Prüfe, ob die Änderung eine Entscheidung betrifft → in README dokumentieren.
- Bei Änderungen am Metamodell: Beispiele/Mapping-Tabellen aktualisieren.

---
> Source: [edigonzales/ili2grails](https://github.com/edigonzales/ili2grails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
