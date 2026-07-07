## lm-visualizer-mac

> Guide pour travailler sur ce dépôt avec Claude Code.

# CLAUDE.md

Guide pour travailler sur ce dépôt avec Claude Code.

## Le projet

**LM Visualizer** — app macOS native (Swift / SwiftUI) qui visualise l'activité
de **LM Studio**. Chaque requête reçue par LM Studio anime un réseau de neurones
et affiche les stats d'inférence (tok/s, TTFT, tokens, durée) ainsi que les stats
machine (CPU / RAM / GPU).

Point clé : **pas de proxy**. L'app lit les logs de LM Studio via son CLI
(`lms log stream --json --stats`). Toutes les requêtes reçues sur `:1234` sont
visualisées, quelle que soit leur source — aucun client à reconfigurer.

## Build & lancement

Le `.xcodeproj` est **généré par [XcodeGen](https://github.com/yonaskolb/XcodeGen)**
à partir de `project.yml` (source de vérité). Il n'est pas versionné.

```bash
xcodegen generate                 # après modif de project.yml (brew install xcodegen)
open "LM Visualizer.xcodeproj"     # puis ⌘R dans Xcode (schéma LMVisualizer, My Mac)
```

En CLI :
```bash
xcodebuild -project "LM Visualizer.xcodeproj" -scheme LMVisualizer -configuration Release build
```

Les nouveaux fichiers `.swift` de `LMVisualizer/` sont pris automatiquement par
XcodeGen — pas besoin de les ajouter manuellement au projet.

## Architecture

```
Clients (Cursor, curl, script…) ──▶ LM Studio :1234
                                          │ logs
                        lms log stream --json --stats
                                          │
                                   LogStreamReader ──▶ Monitor ──▶ vues SwiftUI
```

| Fichier (`LMVisualizer/`) | Rôle |
|---|---|
| `LMVisualizerApp.swift` | `@main` : `MenuBarExtra` + `Window`, `AppDelegate`, modes `--snapshot`/`--menu`/`--icon`. |
| `Monitor.swift` | État central `@Observable @MainActor` (stats, journal, sortie). Singleton `Monitor.shared`. |
| `LogStreamReader.swift` | Lance et parse `lms log stream` (appariement début→fin FIFO). |
| `SystemStats.swift` | CPU/RAM via Mach (`host_statistics`), GPU via `ioreg`. |
| `GridEngine.swift` | Simulation + rendu de l'animation du réseau de neurones. |
| `Views.swift` | Vues SwiftUI (`DashboardView`, `MenuBarView`, `IconView`), `#Preview`, et l'enum `Snapshot`. |
| `Theme.swift` | Palette de couleurs (reprise de la version web). |

## Conventions

- **Langue** : tout le code, les commentaires et l'UI sont en **français** (accents
  inclus). Conserve ce style.
- **Concurrence** : `Monitor` est `@MainActor`. Les lectures système et le parsing
  de logs reviennent toujours sur le main actor avant de toucher l'état.
- **Couleurs** : passe par `Theme.*`, jamais de couleur en dur dans les vues.
- L'app n'est **pas sandboxée** (elle lance `lms`, `ioreg`, `vm_stat`) — voir
  `project.yml`. Signature ad-hoc pour l'usage local.

## Captures d'écran

Le rendu du tableau de bord, de la barre de menus et de l'icône se fait hors écran
via des flags CLI (**build DEBUG uniquement** — le code est derrière `#if DEBUG`) :

```bash
EXE="$(find ~/Library/Developer/Xcode/DerivedData -name 'LM Visualizer' -path '*Products/Debug*' -type f | head -1)"
"$EXE" --snapshot docs/screenshots/dashboard.png   # tableau de bord (données d'exemple)
"$EXE" --menu     docs/screenshots/menubar.png      # panneau de la barre de menus
"$EXE" --icon     /tmp/icon.png                     # master d'icône 1024
```

Ces modes utilisent `Monitor.previewSample()` : aucun LM Studio requis. Régénère
les captures après une modification visible de l'UI.

## Icône

Dessinée en SwiftUI par `IconView` (`Views.swift`). Pour régénérer l'appiconset
après modification, voir la section « Icône » du `README.md`.

---
> Source: [Vincweb/lm-visualizer-mac](https://github.com/Vincweb/lm-visualizer-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
