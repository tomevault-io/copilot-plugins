## two-channel-workflow

> Zwei getrennte GameHelper-Quellprojekte (Stabil vs Experimental)


# GameHelper: Stabil vs Experimental

Es gibt **zwei getrennte Quellprojekte** — behandle sie wie zwei Programme.

| Kanal | Quellcode | Wann bearbeiten |
|-------|-----------|-----------------|
| **Experimental** | `D:\ZusatzProgramme\Gamehelper-Experimental-Src` | Neue Features, Tests, Plugin-Experimente |
| **Stabil** | `D:\ZusatzProgramme\Gamehelper` | Nur Fixes/Releases fuer alle Nutzer |

## Regeln fuer den Agenten

1. **Standard bei neuen Features / Plugin-Arbeit:** nur `Gamehelper-Experimental-Src` aendern.
2. **Nicht** automatisch `Gamehelper` (stabil) mitbearbeiten.
3. **Uebertrag nach Stabil** nur wenn der User explizit sagt (z.B. „Experimental nach Stabil uebertragen“, „in Gamehelper uebernehmen“).
4. Dann `D:\ZusatzProgramme\Gamehelper\scripts\transfer-experimental-to-stable.ps1` nutzen (z.B. `-Paths Plugins/Radar` oder `-All`).
5. **GitHub Stabil** (`rebuild-and-publish.bat`) nur auf Anweisung — getrennt von Experimental-Publish.
6. Deploy-Ordner: jeweils `publish\` im Quellprojekt (Stabil und Experimental).

## Builds

- Experimental: `Gamehelper-Experimental-Src\rebuild-experimental.bat` / `rebuild-and-publish-experimental.bat` -> `publish\`
- Stabil: `Gamehelper\rebuild-and-publish.bat` / `rebuild-test.bat` -> `publish\`

---
> Source: [MordWraith/Gamehelper](https://github.com/MordWraith/Gamehelper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
