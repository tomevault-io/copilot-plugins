## toolbox

> Dieses Repository ist eine zentrale Sammlung von DevOps- und SysAdmin-Tools. Alle Scripts sind robust entwickelt und Enterprise-tauglich.

# Claude Code Anweisungen - Toolbox Repository

## 📋 Projekt-Kontext

Dieses Repository ist eine zentrale Sammlung von DevOps- und SysAdmin-Tools. Alle Scripts sind robust entwickelt und Enterprise-tauglich.

### Repository-Struktur
```
toolbox/
├── README.md                 # Haupt-Dokumentation
├── LICENSE                   # MIT-Lizenz
├── tool_name/
│   ├── tool_name            # Ausführbares Script (OHNE Dateiendung)
│   └── README.md            # Vollständige Tool-Dokumentation
└── CLAUDE.md                # Diese Datei
```

## 🛠️ Code-Standards

### Script-Organisation (ZWINGEND)
- **Eigener Ordner:** Jedes Script hat seinen eigenen Ordner
- **Bindestrich-Namen:** Tool-Namen verwenden Bindestriche (-), KEINE Unterstriche (_)
- **Keine Dateiendung:** Scripts haben KEINE Dateiendung (nicht .sh)
- **Executable:** Alle Scripts müssen ausführbar sein (`chmod +x`)
- **README.md:** Jedes Script hat ein README im gleichen Aufbau
- **Haupt-README:** Jedes Script ist im Haupt-README aufgeführt
- **Sicherheitshinweise:** Wo nötig in Haupt-README und Script-README

### Bash-Scripts (ZWINGEND)

#### Basis-Anforderungen
- **Shebang:** `#!/usr/bin/env bash` (IMMER erste Zeile)
- **Set-Optionen:** `set -euo pipefail` (IMMER nach Shebang und Kommentaren)
- **Script-Header-Kommentar:** Zweck und Verwendung kurz dokumentieren

#### Farb-Definitionen (STANDARD-PATTERN)
```bash
# Farben für Output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'  # No Color
```

#### Pflicht-Log-Funktionen (STANDARD-PATTERN)
```bash
log_info() {
    echo -e "${BLUE}[INFO]${NC} $1" >&2
}

log_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1" >&2
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1" >&2
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1" >&2
}

log_verbose() {
    if [[ "$VERBOSE" == "true" ]]; then
        echo -e "${BLUE}[VERBOSE]${NC} $1" >&2
    fi
}
```

**WICHTIG:** NIEMALS `echo` direkt für Log-Ausgaben verwenden! Immer log_* Funktionen nutzen.

#### Pflicht-Funktion: show_help()
```bash
show_help() {
    cat << EOF
Tool-Name - Kurzbeschreibung

VERWENDUNG:
    $0 [OPTIONEN]

BESCHREIBUNG:
    Ausführliche Beschreibung des Tools.

OPTIONEN:
    -v, --verbose            Detaillierte Ausgabe
    -n, --dry-run            Nur anzeigen was gemacht würde
    -f, --force              Überschreibe ohne Nachfrage
    -h, --help               Diese Hilfe anzeigen

UMGEBUNGSVARIABLEN:
    VAR_NAME                 Beschreibung der Variable

BEISPIELE:
    $0                       # Standard-Verwendung
    $0 --dry-run             # Testlauf ohne Änderungen
    $0 --verbose             # Mit detaillierter Ausgabe

VORAUSSETZUNGEN:
    - tool1 muss installiert sein
    - tool2 muss verfügbar sein

HINWEISE:
    - Wichtige Hinweise zur Verwendung
    - Sicherheitsaspekte
EOF
}
```

#### Pflicht-Funktion: check_dependencies()
```bash
check_dependencies() {
    local missing_deps=()

    if ! command -v tool1 &> /dev/null; then
        missing_deps+=("tool1")
    fi

    if ! command -v tool2 &> /dev/null; then
        missing_deps+=("tool2")
    fi

    if [[ ${#missing_deps[@]} -gt 0 ]]; then
        log_error "Fehlende Abhängigkeiten: ${missing_deps[*]}"
        log_info "Installation:"
        log_info "  macOS: brew install ${missing_deps[*]}"
        log_info "  Linux: apt install ${missing_deps[*]} oder yum install ${missing_deps[*]}"
        return 1
    fi

    return 0
}
```

