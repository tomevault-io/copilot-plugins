## openclaw-secret-manager

> Unified secret manager supporting 1Password, Bitwarden, and local encrypted storage with secret rotation and audit logging.


# OpenClaw Secret Manager

A unified, secure secret management solution for OpenClaw that supports multiple backends:
- **1Password** (via `op` CLI)
- **Bitwarden** (via `bw` CLI)
- **Local Encrypted Storage** (via `age` encryption)

## Features

- 🔐 **Multi-Backend Support**: Seamlessly use 1Password, Bitwarden, or local encrypted files
- 🔄 **Secret Rotation**: Automated secret rotation with configurable policies
- 📊 **Audit Logging**: Comprehensive audit trail for all credential access
- 🛡️ **Security First**: No secrets in logs, encrypted at rest, secure by default

## Installation

```bash
# Install dependencies
brew install 1password-cli bitwarden-cli age

# Copy skill to your OpenClaw skills directory
cp -r openclaw-secret-manager ~/.openclaw/skills/
```

## Quick Start

```bash
# Initialize the secret manager
openclaw-secret-manager init --backend 1password

# Store a secret
openclaw-secret-manager set --key prod/db/password --value "super-secret"

# Retrieve a secret
openclaw-secret-manager get --key prod/db/password

# Rotate a secret
openclaw-secret-manager rotate --key prod/db/password --policy 90d
```

## Configuration

Create `~/.config/openclaw-secret-manager/config.yaml`:

```yaml
# Default backend: 1password, bitwarden, or local
default_backend: 1password

# Audit logging
audit:
  enabled: true
  log_file: ~/.config/openclaw-secret-manager/audit.log
  max_size: 100MB
  max_age: 30d

# Rotation policies
rotation:
  enabled: true
  default_policy: 90d
  notify_before: 7d

# Backend-specific settings
backends:
  1password:
    account: my.1password.com
    vault: Production
    
  bitwarden:
    server: https://vault.bitwarden.com
    
  local:
    storage_path: ~/.config/openclaw-secret-manager/secrets
    key_file: ~/.config/openclaw-secret-manager/keys/master.key
```

## Backend Usage

### 1Password Backend

Uses the `op` CLI. Requires desktop app integration.

```bash
# Authenticate (must run in tmux per 1password skill)
tmux new-session -s 1p-auth -d
op signin

# Read secret
openclaw-secret-manager get --backend 1password --key op://vault/item/field
```

### Bitwarden Backend

Uses the `bw` CLI.

```bash
# Login
bw login username@example.com
export BW_SESSION=$(bw unlock --raw)

# Read secret
openclaw-secret-manager get --backend bitwarden --key "item-id/field"
```

### Local Encrypted Backend

Uses `age` for encryption. No external dependencies.

```bash
# Generate master key
age-keygen -o ~/.config/openclaw-secret-manager/keys/master.key

# Store secret
openclaw-secret-manager set --backend local --key api-keys/stripe --value "sk_live_..."

# Retrieve
openclaw-secret-manager get --backend local --key api-keys/stripe
```

## Audit Logging

All access is logged to `audit.log`:

```
[2024-01-15T10:30:00Z] ACCESS  user=deciple1 backend=1password key=prod/db/password action=get status=success
[2024-01-15T10:31:00Z] ROTATE  user=deciple1 backend=1password key=prod/db/password action=rotate status=success old_version=3 new_version=4
[2024-01-15T10:32:00Z] ACCESS  user=deciple1 backend=local key=api-keys/stripe action=get status=denied (unauthorized)
```

## Secret Rotation

Configure rotation policies per secret:

```bash
# Set rotation policy
openclaw-secret-manager policy set --key prod/db/password --rotate-every 90d --notify-before 7d

# Check rotation status
openclaw-secret-manager policy status --key prod/db/password

# Manual rotation trigger
openclaw-secret-manager rotate --key prod/db/password --generate

# Auto-generate new password
openclaw-secret-manager rotate --key prod/db/password --generate --length 32
```

### Rotation Strategies

| Strategy | Description |
|----------|-------------|
| `generate` | Auto-generate cryptographically secure password |
| `prompt` | Prompt user for new value |
| `script` | Execute custom rotation script |
| `external` | Call external rotation API (e.g., AWS Secrets Manager) |

## Security Best Practices

1. **Always use tmux** for 1Password auth (per 1password skill requirements)
2. **Never commit** the config file with secrets
3. **Rotate keys regularly** using the rotation feature
4. **Review audit logs** periodically
5. **Use separate vaults** for different environments
6. **Enable 2FA** on all password manager accounts

## Troubleshooting

### "account is not signed in"

Run authentication in tmux:
```bash
tmux new-session -s op-auth -d
op signin
```

### Permission denied on local backend

Ensure proper file permissions:
```bash
chmod 600 ~/.config/openclaw-secret-manager/keys/master.key
chmod 700 ~/.config/openclaw-secret-manager/secrets
```

### Audit log not writing

Check directory permissions:
```bash
mkdir -p ~/.config/openclaw-secret-manager
chmod 755 ~/.config/openclaw-secret-manager
```

---
> Source: [EcosystemNetwork/openclaw-secret-manager](https://github.com/EcosystemNetwork/openclaw-secret-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
