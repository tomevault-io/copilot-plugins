## proxmox-cicd-deploy-skill

> Design, debug, and harden CI/CD deploy pipelines to Proxmox (single-node and cluster) from GitHub Actions hosted runners. Use when a workflow deploys through SSH, runs pct exec inside LXC containers, fails with network/auth/node-resolution errors, or shows shell-wrapper issues with appleboy/ssh-action (drone-ssh).


# Proxmox CI/CD Deploy Guardian

## Overview

Use this skill to build reliable Proxmox deploy jobs and to debug failing GitHub Actions deploy runs quickly.
Prioritize deterministic staging, explicit logging, and fail-fast behavior across network, auth, cluster-node, and in-container execution layers.

## Placeholder contract

Use these placeholders consistently in generated workflows and scripts:

| Placeholder | Meaning | Example |
| --- | --- | --- |
| `CT_ID` | Target LXC container ID in Proxmox | `124` |
| `APP_DIR` | Absolute app path inside the container | `/opt/myapp` |
| `EXPECTED_NODE` | Proxmox node expected/reachable for precheck | `pve2` |
| `SERVICE_NAME` | Systemd service to restart after deploy | `myapp` |
| `HEALTH_URL` | In-container health endpoint URL | `http://127.0.0.1:8000/health` |

## Workflow

### 1. Lock deployment topology first

Confirm this chain before editing workflow code:
- GitHub hosted runner can reach only public Proxmox endpoint.
- Target LXC is private.
- Deploy must run as: `runner -> public proxmox ssh -> pct exec on CT node -> commands inside container`.

If workflow attempts direct SSH to private LXC IP, treat as architecture bug and replace with Proxmox-host mediation.

### 2. Classify the failure layer from logs

Read failing logs and classify into one layer only before changing code:
- Layer A: Runner to Proxmox reachability/auth (cannot connect, permission denied).
- Layer B: Cluster node mismatch (`CT config/file does not exist` on current node).
- Layer C: Wrapper/shell control-flow issues (drone-ssh + `set -e` behavior).
- Layer D: In-container runtime issues (git safe.directory, venv, service restart, health endpoint).

Do not patch multiple layers at once unless logs prove coupled failures.

### 3. Enforce standard deploy stages

Use fixed stages and logs:
- `DEBUG HOST`
- `PRECHECK SSH`
- `NODE RESOLUTION`
- `SYNC`
- `INSTALL`
- `RESTART`
- `HEALTH`

Include explicit `hostname`, `CURRENT_NODE`, `CT_NODE`, and execution method (`local` or `ssh_fallback`).

### 4. Resolve CT node dynamically in cluster mode

Always discover container node from cluster fs:
- `ls /etc/pve/nodes/*/lxc/<CT_ID>.conf`
- parse node name from path

If `CT_NODE != CURRENT_NODE`, route `pct exec` through internal node SSH only in batch mode.
Fail fast if internal SSH trust is not configured.

### 5. Use drone-ssh-safe shell patterns

When using `appleboy/ssh-action` with `set -euxo pipefail`:
- Prefer explicit state variables over complex chained boolean control flow.
- Avoid fragile inline condition patterns that can surface unintended non-zero statuses.
- Keep retry loops explicit and bounded.

### 6. Normalize in-container git behavior

Before `git fetch/reset` in container-managed repos, set:
- `git config --global --add safe.directory <APP_DIR>`

This prevents `detected dubious ownership` failures.

### 7. Final health verification

Run service restart and health probe inside CT with retries:
- minimum 5 attempts
- `curl -fsS <HEALTH_URL>`
- sleep between retries
- fail hard on final miss

## Baseline deploy pattern

Use this shape for Proxmox cluster deploy jobs:

