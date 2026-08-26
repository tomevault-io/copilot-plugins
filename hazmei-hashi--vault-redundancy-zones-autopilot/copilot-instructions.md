## vault-redundancy-zones-autopilot

> Multipass-based lab for Vault Enterprise Autopilot + Redundancy Zones. Shell scripts only — no build/test tooling.

# AGENTS.md

Multipass-based lab for Vault Enterprise Autopilot + Redundancy Zones. Shell scripts only — no build/test tooling.

---

## Bash 3.2 Compatibility (HOST scripts only — #1 bug source)

`provision.sh` and `teardown.sh` run on macOS bash 3.2. **Never use:**

- `declare -A` — use indexed arrays + parallel-array lookups
- `${var,,}` / `${var^^}` — use `case` or `tr`
- `mapfile` / `readarray`
- `wait -n` — use `wait "$pid" || rc=$?` in a loop
- `${arr[-1]}` (negative indices)
- `;;&` / `;&` case fall-through

`scripts/install-vault.sh` runs **inside Ubuntu VMs** (bash 4+) — exempt from these rules.

---

## macOS/BSD Portability

- `mktemp` templates must have trailing X's with **no suffix after them**: `mktemp /tmp/foo.XXXXXX` — BSD requires X's at the end; `foo-XXXXXX.hcl` fails on macOS. Minimum 6 X's.
- Avoid `sed -i` without a backup-suffix arg, GNU-only `date`, `readlink -f`, `grep -P`.
- Config rendering uses a pure-bash `while read` loop (not sed multi-line) — this is intentional for BSD portability.

---

## How to Run

```sh
# Provision (VAULT_LICENSE required — preflight exits without it)
export VAULT_LICENSE="<enterprise-license>"
./provision.sh

# Teardown
./teardown.sh
./teardown.sh --purge-secrets   # also shreds .vault-init.json non-interactively
```

No tests, no linter, no build step. Verification = running `provision.sh` end-to-end against Multipass.

**Host prerequisites** (preflight-checked): `multipass`, `jq`, `curl`, `vault` CLI on PATH.

---

## Topology / Env Vars

| Var | Default | Notes |
|-----|---------|-------|
| `ZONES` | 3 | |
| `NODES_PER_ZONE` | 2 | max 26 |
| `VM_CPUS` | 1 | |
| `VM_MEM` | 1G | |
| `VM_DISK` | 5G | |
| `PARALLELISM` | 3 | caps concurrent VM launches |
| `VAULT_VERSION` | (latest) | accepts `1.17.3`, `1.17.3+ent`, `1.17.3+ent-1`; bare forms auto-resolved via `apt-cache madison` |

**Teardown MUST receive the same vars as provision** (it reconstructs the VM list from them).

**VM naming:** `z<zone>-<letter>` (z1-a, z1-b, z2-a…). Bootstrap node is always `z1-a`. Zone is derived from the name (`z2-b` → `zone-2`) — the naming scheme is load-bearing, don't change it casually.

**The VM-name generation loop is duplicated verbatim in both scripts** (intentional, not extracted). If you change one, change the other — both carry a sync-note comment.

---

## Execution Flow

`provision.sh` has 4 labeled steps:

1. **[1/4] Phase A** — launch VMs + discover IPs (bounded-parallel, capped by `PARALLELISM`)
2. **[2/4] Phase B** — render config + transfer + install Vault (bounded-parallel)
3. **[3/4]** — init + unseal bootstrap `z1-a` (**sequential**)
4. **[4/4]** — unseal remaining nodes (**sequential**)

**The two-phase split is deliberate:** ALL IPs must be known (Phase A) before ANY config is rendered (Phase B), because every node's config needs the retry_join anchor IPs (the `-a` node of each zone). Do not merge the phases.

Parallel jobs write to per-node temp logs; failures aggregated per batch. Init/unseal is intentionally not parallelized.

---

## Cluster Wiring (Vault-Specific)

- **Joining is `retry_join` only** (in `config/vault.hcl.tmpl`). There is no `vault operator raft join` command anywhere.
- **There is no `-autopilot-redundancy-zone` CLI flag** — it doesn't exist. Redundancy zone is set exclusively via `autopilot_redundancy_zone` in the `storage "raft"` stanza.
- **License is autoloaded** via `license_path` in config (file transferred to each VM before `systemctl start`). Do NOT use `vault write sys/license` — that endpoint is removed in modern Vault; an unlicensed Enterprise node cannot even unseal.
- **Raft ordering:** a node must be joined before it can be unsealed. Unsealing uses the same Shamir keys as bootstrap.
- `config/vault.hcl.tmpl` uses a `RETRY_JOIN_BLOCKS` placeholder replaced by provision.sh with one `retry_join` block per zone anchor. The template **must end with a trailing newline** (render loop uses `while read || [[ -n "$line" ]]` to guard the last line — keep the newline).

---

## Lab-Only Settings (Do Not "Harden" Unless Asked)

- TLS disabled (`tls_disable = 1`, `http://` everywhere) — local testing only.
- Manual Shamir unseal (3 key shares / 2 threshold). No auto-unseal.
- Swap disabled inside each VM (`swapoff -a` + fstab edit in `install-vault.sh`) — HashiCorp best practice to avoid paging secrets to disk. Do not re-add a swapfile.
- 1 vCPU / 1 GiB per VM is tight; peak host RAM ≈ `PARALLELISM × VM_MEM` during launch. With guest swap off, an overcommitted VM gets OOM-killed rather than slowed.

---

## Secrets

- `vault operator init` output is written to `.vault-init.json` (gitignored). Root token is **not** echoed to stdout by design.
- `*.hclic` license files are gitignored. Never commit either or echo the root token.
- `provision.sh` is re-runnable: skips existing VMs, skips init if `.vault-init.json` exists, install is idempotent.

---
> Source: [hazmei-hashi/vault-redundancy-zones-autopilot](https://github.com/hazmei-hashi/vault-redundancy-zones-autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