#### Empfohlene Funktion: parse_arguments()
Für komplexe Scripts mit vielen Optionen:
```bash
parse_arguments() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -v|--verbose)
                VERBOSE=true
                shift
                ;;
            -n|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -f|--force)
                FORCE=true
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            *)
                log_error "Unbekannte Option: $1"
                show_help
                exit 1
                ;;
        esac
    done
}
```

#### Empfohlene Funktion: validate_parameters()
Für Parameter-Validierung nach dem Parsen:
```bash
validate_parameters() {
    if [[ -z "${REQUIRED_VAR:-}" ]]; then
        log_error "REQUIRED_VAR ist nicht gesetzt"
        return 1
    fi

    if [[ ! -d "${DIRECTORY:-}" ]]; then
        log_error "Verzeichnis existiert nicht: $DIRECTORY"
        return 1
    fi

    return 0
}
```

#### Standard-Optionen (Pflicht wo anwendbar)
- **`-h, --help`** - Hilfe anzeigen (PFLICHT für ALLE Scripts)
- **`-v, --verbose`** - Detaillierte Ausgabe (PFLICHT für komplexe Scripts)
- **`-n, --dry-run`** - Testlauf ohne Änderungen (PFLICHT für destruktive Operationen)
- **`-f, --force`** - Überschreibe ohne Nachfrage (OPTIONAL für destruktive Operationen)

#### Globale Variablen (Empfohlen)
```bash
# Globale Variablen am Script-Anfang
VERBOSE=false
DRY_RUN=false
FORCE=false
```

#### Statistik-Tracking (Optional für komplexe Scripts)
```bash
# Statistiken (optional)
STATS_TOTAL=0
STATS_SUCCESS=0
STATS_ERRORS=0
STATS_SKIPPED=0
```

#### Main-Funktion Pattern (Empfohlen)
```bash
main() {
    # 1. Abhängigkeiten prüfen
    if ! check_dependencies; then
        exit 1
    fi

    # 2. Parameter parsen
    parse_arguments "$@"

    # 3. Parameter validieren
    if ! validate_parameters; then
        show_help
        exit 1
    fi

    # 4. Hauptlogik
    # ... Script-Funktionalität ...

    # 5. Abschluss/Statistiken
    log_success "Fertig!"
}

# Script-Ausführung
main "$@"
```

#### Weitere Best Practices
- **Backup-Erstellung:** Vor Änderungen an wichtigen Dateien
- **Destructive Operations:** Immer Bestätigung verlangen, ausser Force-Modus aktiv
- **Exit-Codes:** 0 = Erfolg, 1 = Allgemeiner Fehler, 2+ = Spezifische Fehler
- **Temporäre Dateien:** Mit `trap` automatisch löschen (`trap 'rm -rf "$TMPDIR"' EXIT`)
- **Error-Messages:** Immer auf stderr (`>&2`) ausgeben

### Python-Scripts
- **Shebang:** `#!/usr/bin/env python3`
- **Imports:** Alle System-Imports am Anfang der Datei
- **Error-Handling:** Try-catch mit aussagekräftigen Fehlermeldungen
- **Argument-Parsing:** `argparse` mit ausführlicher Hilfe

### Portabilität (ZWINGEND)
- **macOS/Linux Kompatibilität:** Alle Scripts müssen auf beiden Systemen funktionieren
- **BSD/GNU Tool-Unterschiede:** Berücksichtigen und testen (z.B. `stat`, `sed -i`)
- **Fallback-Funktionen:** Für externe Tools (z.B. `numfmt`, `dig`, `gsed`)
- **OS-Erkennung:** `uname` oder ähnliches für systemspezifische Pfade
- **GNU-Tools auf macOS:** Prüfung auf Brew-installierte Versionen (gsed, gawk, etc.)

## 📚 Dokumentations-Standards

