## kanbanflow-powershell

> PowerShell-Modul als Interface fuer die KanbanFlow API (https://kanbanflow.com). Implementiert ca. 95% der API in leicht konsumierbarer Form.

# CLAUDE.md

## Projektbeschreibung

PowerShell-Modul als Interface fuer die KanbanFlow API (https://kanbanflow.com). Implementiert ca. 95% der API in leicht konsumierbarer Form.

## TechStack

- PowerShell

## Architektur-Vorlage

powershell-module

## Wichtige Befehle

```powershell
# Modul importieren
Import-Module ./Kanbanflow/Kanbanflow.psd1

# Tests ausfuehren
Invoke-Pester
```

## Konventionen

- CmdletBinding fuer alle oeffentlichen Funktionen
- Verb-Noun Naming (z.B. Get-KBFTask, New-KBFTask, Update-KBFTask)
- Alle Funktionen verwenden das Praefix "KBF"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stho32) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
