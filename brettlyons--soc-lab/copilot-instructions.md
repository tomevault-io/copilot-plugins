## soc-lab

> Always use the `planning-with-files` skill for session continuity. On every new session:

# SOC Lab — Claude Instructions

## Planning
Always use the `planning-with-files` skill for session continuity. On every new session:
1. Invoke the `planning-with-files` skill to recover context from `task_plan.md`, `findings.md`, and `progress.md`
2. Update `progress.md` at the end of each session
3. Commit and push any changes before the user shuts down

## Key Files
- `task_plan.md` — phases, architecture, decisions, notes, errors
- `progress.md` — session-by-session log
- `findings.md` — network topology, IP table, credentials pointers
- `scripts/host-setup/host-network-setup.sh` — run to restore lab networking after host reinstall

## Lab Quick Reference
- fw-router SSH: `ssh -i ~/.ssh/fw-router-key root@192.168.122.10`
- Wazuh dashboard: `https://192.168.10.10` (credentials in `~/wazuh-install-files.tar` on the VM)
- Restore networking: `bash scripts/host-setup/host-network-setup.sh`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brettlyons) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