### Deutsche Dokumentation (Schweiz-tauglich)
- **Alle README-Dateien auf Deutsch** mit Schweizer Rechtschreibung
- **Keine ß-Zeichen:** Immer "ss" verwenden (z.B. "muss" statt "muß")
- **Deutsche Kommentare** in Scripts (Schweiz-tauglich)
- **Deutsche Log-Ausgaben** (Schweiz-tauglich)
- **Englische Variablen-/Funktionsnamen** sind OK

### README.md Struktur (pro Tool)
1. **Titel & Kurzbeschreibung**
2. **🎯 Zweck** - Was macht das Tool?
3. **📋 Voraussetzungen** - Dependencies
4. **🚀 Installation** - Download-Anweisungen
5. **💻 Verwendung** - Syntax und Optionen
6. **📄 Ausgabe-Beispiel** - Beispiel-Output
7. **🔧 Funktionsweise** - Wie es funktioniert
8. **🏢 Anwendungsfälle** - Praktische Beispiele
9. **⚠️ Hinweise** - Sicherheit, Performance
10. **🔍 Troubleshooting** - Häufige Probleme
11. **💡 Tipps & Tricks** - Best Practices
12. **📚 Verwandte Tools** - Ähnliche/ergänzende Tools

### Beispiel-Code in README
- Immer mit `bash` oder entsprechender Sprache taggen
- Vollständige, ausführbare Beispiele
- Kommentare für komplexe Befehle

## 🔒 Sicherheits-Richtlinien

### Sensible Daten
- **Keine SSH-Keys, Passwörter oder Tokens** in Code oder Dokumentation
- **Beispiel-Konfigurationen** mit Platzhaltern
- **Warnungen** bei Scripts die Systemänderungen vornehmen

### Berechtigungen
- **Sudo-Anforderungen** klar dokumentieren
- **Minimal-Rechte-Prinzip** befolgen
- **Validierung** von Benutzereingaben

## 🧪 Test-Standards

### Manueller Test-Workflow
```bash
# Syntax-Check für alle Bash-Scripts
find . -name "*.sh" -exec bash -n {} \;

# ShellCheck (falls installiert)
find . -name "*.sh" -exec shellcheck {} \;

# Dry-Run Tests (wenn verfügbar)
./tool_name --dry-run --verbose
```

### Funktionalitäts-Sicherstellung
- **Rückwärts-Kompatibilität** bei Änderungen an bestehenden Tools
- **Erweiterte Features** sind erlaubt und erwünscht
- **Breaking Changes** nur nach ausdrücklicher Genehmigung

## 📦 Tool-spezifische Hinweise

### brewfile-commenter.sh
- Backup-Erstellung ist kritisch
- jq-Abhängigkeit prüfen
- Homebrew-API-Aufrufe können fehlschlagen

### crane_fqdn.sh
- crane-Binary erforderlich
- Netzwerk-Zugriff zu Registries nötig
- Temporäre Dateien automatisch löschen

### fix-perms.sh
- NUR auf macOS verwenden
- Sudo-Rechte erforderlich
- Backup-System ist essentiell
- SIP-Beschränkungen beachten

### fix-ssh-key
- SSH-Tools müssen verfügbar sein
- DNS-Auflösung optional aber hilfreich
- ~/.ssh/known_hosts Backup empfehlen

### k8s_vuln.sh
- trivy und kubectl Dependencies
- Kubernetes-Cluster-Zugriff erforderlich
- Performance bei grossen Clustern beachten

### serve_this
- Python 3.6+ erforderlich
- OpenSSL für HTTPS
- Selbstsignierte Zertifikate = Browser-Warnungen

### udm_backup
- SSH-Zugriff zur UniFi Dream Machine
- jq für JSON-Verarbeitung
- SCP-Transfer kann bei grossen Backups Zeit brauchen

## 🚀 Development-Workflow

### Bei Script-Änderungen
1. **Bestehende Funktionalität** vollständig verstehen
2. **Dry-Run Tests** implementieren/verwenden
3. **Error-Handling** für alle kritischen Operationen
4. **README.md** entsprechend aktualisieren
5. **Haupt-README.md** bei strukturellen Änderungen anpassen

