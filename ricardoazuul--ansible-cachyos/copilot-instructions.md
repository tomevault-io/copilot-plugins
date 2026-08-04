## ansible-cachyos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Multi-playbook Ansible project that configures a laptop running CachyOS or Ubuntu. It runs locally against `localhost` and installs packages via the appropriate package manager for the detected OS (pacman/paru for Arch-based, apt/snap for Debian-based), plus app-specific methods (AppImage, systemd).

## Commands

Install Ansible collections (required before first run):
```bash
ansible-galaxy collection install -r requirements.yml
```

Run the full playbook:
```bash
ansible-playbook playbooks/laptop.yml --ask-become-pass
```

Lint (mirrors CI):
```bash
ansible-lint
```

Run with verbose output for debugging:
```bash
ansible-playbook playbooks/laptop.yml --ask-become-pass -v
```

## Architecture

```
ansible.cfg           # Sets inventory = inventory/hosts.ini
inventory/hosts.ini   # Groups: cachyos_laptop, ubuntu_laptop; parent group: laptop
vars/packages.yml     # Package lists: cachyos_pacman_packages, cachyos_aur_packages,
                      #                ubuntu_apt_packages, ubuntu_snap_packages,
                      #                ubuntu_snap_classic_packages
playbooks/laptop.yml  # Entry-point: imports cachyos.yml and ubuntu.yml
playbooks/cachyos.yml # CachyOS / Arch Linux tasks (targets cachyos_laptop group)
playbooks/ubuntu.yml  # Ubuntu / Debian tasks (targets ubuntu_laptop group)
requirements.yml      # community.general + kewlfft.aur collections
```

`playbooks/laptop.yml` is the entry-point that imports the two OS-specific playbooks:
- `playbooks/cachyos.yml` — targets the `cachyos_laptop` inventory group; handles pacman, AUR/paru, plus common tasks (Espanso AppImage, rclone systemd service)
- `playbooks/ubuntu.yml` — targets the `ubuntu_laptop` inventory group; handles apt, snap, plus common tasks (Espanso AppImage, rclone systemd service)

Each OS-specific playbook can also be run independently against its respective hosts.

### AUR handling (CachyOS / Arch Linux only)

AUR packages require an unprivileged `aurbuilder` user. The playbook:
1. Creates the `aurbuilder` system user
2. Grants it passwordless sudo for `pacman` and `paru` only
3. Checks whether `paru` is installed; if not, clones and builds it from source
4. Uses `kewlfft.aur` to install packages via `paru` as `aurbuilder`

### Adding packages

- **Official/CachyOS packages** → add to `cachyos_pacman_packages` in `vars/packages.yml`
- **AUR packages** → add to `cachyos_aur_packages` in `vars/packages.yml`
- **Ubuntu apt packages** → add to `ubuntu_apt_packages` in `vars/packages.yml`
- **Ubuntu snap packages** → add to `ubuntu_snap_packages` (or `ubuntu_snap_classic_packages` for classic confinement) in `vars/packages.yml`
- **Other install methods** (AppImage, systemd service, etc.) → add tasks directly to `playbooks/cachyos.yml`, `playbooks/ubuntu.yml`, or both as appropriate

### Inventory setup

- **CachyOS / Arch Linux**: uncomment (or add) `localhost ansible_connection=local` under `[cachyos_laptop]`
- **Ubuntu**: uncomment (or add) your host under `[ubuntu_laptop]`

## CI

GitHub Actions runs `ansible-lint` on every pull request to `main`. No deployment pipeline — the playbook is run manually on the target machine.

---
> Source: [RicardoAzuul/ansible-cachyos](https://github.com/RicardoAzuul/ansible-cachyos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
