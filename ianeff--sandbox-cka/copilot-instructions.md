## sandbox-cka

> This repository is a comprehensive study environment for the Certified Kubernetes Administrator (CKA) exam. It combines a local Kubernetes sandbox (managed by Vagrant and VirtualBox) with a custom drill engine and extensive study notes.

# CKA Exam Sandbox & Study Guide

## Project Overview

This repository is a comprehensive study environment for the Certified Kubernetes Administrator (CKA) exam. It combines a local Kubernetes sandbox (managed by Vagrant and VirtualBox) with a custom drill engine and extensive study notes.

**Key Technologies:**

* **Infrastructure:** Vagrant, VirtualBox, Ubuntu 24.04
* **Orchestration:** Kubernetes v1.34 (Bootstrapped with Kubeadm)
* **Networking:** Cilium v1.18+ (eBPF datapath, kubeProxyReplacement), MetalLB (Layer 2)-   **Storage:** Rancher Local Path Provisioner with 20GB extra disk mounted at `/data`*   **Automation:** Python 3.14+ (Drill engine managed by `uv`), Bash (Provisioning)
* **Optimization:** Local caching layer (APT + OCI Registries) running in Docker on the host.

## Networking Architecture

The cluster uses a modern, "exam-hardened" networking stack:
* **CNI**: Cilium v1.18+ with `kubeProxyReplacement=true` (No kube-proxy).
* **LoadBalancer**: MetalLB in Layer 2 ARP mode (IP Range: `192.168.56.200` - `192.168.56.240`).
* **Gateway API**: Enabled via Cilium (GatewayClass: `cilium`), supporting `HTTPRoute` and `TLSRoute`.
* **Storage**:
  * **CSI**: Rancher Local Path Provisioner (StorageClass: `local-path`).
  * **Extra Disk**: 20GB mounted at `/data` on all nodes.
    * `/data/containerd` → bind-mounted to `/var/lib/containerd` (container runtime storage).
    * `/data/local-path-provisioner` → PersistentVolume storage (configured via ConfigMap).
  * PVs created by `local-path` StorageClass use the extra disk, not the root filesystem.

## Setup & Usage

### 1. Local Cache (Optional but Recommended)

To speed up provisioning and save bandwidth, this project uses a local cache container running on your host machine. It provides an APT proxy (via `apt-cacher-ng`) and OCI pull-through mirrors (via `registry`) for Docker Hub, K8s, GHCR, and Quay.

* **Manager Script:** `scripts/cache.sh`
* **Commands:**

    ```bash
    # Start the cache container (background)
    bash scripts/cache.sh up

    # Check status
    bash scripts/cache.sh status

    # View logs
    bash scripts/cache.sh logs

    # Stop the cache
    bash scripts/cache.sh down
    ```

* **Note:** The Vagrant VMs automatically detect the host IP (assuming `192.168.56.1` for the host-only adapter) and configure themselves to use this cache during provisioning.

### 2. Infrastructure Setup

The cluster consists of one control plane node (`ubukubu-control`) and multiple worker nodes (`ubukubu-node-N`).

* **Start Cluster:** `vagrant up`
  * *Note: This will provision the nodes. If the cache is up, it will use it.*
* **SSH into Control Plane:** `vagrant ssh ubukubu-control`
* **SSH into Worker:** `vagrant ssh ubukubu-node-1`

### 3. Configuration Management

A Python script is provided to sync the cluster`s`kubeconfig` to your host machine, allowing you to use `kubectl` directly from your host terminal.

* **Script:** `scripts/manage_k8s_config.py`
* **Dependencies:** Python 3.6+
* **Usage:**

    ```bash
    # Add cluster config to host (merges into ~/.kube/config)
    python3 scripts/manage_k8s_config.py add

    # Remove cluster config
    python3 scripts/manage_k8s_config.py remove
    ```

### 4. Drill Engine

The project includes a custom CLI for practicing exam scenarios, managed by `uv`.

* **Script:** `drill.py`
* **Drill Structure:** Located in `drills/<category>/<drill-name>/` containing `problem.md`, `setup.sh`, and `verify.sh`.
* **Commands:**

    ```bash
    # List available drills
    uv run drill.py list

    # Start a drill (Runs setup script on control plane & shows problem)
    uv run drill.py start <category>/<drill-name>

    # Verify solution (Runs verification script on control plane)
    uv run drill.py verify <category>/<drill-name>
    ```

## Directory Structure

* **`cache/`**: Docker Compose setup for the local caching layer (APT proxy + Registry mirrors).
* **`drills/`**: Contains practice scenarios organized by category (e.g., `networking`, `storage`, `troubleshooting`).
* **`scripts/`**:
  * `cache.sh`: Manages the local cache container.
  * `common.sh`: Common provisioning steps (installing runtime, kubeadm, etc.).
  * `control-plane.sh`: Initializes the control plane.
  * `node.sh`: Joins worker nodes to the cluster.
  * `manage_k8s_config.py`: Host-side kubeconfig management utility.
  * `install_cilium.sh`: Installs Cilium CNI + Gateway API + MetalLB.
* **`declarative_imperatives/`**: A collection of YAML manifests for various K8s resources, likely used for reference or ad-hoc testing.
* **`generated_supplimentaries/`**: Additional AI-generated guides and tips.
* **`notes.md`**: Main study notes following the CKA curriculum.
* **`Vagrantfile`**: Defines the VM configuration (resources, networking, provisioning, cache injection).

## Development Conventions

* **Drills:** New drills should be added as directories in `drills/<category>/`. They must include a `problem.md` (description), `setup.sh` (environment prep), and `verify.sh` (grading logic).
  * **Curriculum:** The curriculum lives at `CKA_exam_cirriculum_v1.34.md`.
  * **Technical Requirements:**
    * `setup.sh`: Should be **idempotent** (e.g., `kubectl create ... --dry-run=client -o yaml | kubectl apply -f -`).
    * `verify.sh`: Must exit non-zero on failure and be non-interactive; use short, deterministic checks.
    * **Crucial:** Use `timeout` for all network checks (e.g., `timeout 2 nc -zv 10.96.0.1 80`).
  * **CKA Scenario Design:** When preparing realistic drills, generate tasks with:
    * Minimal wording and no explanation or hints.
    * No mention of specific tools or commands.
    * Ambiguous but solvable requirements.
    * One or more intentional misconfigurations.
    * A clear end-state verifiable with `kubectl`.
    * **Do NOT include the solution** in the `problem.md`.
* **Provisioning:** Infrastructure changes should happen in `Vagrantfile` or the shell scripts in `scripts/`.
  * **Caching:** `common.sh` handles the logic for injecting APT proxy and containerd mirror configs if the cache is enabled.
* **Python:** Helper scripts are written in standard Python 3 and should use standard libraries where possible to minimize host dependencies. `drill.py` is modern (3.14+).

### Drill Generation QA Protocol (MANDATORY)

When generating new drills, you MUST perform the following QA pass for EACH drill before declaring it complete:

1. **Idempotency Check:** Verify that `setup.sh` can be run multiple times without error or unintended side effects.
2. **Consistency Check:**
    * Ensure `setup.sh` accurately prepares the environment described in `problem.md`.
    * Ensure `verify.sh` strictly validates the end state requested in `problem.md`.
3. **Live Verification:**
    * Run `setup.sh`.
    * **SOLVE THE DRILL** by creating the resources/solution described in `problem.md`.
    * Run `verify.sh` and ensure it returns exit code 0.
    * Delete the resources to reset the cluster to baseline.

**Critically:** Do not produce drills that have not passed this QA cycle. Tests must be robust!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IanEff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IanEff)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
