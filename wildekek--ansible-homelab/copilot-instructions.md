## ansible-homelab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an Ansible homelab configuration repository that manages infrastructure across multiple environments including Proxmox VMs, physical machines, and containerized services.

## Key Commands

### Basic Operations
- `ansible-playbook homelab.yaml` - Run the main homelab playbook
- `ansible-playbook update.yaml` - Update all systems and Docker containers
- `ansible-playbook docker-clean.yaml` - Clean up Docker resources
- `ansible-galaxy install -r requirements.yml` - Install required Ansible collections

### Inventory Management
- `ansible-inventory --list` - View all hosts and groups
- `ansible-inventory --graph` - Show inventory structure
- The repository uses dual inventory sources: `inventory.yaml` (static) and `inventory_proxmox.yaml` (dynamic Proxmox plugin)

### Testing and Validation
- `ansible-playbook --check homelab.yaml` - Dry run of main playbook
- `ansible all -m ping` - Test connectivity to all hosts

## Architecture

### Inventory Structure
- **Static inventory** (`inventory.yaml`): Physical machines and manually defined hosts
- **Dynamic inventory** (`inventory_proxmox.yaml`): Automatically discovers Proxmox VMs/LXCs using the community.proxmox plugin
- **Group variables**: No group_vars directory found; variables are defined within playbooks and roles

### Role-Based Organization
The repository uses a role-based approach where hosts are grouped by function:
- `role_docker` - Hosts running Docker containers
- `role_adguard` - AdGuard Home DNS filtering
- `role_pbs_client` - Proxmox Backup Server clients
- `role_labelprinter` - Label printer management

### Key Roles
- **docker_host**: Installs Docker, creates users/groups, sets up directories
- **adguard**: AdGuard Home configuration
- **proxmox_backup_client**: Proxmox Backup Server client setup
- **labelprinter**: Label printer service management
- **shairport_sync**: AirPlay audio receiver (newest addition)

### Authentication & Security
- SSH key authentication configured (`.ssh/id_ansible`)
- Ansible Vault for secrets (`.vault_pass` file)
- Proxmox API token stored in vault format
- Default remote user: `willem` (overrideable per host)

### Playbook Structure
- **homelab.yaml**: Main deployment playbook with role assignments
- **update.yaml**: System and container updates
- **docker-clean.yaml**: Docker maintenance tasks
- **bootstrap-cloudinit.yaml**: Cloud-init configuration for VM deployment

### Docker Integration
- Docker containers managed via `community.docker.docker_compose_v2`
- Docker files expected in `./docker` directory relative to playbook execution
- Container lifecycle managed through update and cleanup playbooks

## Configuration Notes

- Ansible configuration in `ansible.cfg` sets default inventory files and vault password
- Python interpreter set to `auto_silent` for compatibility
- Repository cache updates are performed but marked as `changed_when: false`
- Host filtering excludes stopped VMs and Windows machines by default

## CI/CD Integration (Semaphore)

### Configuration Files

#### `proxmox.cfg` (local, gitignored)
All Proxmox and PBS connection settings live in a local `proxmox.cfg` (ini format, gitignored). Copy the example and fill in your values:
```bash
cp proxmox.cfg.example proxmox.cfg
```

Structure:
```ini
[pve]
host = your-proxmox-host
token_secret = your-pve-api-token

[pbs]
host = your-pbs-host
user = your-pbs-user@pbs
token = your-token-name
datastore = your-datastore
password = your-pbs-api-token
fingerprint = your-pbs-tls-fingerprint
```

#### Local Development
```bash
cp proxmox.cfg.example proxmox.cfg
# fill in proxmox.cfg
ansible-playbook update.yaml
```

#### Semaphore Configuration
Set `PVE_TOKEN_SECRET` as an environment variable in Semaphore. It is used as a fallback when `token_secret` is not set in `proxmox.cfg`.

### Configuration Details

- **`proxmox.cfg.example`**: Template (tracked in git, contains no secrets)
- **`proxmox.cfg`**: Local config with credentials (gitignored, never committed)
- **`inventory/proxmox.yaml`**: reads `host` and `token_secret` from `[pve]`, falls back to `PVE_TOKEN_SECRET` env var
- **`update.yaml`**: same fallback for snapshot tasks
- **`roles/proxmox_backup_client`**: reads all `[pbs]` fields and assembles the repository string via Jinja2

### Troubleshooting

If runs fail with token errors:
1. For local: verify `proxmox.cfg` exists and `token_secret` is set under `[pve]`
2. For Semaphore: check `PVE_TOKEN_SECRET` is set in task configuration
3. Ensure the token is valid and active in Proxmox

---
> Source: [wildekek/ansible-homelab](https://github.com/wildekek/ansible-homelab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