```bash
set -euxo pipefail

CT_ID="<CT_ID>"
APP_DIR="<APP_DIR>"
EXPECTED_NODE="<EXPECTED_NODE>"
SERVICE_NAME="<SERVICE_NAME>"
HEALTH_URL="<HEALTH_URL>"
PCT_EXEC_RETRIES=2

CURRENT_NODE="$(hostname -s)"

if ! ssh -o BatchMode=yes -o ConnectTimeout=8 -o StrictHostKeyChecking=no "${EXPECTED_NODE}" hostname >/tmp/precheck-node.out 2>/tmp/precheck-node.err; then
  echo "ERROR: SSH trust between Proxmox nodes not configured"
  cat /tmp/precheck-node.err || true
  exit 1
fi

CT_CONF_PATH="$(ls /etc/pve/nodes/*/lxc/${CT_ID}.conf 2>/dev/null | head -n1 || true)"
[ -z "${CT_CONF_PATH}" ] && echo "ERROR: CT ${CT_ID} not found" && exit 1

CT_NODE="$(echo "${CT_CONF_PATH}" | sed -E 's#^/etc/pve/nodes/([^/]+)/lxc/[0-9]+\.conf$#\1#')"
[ -z "${CT_NODE}" ] && echo "ERROR: Cannot parse CT node" && exit 1

pct_exec() {
  CMD="$1"
  ATTEMPT=1

  while [ "${ATTEMPT}" -le "${PCT_EXEC_RETRIES}" ]; do
    RUN_OK=1
    LOCAL_MATCH=0
    [ "${CT_NODE}" = "${CURRENT_NODE}" ] && LOCAL_MATCH=1 || true

    if [ "${LOCAL_MATCH}" -eq 1 ]; then
      echo "pct_exec method=local node=${CURRENT_NODE} attempt=${ATTEMPT}"
      if pct exec "${CT_ID}" -- bash -lc "set -euxo pipefail; ${CMD}"; then
        RUN_OK=0
      fi
    else
      echo "pct_exec method=ssh_fallback node=${CT_NODE} attempt=${ATTEMPT}"
      if ssh -o BatchMode=yes -o ConnectTimeout=8 -o StrictHostKeyChecking=no "${CT_NODE}" "pct exec ${CT_ID} -- bash -lc 'set -euxo pipefail; ${CMD}'"; then
        RUN_OK=0
      fi
    fi

    [ "${RUN_OK}" -eq 0 ] && return 0 || true
    echo "WARN: pct_exec failed on attempt=${ATTEMPT}"
    ATTEMPT=$((ATTEMPT + 1))
    sleep 2
  done

  echo "ERROR: pct_exec failed after ${PCT_EXEC_RETRIES} attempts"
  exit 1
}

pct_exec "git config --global --add safe.directory ${APP_DIR} && cd ${APP_DIR} && git fetch origin main && git reset --hard origin/main"
pct_exec "cd ${APP_DIR} && source .venv/bin/activate && pip install -r requirements.txt"
pct_exec "systemctl restart ${SERVICE_NAME}"

HEALTH_OK=0
for ATTEMPT in 1 2 3 4 5; do
  if pct_exec "curl -fsS ${HEALTH_URL}"; then
    HEALTH_OK=1
    break
  fi
  sleep 3
done

[ "${HEALTH_OK}" -ne 1 ] && echo "ERROR: health check failed" && exit 1
```

## Troubleshooting quick map

- `Cannot reach <host>:<port> from GitHub runner`:
  deploy path is wrong for private network; route through public Proxmox host.
- `Permission denied (publickey,password)`:
  fix `PROXMOX_USER`/`PROXMOX_SSH_KEY` pairing and authorized_keys on host.
- `.../lxc/<id>.conf does not exist`:
  executing on wrong node; detect and route to actual `CT_NODE`.
- `detected dubious ownership`:
  set git safe.directory for app path inside CT.
- health endpoint fails after restart:
  inspect `systemctl status <SERVICE_NAME>` and journal logs inside container.

## References

- For reusable snippets and error signatures, load:
  `references/proxmox-cicd-playbook.md`

---
> Source: [NANDI-Services/proxmox-cicd-deploy-skill](https://github.com/NANDI-Services/proxmox-cicd-deploy-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
