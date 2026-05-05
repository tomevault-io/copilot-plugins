## proxmox-automation

> description: Proxmox scripts, Cloud-Init snippets, and VMID scheme

---
description: Proxmox scripts, Cloud-Init snippets, and VMID scheme
globs: proxmox/**/*
alwaysApply: false
---

# Proxmox — Scripts, Snippets & VMID Scheme

Code under `proxmox/` automates template creation and Cloud-Init baseline. Stay consistent with the existing VMID scheme and Cloud-Init patterns.

## VMID Ranges

- **9000–9099** — Templates (Cloud-Init base images). Do not run directly.
- **100–199** — Core infrastructure (e.g. 110 = core).
- **200–299** — Workload VMs (apps, media, accelerated). Use increments of 10 (210, 220, 230…) to allow insertion without renumbering.
- **800–899** — Temporary / experiments / throwaway VMs.

## Scripts (`proxmox/scripts/`)

- Prefer **portable shell:** `#!/bin/sh` unless bash features are required.
- **Template creation:** Use `qm create`, `qm set`, `qm importdisk`, etc. Accept VM_ID, VM_NAME, STORAGE as arguments or env with sensible defaults.
- **Idempotency / safety:** Check if VM ID already exists before creating; exit with a clear message. Validate required tools (e.g. `qm`, `wget`).
- **Cloud-Init:** Attach vendor snippet via `qm set $VM_ID --cicustom "vendor=local:snippets/cloud-init-config.yaml"`. Document that user must add SSH key and "Convert to Template" in the GUI when relevant.

## Snippets (`proxmox/snippets/`)

- **Format:** YAML `#cloud-config` for Cloud-Init.
- **Contents:** User creation (sudo, docker group, lock_passwd), package_update/upgrade, Docker repo, essential packages (docker-ce, qemu-guest-agent, avahi-daemon, etc.), `write_files` for daemon config (e.g. Docker log rotation), `runcmd` for enable/start services and one-off setup (e.g. swap file).
- **Naming:** Use descriptive filenames (e.g. `cloud-init-config.yaml`). Reference from scripts as `snippets/<name>.yaml`.
- **Reproducibility:** Snippets should produce a generic Docker host baseline; no role-specific or VM-specific secrets in shared snippets.

## Conventions

- Do not hardcode secrets in scripts or snippets; document where to set SSH keys and any secrets (e.g. Proxmox GUI).
- Keep template creation and Cloud-Init in sync with the docs (Chapter 1, Chapter 2) so the written journey matches what the scripts do.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
