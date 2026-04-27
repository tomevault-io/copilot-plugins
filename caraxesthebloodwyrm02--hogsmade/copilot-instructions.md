## hogsmade

> Dedicated OS guardrails workflow for Arch Linux workstation security hardening


# OS Guardrails

Use this rule when the task involves Arch Linux workstation hardening, `sysctl` security parameters, `nftables` firewall or audit rules, `auditd`, service exposure review, suspicious network activity, or reboot-persistence verification.

Canonical sources:
- `/home/caraxes/skills/os-guardrails/SKILL.md`
- `/home/caraxes/skills/os-guardrails/references/guardrail-baseline.md`

If this rule drifts from the canonical skill, follow the files above.

## When This Rule Applies

Apply this workflow when asked to:
- review or harden OS-level security posture
- inspect or tune `sysctl` kernel parameters
- review, create, or fix `nftables` rules
- investigate `AUDIT-*` kernel journal entries
- verify open ports, listeners, or service exposure
- check whether guardrails survive reboot
- assess `auditd` enable/disable posture
- perform post-update security verification after `pacman -Syu`

## Protocol

### 1. Inspect First

Start with read-only inspection before proposing changes.

Review:
- live `sysctl` values
- persisted `sysctl` configuration
- `nftables` service state
- loaded `inet filter` and `inet audit_log` tables
- listening TCP and UDP sockets
- `sshd` state
- `auditd` state and rules presence
- persistence of firewall audit includes
- externally-bound listeners vs loopback-only listeners

Prefer runtime truth over config intent. A live value is authoritative.

### 2. Classify Risk

Separate findings into three tiers:

#### Tier 1 — Safe
Safe persistence, backups, documentation, and verification-only actions:
- persisting already-active settings
- backing up configs
- validating current rules
- adding reboot persistence for already-working configs

#### Tier 2 — Low Risk
Reversible runtime changes that require explicit approval:
- raising `kernel.yama.ptrace_scope`
- enabling or disabling `auditd`
- tightening `nftables` audit logging
- reducing service exposure
- correcting loopback or listener binding

#### Tier 3 — Excluded Unless Explicitly Named
Do not propose or apply these unless the user explicitly names the exact item:
- `linux-hardened`
- kernel lockdown mode
- `module.sig_enforce=1`
- `hidepid=2` on `/proc`
- `hardened_malloc` preload
- AppArmor rollout
- SELinux rollout
- LKRG

Reason: this workstation uses NVIDIA and KVM, and these changes can break drivers, modules, virtualization, desktop behavior, or developer tooling.

### 3. Harden In Order

If approved, apply changes in this sequence:
1. `sysctl`
2. `nftables`
3. service posture
4. reserve audit layer (`auditd`)

Do not jump directly to invasive changes if a lower-risk layer solves the problem.

### 4. Verify Live State

After changes, verify:
- runtime values, not just files
- firewall table presence
- audit table presence if enabled
- input policy on base firewall
- service states
- listener exposure
- reboot persistence paths

Use `/home/caraxes/skills/os-guardrails/references/guardrail-baseline.md` as the target baseline.

### 5. Report by Layer

Summarize status by layer:
- Kernel
- Firewall
- Logging
- Filesystem
- Services
- Reserve / auditd

Call out:
- current state
- persistence
- gaps
- next actions
- rollback path where relevant

## nftables Rule Discipline

When editing `/home/caraxes/config/firewall-audit.nft` or related audit rules, follow these rules strictly.

### Always Do

- Exclude loopback first:
  - `oifname "lo" accept`
  - `iifname "lo" accept`
- Whitelist known-good traffic before any catch-all logging
- Rate-limit every log rule
- Use positive priority so audit rules run after the base firewall
- Filter catch-all logging on `ct state new`
- Keep configs idempotent when possible
- Treat silence under normal use as the success condition

### Known-Good Traffic to Exclude First

- loopback traffic
- `tcp dport { 80, 443 }`
- DNS to known resolvers
- DHCP
- NTP
- LAN broadcast and multicast
- `ct state established,related`

### Never Do

- never log all outbound traffic
- never log loopback traffic
- never flag normal HTTPS or HTTP as suspicious
- never flag known DNS resolvers as C2 or IRC
- never flag normal LAN discovery as port scans
- never use un-rate-limited log rules
- never run audit chains at negative priority

If the journal becomes noisy under normal use, the rules are wrong and must be tightened.

## Guardrail Baseline Highlights

Use these expectations as a quick mental checklist:

- `kernel.randomize_va_space = 2`
- `kernel.kptr_restrict = 2`
- `kernel.dmesg_restrict = 1`
- `kernel.yama.ptrace_scope = 2`
- `kernel.unprivileged_bpf_disabled = 1`
- `fs.protected_hardlinks = 1`
- `fs.protected_symlinks = 1`
- `fs.protected_fifos = 2`
- `fs.protected_regular = 2`
- `fs.suid_dumpable = 0`
- `nftables` enabled
- base firewall input policy `drop`
- audit firewall table present when enabled
- `sshd` disabled unless explicitly needed
- developer services bound to loopback only
- no externally-bound TCP listeners for local tooling

## Key Files

- `/home/caraxes/skills/os-guardrails/SKILL.md`
- `/home/caraxes/skills/os-guardrails/references/guardrail-baseline.md`
- `/home/caraxes/config/firewall-audit.nft`
- `/home/caraxes/config/finalize-guardrails.sh`
- `/home/caraxes/config/audit.rules`
- `/home/caraxes/config/service-hardening.conf`
- `/home/caraxes/config/sysctl-hardening-backup.conf`
- `/home/caraxes/docs/security-hardening-guide.md`

## Constraints

- Never weaken existing security controls without explicit request.
- Never touch excluded Tier 3 items unless the user names the exact item.
- Always prefer reversible, idempotent changes.
- Always back up system-facing config before modifying it.
- Tie every action to the stated security objective.
- Prefer a whitelist-then-alert firewall design over broad logging.
- A noisy audit log is a broken configuration, not a successful one.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caraxesthebloodwyrm02) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
