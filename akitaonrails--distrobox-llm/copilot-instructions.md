## distrobox-llm

> Guidance for Claude Code when working with this repo.

# CLAUDE.md

Guidance for Claude Code when working with this repo.

## What this is

Ansible playbooks for reproducibly creating and configuring an Arch-based distrobox named `llm`. The box hosts CUDA, cuDNN, Ollama (CUDA build), whisper.cpp (CUDA build), and the LM Studio AppImage. Purpose: isolate the heavy LLM/ML binary stack from the host, where only the NVIDIA driver + thin launchers remain.

## Conventions

- Ansible variable prefix: `dl_` (distrobox-llm).
- Sister repo: `~/Projects/distrobox-gaming` uses `dg_` and the same role/tag structure — match its style.
- Sister repo: `~/.config` is the user's host dotfiles. The package-inventory doc at `.docs/package-inventory.md` in that repo tracks what's on the host vs what's distrobox'd.

## Commands

Routine maintenance (no ansible needed):
```sh
bin/llm-update                         # yay -Syu inside the box
bin/llm-enter                          # enter the box shell
```

Ansible (from `ansible/`) — for declared-state reconciliation, not routine updates:
```sh
ansible-playbook site.yml              # full setup from scratch
ansible-playbook site.yml --tags bootstrap  # just create box + install yay
ansible-playbook site.yml --tags packages   # reconcile pacman + AUR package lists
ansible-playbook site.yml --tags apps       # (re-)download AppImages
ansible-playbook site.yml --tags verify     # assertions only
ansible-playbook backup.yml
ansible-playbook restore.yml -e restore_archive=<path>
```

Rule of thumb: if you edited `group_vars/all/*.yml`, run ansible. Otherwise `llm-update` is faster and simpler.

## Key invariants

- Box home MUST stay at `/mnt/data/distrobox/{{ dl_box_name }}` (keeps host /home free of big stuff + away from snapshots).
- NVIDIA passthrough via `distrobox create --nvidia` — must be true for this use case.
- UID matches host (1026) automatically because distrobox runs as the invoking user.
- AppImages live under `$HOME/Apps/` inside the box. Do NOT copy AppImages from the host — always re-download to keep the box reproducible from just this repo.
- Prefer binary AUR packages (`-cuda`, `-bin`, `-appimage`). Avoid `-git` unless there's no alternative.

## Adding a new LLM / ML tool

1. If available in pacman repo: add to `group_vars/all/packages.yml` → `dl_pacman_packages`.
2. If AUR with binary variant: add to `dl_aur_packages`.
3. If AppImage: add entry in `group_vars/all/apps.yml` + add a task in `roles/install_appimages/tasks/main.yml`.
4. `ansible-playbook site.yml --tags packages,apps` to apply.
5. Bump the verify role if it's a long-running service worth a smoke test.

## Don't

- Don't install host-side packages from this repo. Host changes go to `~/.config`.
- Don't add emulator/game tooling — that belongs in `distrobox-gaming`.
- Don't store model weights in git or in backups (the backup.yml already excludes `ollama/models` and `.lmstudio/models`).

---
> Source: [akitaonrails/distrobox-llm](https://github.com/akitaonrails/distrobox-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
