## nutanix-ova-backup-restore-tool

> This document provides context for the Gemini Code Assistant to understand the Nutanix OVA Backup & Restore Tool project.

# Gemini Code Assistant Context

This document provides context for the Gemini Code Assistant to understand the Nutanix OVA Backup & Restore Tool project.

## Project Overview

This project is a command-line tool for backing up and restoring Nutanix VMs. The tool is written in bash and uses `curl` and `jq` to interact with the Nutanix Prism Central API.

The tool is organized into a set of scripts that provide different functionalities:

*   `setup_wizard.sh`: Configures the tool with Nutanix Prism Central credentials.
*   `export_menu.sh`: Exports VMs to OVA files.
*   `restore_menu.sh`: Restores VMs from OVA files.
*   `custom_restore.sh`: Restores a single VM with custom settings.
*   `manage_restore_points.sh`: Manages backup restore points.
*   `ui_lib.sh`: A shared library that provides common UI/UX, API functions, and utilities for the other scripts.

## Building and Running

The tool does not require any building. To run the tool, you need to have `bash`, `curl`, and `jq` installed.

To run the tool, first make the scripts executable:

```bash
chmod +x scripts/*.sh
```

Then, run the setup wizard to configure the tool:

```bash
./scripts/setup_wizard.sh
```

After the setup is complete, you can run the other scripts to perform the desired actions. For example, to export VMs, run:

```bash
./scripts/export_menu.sh
```

## Development Conventions

The scripts are written in bash and follow a consistent style. The `ui_lib.sh` script provides a set of common functions that are used by the other scripts. This helps to ensure that the UI/UX is consistent across the different scripts.

The scripts use `curl` to make API calls to the Nutanix Prism Central API. The `jq` command is used to parse the JSON responses from the API.

The scripts are well-documented with comments that explain the purpose of each script and the functions within each script.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/superlaser97)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/superlaser97)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