### Bei neuen Tools (ZWINGEND - ALLE Schritte erforderlich!)
1. **Eigenen Ordner** erstellen: `tool-name/` (mit Bindestrichen!)
2. **Ausführbares Script:** `tool-name` (OHNE .sh Endung!)
3. **Script executable machen:** `chmod +x tool-name/tool-name`
4. **ZWINGEND: README.md erstellen** nach obiger Struktur im tool-name/ Ordner
5. **ZWINGEND: Haupt-README.md erweitern:**
   - Tool in "Verfügbare Tools" Sektion hinzufügen
   - Beispiel in "Schnellstart" Sektion hinzufügen
   - Eintrag in "Tool-Status" Tabelle hinzufügen
   - Abhängigkeiten in "Abhängigkeiten" Tabelle hinzufügen
6. **Sicherheitshinweise** hinzufügen (Haupt-README und Tool-README)
7. **macOS/Linux Kompatibilität** testen und sicherstellen

⚠️ **WICHTIG:** Ohne vollständige README.md (sowohl Tool-README als auch Haupt-README-Updates) ist ein Tool NICHT vollständig und darf nicht als fertig betrachtet werden!

## ⚠️ Wichtige Beachtungen

### Niemals ändern
- **LICENSE** Datei - MIT-Lizenz beibehalten
- **Kern-Funktionalität** bestehender Tools ohne Genehmigung

### Immer prüfen
- **Bindestrich-Namen** für Tools (nicht Unterstriche!)
- **Keine Dateiendungen** bei Scripts (nicht .sh!)
- **Executable-Rechte** gesetzt (`chmod +x`)
- **Portable Shebang-Zeilen** (`#!/usr/bin/env bash`)
- **macOS/Linux Kompatibilität** getestet
- **Schweizer Rechtschreibung** (kein ß)
- **Destructive Operations** mit Confirmation-Dialog
- **Error-Codes** für CI/CD-Integration
- **Help/Usage-Ausgaben** vollständig und korrekt
- **Beispiele** in README funktionieren wirklich
- **Sicherheitshinweise** wo erforderlich

### Performance-Überlegungen
- **Grosse Cluster/Dateien** - Timeout- und Progress-Mechanismen
- **Netzwerk-Operations** - Retry-Logic und Timeouts
- **Parallelisierung** wo sinnvoll (aber dokumentiert)

## 🎯 Qualitätskriterien

Ein Tool gilt als "✅ Ready" wenn:
- ✅ **Eigener Ordner** erstellt (mit Bindestrichen!)
- ✅ **Bindestrich-Namen** verwendet (keine Unterstriche!)
- ✅ **Keine Dateiendung** (.sh entfernt)
- ✅ **Executable-Rechte** gesetzt
- ✅ **macOS/Linux kompatibel** (getestet)
- ✅ **Schweizer Rechtschreibung** (kein ß)
- ✅ **Tool-README.md** vollständig nach Standard-Struktur erstellt
- ✅ **Haupt-README** komplett aktualisiert:
  - ✅ Tool in "Verfügbare Tools" aufgeführt
  - ✅ Beispiel in "Schnellstart" hinzugefügt
  - ✅ Eintrag in "Tool-Status" Tabelle
  - ✅ Abhängigkeiten in "Abhängigkeiten" Tabelle
- ✅ **Sicherheitshinweise** hinzugefügt (wo nötig)
- ✅ **Error-Handling** implementiert
- ✅ **Dry-Run Modus** verfügbar (`--dry-run`, `-n`)
- ✅ **Confirmation-Dialogs** für destructive Operations
- ✅ **Help-System** funktional (`--help`, `-h`)
- ✅ **Portable Implementation** (BSD/GNU-Tools berücksichtigt)
- ✅ **Beispiele getestet** und funktional

🚨 **KRITISCH:** Ohne vollständige Dokumentation (Tool-README + Haupt-README Updates) ist ein Tool NICHT fertig!

---

**Letzte Aktualisierung:** 2025-10-22
**Version:** 1.3 - Vollständige Bash-Script Standards definiert (Log-Funktionen, show_help, check_dependencies, parse_arguments, validate_parameters, Standard-Optionen)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tuxpeople) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
