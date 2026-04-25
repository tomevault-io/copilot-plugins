## calsync-swift

> Dieses Dokument enthält Kontext und Richtlinien für Claude Code (AI-Assistent) bei der Arbeit an diesem Projekt.

# CLAUDE.md

Dieses Dokument enthält Kontext und Richtlinien für Claude Code (AI-Assistent) bei der Arbeit an diesem Projekt.

## Projektübersicht

CalSync ist eine native macOS Menubar-App zur Synchronisation von Platzhalter-Terminen zwischen mehreren Apple-Kalendern. Die App wurde von einer Python CLI-Implementierung portiert.

## Technologie-Stack

- **Sprache:** Swift 6.0
- **UI Framework:** SwiftUI
- **Target:** macOS 15.0+ (Tahoe)
- **Architektur:** MVVM mit Services
- **Build-System:** Xcode + xcodegen

## Wichtige Architektur-Entscheidungen

### Tracking-Marker-Format (KRITISCH)
Das Tracking-Marker-Format im Notizen-Feld **muss exakt** beibehalten werden für Kompatibilität mit der Python-Version:
```
[CALSYNC:{"tid":"...","src":"...","scal":"...","hash":"...","sstart":"..."}]
```

### Hash-Berechnung
Der Event-Hash wird aus folgenden Attributen berechnet (Reihenfolge wichtig!):
- `all_day`, `availability`, `end`, `participant_status`, `start`
- JSON mit `sort_keys=True`, dann SHA256, erste 16 Zeichen

### EventKit-Konstanten
```swift
// Availability
0 = Busy, 1 = Free, 2 = Tentative, 3 = Unavailable (OOO)

// Participant Status
1 = Pending, 2 = Accepted, 3 = Declined, 4 = Tentative
```

## Projektstruktur

```
CalSync/
├── CalSyncApp.swift              # @main Entry Point
├── Info.plist                    # LSUIElement=true (kein Dock-Icon)
├── CalSync.entitlements          # Kalender-Berechtigung
├── Models/
│   ├── CalendarEvent.swift       # Event-Datenmodell
│   ├── CalendarInfo.swift        # Kalender-Metadaten
│   ├── PlaceholderInfo.swift     # Tracking-Marker-Logik
│   └── SyncAction.swift          # Sync-Aktionen & Ergebnisse
├── Services/
│   ├── EventKitService.swift     # EventKit-Adapter
│   ├── SyncEngine.swift          # Sync-Orchestrierung
│   ├── ChangeDiffer.swift        # Änderungserkennung
│   ├── EventTracker.swift        # Hash-Berechnung
│   └── BackgroundSyncService.swift
├── ViewModels/
│   ├── AppState.swift            # Globaler App-Zustand
│   └── SettingsStore.swift       # UserDefaults-Wrapper
├── Views/
│   ├── MenuBarView.swift         # Menubar-Menü
│   └── Settings/
│       ├── SettingsWindow.swift
│       ├── GeneralSettingsView.swift
│       ├── CalendarsSettingsView.swift
│       └── AboutView.swift
└── Utilities/
    └── Constants.swift
```

## Build-Befehle

```bash
# Projekt generieren (nach Dateistruktur-Änderungen)
xcodegen generate

# Debug Build
xcodebuild -project CalSync.xcodeproj -scheme CalSync -configuration Debug build

# Release Build
xcodebuild -project CalSync.xcodeproj -scheme CalSync -configuration Release build

# App öffnen
open ~/Library/Developer/Xcode/DerivedData/CalSync-*/Build/Products/Release/CalSync.app
```

## Häufige Aufgaben

### Neuen Service hinzufügen
1. Swift-Datei in `CalSync/Services/` erstellen
2. `xcodegen generate` ausführen
3. In `AppState.swift` instanziieren falls global benötigt

### Settings erweitern
1. Property in `SettingsStore.swift` hinzufügen (mit UserDefaults-Key)
2. UI in `GeneralSettingsView.swift` oder neuen Tab erstellen

### Sync-Logik ändern
- `ChangeDiffer.swift` – Filterregeln, was synchronisiert wird
- `SyncEngine.swift` – Wie Platzhalter erstellt/aktualisiert werden
- `EventTracker.swift` – Hash-Berechnung (Vorsicht: Kompatibilität!)

## Bekannte Einschränkungen

- Kein App Store Signing (nur lokale Nutzung)
- Keine Sandbox (für EventKit-Zugriff ohne Einschränkungen)
- Nur arm64 (Apple Silicon)

## Referenz: Python-Original

Das ursprüngliche Python-Projekt liegt unter:
```
~/Projects/Personal/Python/CalSync/
```

Wichtige Dateien für Referenz:
- `src/calsync/sync/engine.py` – Original Sync-Logik
- `src/calsync/models/placeholder.py` – Tracking-Marker-Format
- `src/calsync/sync/tracker.py` – Hash-Berechnung

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skempken) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
