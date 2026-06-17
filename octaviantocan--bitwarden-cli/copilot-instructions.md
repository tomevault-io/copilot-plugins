## bitwarden-cli

> Use the Bitwarden Secrets Manager CLI bws safely. Trigger when working with Bitwarden, Secrets Manager, bws, BWS_ACCESS_TOKEN, secret get/list/create/edit/delete, project list, bws run, or secret-backed environment variables.


# Bitwarden CLI

Use Bitwarden Secrets Manager CLI (`bws`) without leaking secret values. Prefer metadata, shape checks, and controlled command execution over printing credentials.

## Variables

- **CLI**: `bws`
- **BOOTSTRAP_ENV**: `BWS_ACCESS_TOKEN`
- **SAFE_CHECK_SCRIPT**: `scripts/bws-safe-check.mjs`

## How It Works

`bws` authenticates with a machine-account access token from `BWS_ACCESS_TOKEN`. Secret values stay in Bitwarden unless a task explicitly requires injecting them into a subprocess environment.

## Commands

| Command | Purpose |
| --- | --- |
| `/bitwarden-cli check` | Verify `bws`, auth, and optional secret IDs without printing values |
| `/bitwarden-cli read <secret-id>` | Retrieve a secret only for immediate local use; do not print it |
| `/bitwarden-cli run <command>` | Run a trusted command with Bitwarden-injected env vars |

## Cookbook

Read the relevant cookbook file before running the command.

| Command | Cookbook | Use When |
| --- | --- | --- |
| check | [cookbook/check.md](cookbook/check.md) | Validating setup, auth, projects, or secret IDs |
| read | [cookbook/read.md](cookbook/read.md) | A secret value is needed by a script/process |
| run | [cookbook/run.md](cookbook/run.md) | A command needs secret-backed env vars |

## Safety Rules

1. Never print `BWS_ACCESS_TOKEN` or secret values.
2. Prefer `bws secret list --output json` for key/id metadata.
3. Use `bws secret get <id> --output json` only inside a process that consumes the value.
4. Treat `bws run` as sensitive command execution; never run untrusted commands with inherited secrets.
5. Report secret shape, length, key, id, project id, and command status only when useful.

## Related Skills

Works with: /telegram-notify, /plugin-creator, /create-skill

---
> Source: [OctavianTocan/bitwarden-cli](https://github.com/OctavianTocan/bitwarden-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
