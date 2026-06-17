## ssh-port-migrate-safe-skill

> Safely migrate SSH away from port 22 on Linux servers with backup-first, two-phase rollout (migrate then finalize), firewall updates, validation, and rollback commands. Use when users want to change SSH port, block 22, reduce brute-force attacks, or recover from SSH preauth disconnects after port changes.


# SSH Port Migrate Safe

## Overview

Use this skill when a user needs to move SSH from `22` to a custom port without locking themselves out.
This skill enforces a two-phase workflow: `migrate` first (keep `22`), then `finalize` (disable `22`).

## When To Use

- User asks to change SSH port on VPS or cloud server.
- User wants to disable SSH on `22` and keep only a custom port.
- User gets `kex_exchange_identification` / `Connection closed by remote host` after SSH changes.
- User needs a rollback-safe and repeatable SSH hardening process.

## Workflow

1. Confirm there is at least one recovery path.
Use cloud provider console/VNC/serial console before changing SSH.

2. Run phase 1 (`migrate`) on the server.
This keeps `22` + opens new port to avoid lockout.

3. Validate from local machine.
Test `ssh -p <new-port> <user>@<server-ip>` until stable.

4. Run phase 2 (`finalize`) on the server.
This keeps only new port and blocks `22` in UFW.

5. Share rollback commands in delivery.
Always include backup path and recovery steps.

## Commands

Run script directly on server (root):

```bash
bash scripts/ssh-port-migrate-safe.sh --new-port 22022 --phase migrate
bash scripts/ssh-port-migrate-safe.sh --new-port 22022 --phase finalize
```

If `ufw` is inactive and user wants script-managed firewall enablement:

```bash
bash scripts/ssh-port-migrate-safe.sh --new-port 22022 --phase migrate --enable-ufw-if-inactive
```

Allow only fixed CIDR for new port (optional):

```bash
bash scripts/ssh-port-migrate-safe.sh \
  --new-port 22022 \
  --phase finalize \
  --allow-from-cidr 203.0.113.10/32
```

## Guardrails

- Never run `finalize` before verifying login on new port.
- Never remove all SSH access paths in one step.
- Never store plaintext passwords/tokens in scripts, logs, or skill files.
- Prefer key-based authentication for production hardening.

## Resources

1. `scripts/ssh-port-migrate-safe.sh`
2. `references/usage.md`

---
> Source: [Arianassskm/ssh-port-migrate-safe-skill](https://github.com/Arianassskm/ssh-port-migrate-safe-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
