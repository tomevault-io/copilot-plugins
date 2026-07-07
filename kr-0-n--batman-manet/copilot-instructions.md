## batman-manet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project does

This repo automates the setup of a **MANET (Mobile Ad-hoc NETwork)** using the [B.A.T.M.A.N. (batman-adv)](https://www.open-mesh.org/projects/batman-adv/wiki/Using-batctl) mesh routing protocol on a Raspberry Pi cluster. Ansible provisions the nodes; shell scripts configure the mesh and bridge interfaces at runtime.

## Running the playbook

```bash
# Provision all nodes and configure batman
ansible-playbook -i pis.ini config-batman.yml

# Target a single host
ansible-playbook -i pis.ini config-batman.yml --limit 192.168.3.19
```

`ansible.cfg` sets `pis.ini` as the default inventory, so `-i pis.ini` can be omitted if running from the repo root.

## Architecture

### Provisioning flow (`config-batman.yml`)

1. **All nodes** — installs `batctl`, copies and runs `config_batman.sh <mesh_ip>` where the IP is derived as `192.168.3.<base_ip_start + node_index>` (base `241`).
2. **Manager node only** — copies and runs `bridge.sh`, bridging `bat0` (mesh) and `eth0` (ethernet) into `br0` so the operator's laptop can reach all mesh nodes via a single ethernet connection.

### Inventory files

- `pis.ini` — **setup-time** inventory: nodes reachable over the ethernet LAN (pre-batman IPs). Used when first running the playbook.
- `pis-batman.ini` — **post-setup** inventory: nodes addressed by their batman mesh IPs (`192.168.3.241–245`), workers accessed via `ProxyJump` through the manager. Used after the mesh is running.

### Shell scripts

| Script | Runs on | Purpose |
|---|---|---|
| `config_batman.sh <mesh_ip>` | Each Pi | Stops NetworkManager/wpa_supplicant, puts `wlan0` in ad-hoc mode on SSID `meshnet` channel 1, loads `batman-adv`, assigns mesh IP to `bat0`, adds default route via `192.168.3.241` |
| `bridge.sh` | Manager Pi | Bridges `eth0` + `bat0` into `br0`, preserving both IP addresses |
| `network_prober.sh` | Each Pi | Measures per-neighbour latency and throughput via `batctl`, then pushes results to a gRPC `LinkService` (port-forwarded from a K8s pod) |
| `network_probe_runner.sh` | Each Pi | Wrapper that runs `network_prober.sh` in a loop with random 10–120 s backoff |

### Systemd services

- `batman.service` — reads IP from `/home/pi/ip_addr` and runs `config_batman.sh` on boot. **Must be installed and the `ip_addr` file created manually** (not done by the playbook).
- `network_prober.service` — runs `network_probe_runner.sh` continuously via `Restart=always`.

### Network topology

```
Laptop ── eth0 ── [Manager Pi (br0: eth0 + bat0)] ── batman-adv mesh ── Worker Pis
                   192.168.3.241/28                    192.168.3.241–245/28
```

The `/28` subnet covers `.241–.254` (14 addresses); default route for all mesh nodes points to the manager (`192.168.3.241`).

### Kubernetes / registry

`registry/registry.yml` deploys a Docker registry (`registry:2`) as a K8s Deployment + NodePort service on port `30500`, backed by a 5 Gi PVC using `local-path` storage class.

## Known manual steps (not automated)

- Copy `batman.service` to `/etc/systemd/system/` on each Pi and run `sudo systemctl enable batman`.
- Create `/home/pi/ip_addr` on each Pi containing its desired mesh IP.
- The `network_prober.sh` expects a `message.proto` at `/home/pi/` and a running `apiserver` pod in K8s with port `50051`.

---
> Source: [kr-0-n/batman-manet](https://github.com/kr-0-n/batman-manet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
