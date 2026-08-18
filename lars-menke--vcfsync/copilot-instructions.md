## vcfsync

> Ein Python-Projekt, das iCloud-Kontakte via CardDAV exportiert und importiert,

# iCloud Contacts Sync — Projektbeschreibung für Claude Code

## Ziel

Ein Python-Projekt, das iCloud-Kontakte via CardDAV exportiert und importiert,
manuell per GitHub Actions Workflow im Browser gestartet werden kann,
und die exportierte VCF-Datei als downloadbares Artifact bereitstellt.

---

## Projektstruktur (zu erstellen)

```
icloud-contacts/
├── CLAUDE.md                  # diese Datei
├── README.md                  # Kurzdoku für den Nutzer
├── requirements.txt           # Python-Abhängigkeiten
├── icloud_contacts.py         # Hauptskript (Export + Import)
└── .github/
    └── workflows/
        ├── export.yml         # Manueller Export-Workflow
        └── import.yml         # Manueller Import-Workflow
```

---

## Hauptskript: icloud_contacts.py

Das Skript existiert bereits und soll 1:1 übernommen werden.
Unten ist der vollständige Code — bitte exakt so in `icloud_contacts.py` schreiben.

### Funktionsweise

- Verbindet sich per CardDAV gegen `https://contacts.icloud.com`
- Authentifizierung via HTTP Basic Auth (Apple-ID + app-spezifisches Passwort)
- Ermittelt automatisch die Principal-URL und Adressbuch-URL des Nutzers
  (iCloud vergibt pro Account eine individuelle Nummer, z.B. `/12345678/`)

**Export:**
1. PROPFIND auf das Adressbuch → Liste aller Kontakt-URLs
2. GET auf jede einzelne URL → vollständige vCard inkl. `PHOTO;ENCODING=b;TYPE=JPEG:...`
3. Alle vCards werden in eine einzelne `.vcf`-Datei geschrieben

**Import:**
1. Vorhandene Kontakte aus iCloud laden → UID-Map aufbauen `{uid: href}`
2. vCards aus der Import-Datei einlesen und UIDs extrahieren
3. Pro Kontakt: UID in Map vorhanden → PUT auf bestehende URL (Update, kein Duplikat)
4. UID nicht vorhanden → PUT auf neue URL (Neuanlage)
5. Optional: `--dry-run` simuliert ohne zu schreiben

### Abhängigkeiten

```
requests>=2.31.0
```

Nur `requests` als externe Abhängigkeit, alles andere ist Python-Standardbibliothek.

### CLI-Interface

```bash
# Export
python icloud_contacts.py export --output meine_kontakte.vcf

# Import
python icloud_contacts.py import --input bearbeitete_kontakte.vcf

# Import simulieren
python icloud_contacts.py import --input bearbeitete_kontakte.vcf --dry-run
```

### Zugangsdaten

Im GitHub Actions Kontext kommen die Zugangsdaten aus Secrets:
- `ICLOUD_USER` — Apple-ID (E-Mail-Adresse)
- `ICLOUD_PASS` — App-spezifisches Passwort (nicht das iCloud-Hauptpasswort)

Lokal liest das Skript alternativ aus einer `.env`-Datei (KEY=VALUE) oder fragt interaktiv ab.

---

## GitHub Actions Workflows

### Export-Workflow: .github/workflows/export.yml

Anforderungen:
- Trigger: `workflow_dispatch` (manuell im Browser startbar, kein Zeitplan)
- Runner: `ubuntu-latest`
- Python: 3.11
- Schritte:
  1. Checkout
  2. Python setup + `pip install -r requirements.txt`
  3. Skript ausführen: `python icloud_contacts.py export --output icloud_kontakte.vcf`
  4. Zugangsdaten aus GitHub Secrets: `ICLOUD_USER` und `ICLOUD_PASS` als Umgebungsvariablen
  5. Exportierte VCF hochladen als GitHub Actions Artifact
     - Name: `icloud-kontakte-export`
     - Datei: `icloud_kontakte.vcf`
     - Retention: 7 Tage (danach automatisch gelöscht, aus Datenschutzgründen)

### Import-Workflow: .github/workflows/import.yml

Anforderungen:
- Trigger: `workflow_dispatch` mit einem Input-Parameter:
  - `vcf_content`: Textarea, in die der Nutzer den VCF-Inhalt einfügen kann
    (Typ: string, required: true)
- Runner: `ubuntu-latest`
- Python: 3.11
- Schritte:
  1. Checkout
  2. Python setup + `pip install -r requirements.txt`
  3. VCF-Inhalt aus dem Input in eine Datei schreiben: `import_input.vcf`
  4. Skript ausführen: `python icloud_contacts.py import --input import_input.vcf`
  5. Zugangsdaten aus GitHub Secrets: `ICLOUD_USER` und `ICLOUD_PASS`

Hinweis zum Import-Workflow: Da GitHub Actions keine Datei-Uploads als
Workflow-Input unterstützt, wird der VCF-Inhalt als Text in eine Textarea
eingefügt. Bei 437 Kontakten mit Base64-Fotos kann das sehr groß werden —
der Input ist auf 65.536 Zeichen limitiert. Einen Hinweis im README ergänzen,
dass für große Dateien ein lokaler Aufruf des Skripts empfohlen wird.

---

## README.md (zu erstellen)

Soll folgende Abschnitte enthalten:

### Einrichtung
1. Repository als **privat** anlegen (Kontaktdaten sind sensibel)
2. GitHub Secrets setzen:
   - `Settings` → `Secrets and variables` → `Actions` → `New repository secret`
   - `ICLOUD_USER` = Apple-ID E-Mail
   - `ICLOUD_PASS` = App-spezifisches Passwort
     (erstellen auf appleid.apple.com → Anmeldung und Sicherheit → App-spezifische Passwörter)

### Export ausführen
1. GitHub → Tab `Actions` → `iCloud Kontakte Export`
2. Klick auf `Run workflow` → `Run workflow`
3. Nach ~2-3 Minuten: Workflow-Run anklicken → unter `Artifacts` die VCF herunterladen
4. Artifact ist 7 Tage verfügbar, danach automatisch gelöscht

### Import ausführen
1. VCF-Inhalt in die Zwischenablage kopieren
2. GitHub → Tab `Actions` → `iCloud Kontakte Import`
3. Klick auf `Run workflow` → VCF-Inhalt in das Textfeld einfügen → `Run workflow`
4. Bei großen Dateien (>65.000 Zeichen): Skript lokal ausführen (siehe unten)

### Lokale Nutzung
```bash
pip install requests
python icloud_contacts.py export --output meine_kontakte.vcf
python icloud_contacts.py import --input bearbeitete_kontakte.vcf
```

### Datenschutzhinweis
- Repository unbedingt **privat** halten
- Zugangsdaten niemals direkt in Code oder Workflow-Dateien schreiben
- Exportierte Artifacts enthalten alle Kontaktdaten inkl. Fotos
- Artifacts werden nach 7 Tagen automatisch gelöscht

---

## Hinweise für Claude Code

- Das Skript `icloud_contacts.py` ist bereits fertig — bitte den vorhandenen
  Code aus der Datei verwenden, nicht neu schreiben
- Workflows sollen funktional und minimal sein, keine unnötigen Schritte
- Alle Secrets werden ausschließlich als Umgebungsvariablen übergeben,
  niemals als Workflow-Input oder in Log-Ausgaben
- Repository soll `.gitignore` enthalten: `.env`, `*.vcf`, `__pycache__/`

---
> Source: [lars-menke/VCFSync](https://github.com/lars-menke/VCFSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
