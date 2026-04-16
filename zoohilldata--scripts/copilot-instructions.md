## scripts

> description: Overview documentation for script management systems that handle installation tracking and alias management within shell environments


START SPECIFICATION:
---
description: Overview documentation for script management systems that handle installation tracking and alias management within shell environments
globs: script-manager/get-scripts.bash,add-to-rc.bash
alwaysApply: false
---


# main-overview

## Development Guidelines

- Only modify code directly relevant to the specific request. Avoid changing unrelated functionality.
- Never replace code with placeholders like `# ... rest of the processing ...`. Always include complete code.
- Break problems into smaller steps. Think through each step separately before implementing.
- Always provide a complete PLAN with REASONING based on evidence from code and logs before making changes.
- Explain your OBSERVATIONS clearly, then provide REASONING to identify the exact issue. Add console logs when needed to gather more information.


The system implements a script management solution with two core components:

## Installation Status Management (Importance: 85)
`script-manager/get-scripts.bash`
- Script discovery and status tracking system
- Filters specific utility scripts from processing workflow
- Determines installation state through .zshrc alias presence
- Hierarchical organization of scripts by directory
- Configurable visibility rules for uninstalled scripts and path display

## Alias Management System (Importance: 90)
`add-to-rc.bash`
- Managed section approach for shell RC files
- Controlled boundary system using explicit markers
- Script-to-alias mapping preserving repository structure
- Conflict prevention through duplicate detection
- Single source of truth enforcement for alias definitions

The business logic centers around maintaining a controlled environment for script installations and aliases, with clear ownership boundaries and automated management workflows. The components work together to provide:

1. Script Discovery
- Repository scanning with controlled depth
- Selective processing rules
- Directory-based organization

2. Installation Tracking
- Status determination through alias presence
- Configurable visibility policies
- Hierarchical status reporting

3. Alias Management
- Automated alias generation
- Conflict prevention
- Managed section isolation
- Change management policies

$END$
END SPECIFICATION

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZooHillData) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
