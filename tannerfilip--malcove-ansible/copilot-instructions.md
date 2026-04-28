## malcove-ansible

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

`malcove-ansible` provisions **mal-box**, a Linux malware analysis workstation, primarily targeting a **Kasm Workspaces Docker image** (Debian/Ubuntu-based). It installs and configures tools for network analysis, reverse engineering, Python/JS malware analysis, document analysis, and YARA rulesets — all via Ansible roles.

## Common Commands

```bash
# Install Galaxy dependencies (run once or after requirements.yml changes)
ansible-galaxy collection install -r requirements.yml

# Full provision
ansible-playbook playbooks/provision_malbox.yml

# Dry run (check mode)
ansible-playbook playbooks/provision_malbox.yml --check

# Run specific roles
ansible-playbook playbooks/provision_malbox.yml --tags malbox_network

# Override YARA tier (core | extended | full)
ansible-playbook playbooks/provision_malbox.yml -e malbox_yara_forge_tier=core

# Lint
ansible-lint
```

## Architecture

The canonical entry point is `playbooks/provision_malbox.yml`, which runs 7 roles in order against the `[malbox]` group (localhost by default). The legacy `playbook.yml` in the root is a monolithic version kept for reference/remote use but is not the preferred approach.

**Role execution order:**
1. `malbox_base` — apt cache, core utilities, shell aliases, PATH config
2. `malbox_network` — tcpdump, tshark, nmap, whois, dnsutils
3. `malbox_reverse_engineering` — apt tools + GitHub release downloads for capa and DIE
4. `malbox_python_tools` — pip3 packages for PE/binary/crypto/document analysis
5. `malbox_node_tools` — Node.js (via `geerlingguy.nodejs`), webcrack, box-js, js-beautify
6. `malbox_document_analysis` — Didier Stevens tools (pdfid, pdf-parser, oledump)
7. `malbox_yara` — YARA Forge rulesets from GitHub releases

**Inventory:** `inventories/localhost/` — single host (`localhost`) with `ansible_connection: local` and `ansible_python_interpreter: /usr/bin/python3`.

**Tool installation pattern:** Roles that pull GitHub releases query the GitHub API for the latest version at runtime (capa, DIE, YARA Forge). Binaries install to `/opt/<tool>` and are symlinked into `/usr/local/bin`.

## Key Defaults

| Variable | Default | Role |
|---|---|---|
| `malbox_base_install_dir` | `/opt` | malbox_base |
| `malbox_base_bashrc_path` | `/root/.bashrc` | malbox_base |
| `malbox_node_tools_node_version` | `22.x` | malbox_node_tools |
| `malbox_yara_forge_tier` | `extended` | malbox_yara |
| `malbox_yara_install_dir` | `/opt` | malbox_yara |

## Lint Rules

`.ansible-lint` enforces `production` profile with these specifics:
- Loop variable names must match `^(__|malbox_)`
- Variable names must match `^[a-z_][a-z0-9_]*$`
- Warnings (not errors) for: `git-latest`, `package-latest`, `risky-file-permissions`, `template-instead-of-copy`

New roles/tasks should follow the `malbox_` variable naming prefix convention used throughout.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TannerFilip) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
