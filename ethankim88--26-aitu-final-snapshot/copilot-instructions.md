## 26-aitu-final-snapshot

> This file defines the minimum runtime rules for multi-agent execution.

# AGENTS Runtime Rules (Short)

This file defines the minimum runtime rules for multi-agent execution.
Follow `CLAUDE.md` for details; this file only enforces execution procedures.

## Required Procedure
1. `start_session`
2. `get_sitrep`
3. Select the highest-priority `pending` action
4. `claim_action`
5. Execute
6. `complete_action(done|failed, result)`
7. `heartbeat`
8. Repeat

## Work Principles
- Prioritize execution over asking questions; if blocked, log and move to the next action
- Skip actions/hosts already `in_progress`
- `claim_host` before working on a host
- Checklist follows `enum -> exploit -> privesc`
- Delegate long-running or interactive tasks via `request_task`
- If the user gives only generic continuation like `continue`, `keep going`, or `do the next thing`, use `/risk-autopilot` first.
- Maintain only one active objective at a time. State is tracked in `.agents/state/active_objective.json` and updated only via `scripts/util/objective_state.sh`.
- Do not break the objective for unrelated hosts/actions/bugs — send them to the backlog only.
- Objective switch is allowed only when `completed`, `hard blocked`, `direct high-value jump`, or `manual-only pause`.
- While an active objective is alive, Bug Bounty is `same-chain only`.

`request_task` recommended for:
- Full port scan (`nmap -p-`)
- Interactive PTY privilege escalation
- Tasks taking more than 10 minutes

## GPU Cracking (Hashcat)
- **hashcat/john cracking must always run on the GPU instance.**
- MCP/task label: `infra:hashcat-gpu:aws-apne2` (do not confuse with competition target host/credentials)
- IP: `<HASHCAT_IP>`
- SSH alias: `ssh aitufinal-hashcat-gpu`
- SSH direct: `ssh -i labs/infra/stacks/hashcat/hashcat-key.pem ubuntu@<HASHCAT_IP>`
- Upload hashes: `scp HASHES aitufinal-hashcat-gpu:~/crack/`
- Wordlist: `/opt/wordlists/rockyou.txt` (on the server)
- Working directory: `~/crack/`
- Instance type: `g4dn.xlarge` (NVIDIA T4)
- Do not register the `ubuntu`/pem SSH access to the hashcat server as an MCP credential. Only register actual competition accounts obtained from cracking via `add_credential` with `source=hashcat-gpu:<hash_source>`.
- Always clean up with `terraform destroy` after use (`labs/infra/stacks/hashcat/`)

## External Linux Callback Machine
- MCP/task label: `infra:callback-linux:public` (do not confuse with competition target host/credentials)
- IP: `<CALLBACK_IP>`
- SSH direct: `ssh root@<CALLBACK_IP>`
- Purpose: reverse shell callback, OOB verification, `nc`/`socat` listener, temporary HTTP payload hosting
- Since competition servers have external internet access, prefer this machine's public IP `<CALLBACK_IP>` over VPN local IP for simple callbacks.
- Use only this single machine as the operational callback host. Do not maintain a separate fallback callback host.
- Run listeners in `tmux`/`nohup` and clean up ports after use.
- Do not register this machine's `root` SSH access as an MCP credential.

## Terminal Safety Rules
- Forbidden: remote `vim/vi/nano`, `less/more/top/htop`, `sshuttle` foreground
- Alternatives: `cat`, `sed`, `tail -n`, `ssh -D 1080 -N -f`, `ligolo-ng`
- Delegate steps requiring interactive TTY via `request_task` instead of handling directly

## Data Recording Principles
- Scan results: `import_scan_data`
- Host info: `add_host`, `update_host`
- Credentials: `add_credential`, store plaintext/hashes separately, only record actually tested combinations via `test_credential_access`
- Extracted files: `add_loot`
- Key history: `log_event`, `log_session_entry`
- Loot path convention: per-port `loots/<ip>/<port>/`, per-host `loots/<ip>/`

## Pivot / Tunneling Rules
- Priority: `ligolo-ng` > `chisel` > `ssh -D`
- Fix mode per segment (`segment -> mode`), no duplicate routes on the same CIDR
- In socks/chisel mode, use `proxychains4 -q` + TCP scan (`-sT`)

## Report Standards (Directly Affects Score)
- Report-based scoring system
- Risk: score decreases by 15% per duplicate acceptance; floor is 40%
- Bug Bounty: max 1,500 points per host
- Attached screenshots must be full-screen only
- Long payloads/reverse shells/curl commands should be submitted as text

## Prohibited Actions
- Attacking other teams, other accounts, or infrastructure services is forbidden
- Attacking hosts outside scope is forbidden
- DoS or aggressive continuous scanning is forbidden
- Using or creating accounts containing the `f13` string is forbidden

## SCADA Notes
- Prioritize obtaining internal credentials/routes before directly attacking the SCADA web interface.

## Command Compatibility
- `/dispatch-manager [sync|next|blocked|done|re-evaluate]`
- `/risk-autopilot [goal override]`
- `/solve-ad <dc_ip> [domain] [user:pass]`
- `/solve-web <target_ip> [port=80]`
- `/solve-scada <target_ip> [port=502]`
- `/update-host-routes <host_ip_or_cidr>`
- `/pivot <pivot_ip>`
- `uv run scripts/recon/scan_and_import.py -t <cidr> [--deep]`

---
> Source: [EthanKim88/26-AITU-FINAL-SNAPSHOT](https://github.com/EthanKim88/26-AITU-FINAL-SNAPSHOT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
