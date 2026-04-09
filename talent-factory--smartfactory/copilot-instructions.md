## smartfactory

> ______________________________________________________________________

______________________________________________________________________

## trigger: manual

# Windsurf Rule: Commit

Dieser Befehl hilft dabei, professionelle Git-Commits mit automatischen
Qualitätschecks und konventionellen Commit-Nachrichten zu erstellen.

Bitte darauf achten, dass der komplette Commit in Deutsch verfasst wird.

## Verwendung

Für einen Standard-Commit:

```bash
/commit
```

Mit Optionen:

```bash
/commit --no-verify     # Überspringt Pre-Commit-Checks
/commit --force-push    # Führt force push aus (Vorsicht!)
/commit --skip-tests    # Überspringt Testausführung
```

## Funktionalität

1. **Automatische Pre-Commit-Checks** (ausser mit `--no-verify`):

   - **Java-Projekte**: Maven/Gradle Builds, Checkstyle, SpotBugs
   - **Python-Projekte**: Ruff/Flake8 Linting, Black Formatierung, pytest
   - **React/Node.js-Projekte**: ESLint, Prettier, TypeScript-Checks,
     Jest/Vitest Tests
   - **Dokumentation**: LaTeX-Kompilierung, Markdown-Validierung,
     AsciiDoc-Rendering

1. **Intelligente Staging-Verwaltung**:

   - Prüft gestakte Dateien mit `git status`
   - Fügt automatisch alle Änderungen hinzu, falls nichts gestakt ist
   - Zeigt Übersicht der zu committenden Änderungen

1. **Diff-Analyse und Commit-Optimierung**:

   - Analysiert `git diff` um Änderungsumfang zu verstehen
   - Erkennt mehrere logische Änderungen und schlägt Aufteilung vor
   - Erstellt atomare Commits für bessere Git-Historie

1. **Konventionelle Commit-Nachrichten**:

   - Verwendet Emoji Conventional Commit Format
   - Automatische Typerkennung basierend auf Änderungen
   - Deutsche und englische Beschreibungen möglich

## Unterstützte Projekttypen

### Java-Projekte

- **Maven**: `mvn compile`, `mvn test`, `mvn checkstyle:check`
- **Gradle**: `./gradlew build`, `./gradlew test`, `./gradlew checkstyleMain`
- **Spring Boot**: Automatische Erkennung und spezifische Checks

### Python-Projekte

- **Linting**: Ruff, Flake8, Pylint
- **Formatierung**: Black, isort
- **Type Checking**: mypy
- **Tests**: pytest, unittest
- **Dependencies**: Poetry, pip-tools, requirements.txt

### React/Node.js-Projekte

- **Package Manager**: npm, pnpm, yarn, bun
- **Linting**: ESLint, TSLint
- **Formatierung**: Prettier
- **Type Checking**: TypeScript Compiler
- **Tests**: Jest, Vitest, Cypress
- **Build**: Vite, Webpack, Next.js

### Dokumentationsprojekte

- **LaTeX**: pdflatex, xelatex Kompilierung
- **Markdown**: markdownlint, Links-Validierung
- **AsciiDoc**: asciidoctor Rendering

## Commit-Typen mit Emojis

- ✨ `feat`: Neue Funktionalität
- 🐛 `fix`: Fehlerbehebung
- 📚 `docs`: Dokumentationsänderungen
- 💎 `style`: Code-Formatierung (keine Logikänderung)
- ♻️ `refactor`: Code-Umstrukturierung ohne neue Features oder Fixes
- ⚡ `perf`: Performance-Verbesserungen
- 🧪 `test`: Tests hinzufügen oder korrigieren
- 🔧 `chore`: Build-Prozess, Tools, Konfiguration
- 🚀 `ci`: Continuous Integration Änderungen
- 🔒 `security`: Sicherheitsverbesserungen
- 🌐 `i18n`: Internationalisierung
- ♿ `a11y`: Barrierefreiheit
- 📦 `deps`: Dependency-Updates

## Best Practices

### Commit-Qualität

- **Atomare Commits**: Jeder Commit sollte eine logische Einheit darstellen
- **Aussagekräftige Nachrichten**: Beschreibe das "Was" und "Warum"
- **Imperative Form**: "Füge Feature hinzu" statt "Feature hinzugefügt"
- **Erste Zeile ≤ 72 Zeichen**: Für bessere Lesbarkeit in Git-Tools

### Code-Qualität vor Commit

- **Linting bestanden**: Code folgt Projektstandards
- **Tests erfolgreich**: Alle Tests laufen durch
- **Build erfolgreich**: Projekt kompiliert ohne Fehler
- **Dokumentation aktuell**: README, Kommentare, Docs sind auf dem neuesten
  Stand

### Projektspezifische Checks

- **Java**: Keine Compiler-Warnungen, Checkstyle-Konformität
- **Python**: PEP 8 Compliance, Type Hints wo möglich
- **React**: Keine ESLint-Fehler, Komponenten-Tests vorhanden

## Beispiel-Workflow

1. **Automatische Erkennung**: Projekttyp wird automatisch erkannt
1. **Pre-Commit-Checks**: Entsprechende Tools werden ausgeführt
1. **Staging-Analyse**: Zeigt zu committende Dateien
1. **Diff-Review**: Analysiert Änderungen und schlägt Commit-Struktur vor
1. **Commit-Erstellung**: Generiert professionelle Commit-Nachricht
1. **Push-Option**: Bietet automatischen Push zum Remote-Repository

## Fehlerbehebung

- **Build-Fehler**: Commit wird abgebrochen, Fehler werden angezeigt
- **Test-Fehler**: Option zum Überspringen mit `--skip-tests`
- **Linting-Probleme**: Automatische Fixes wo möglich, sonst Abbruch
- **Merge-Konflikte**: Warnung und Anleitung zur Auflösung

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/talent-factory)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/talent-factory)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
