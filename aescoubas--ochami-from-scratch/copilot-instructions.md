## ochami-from-scratch

> *   **OCI image builds:** `images/<service>/Dockerfile` (built via buildah/docker)

# OpenCHAMI Agent Guidelines

## Context
*   **OCI image builds:** `images/<service>/Dockerfile` (built via buildah/docker)
*   **Deployment artifacts:** `deploy/compose/`, `deploy/quadlets/`, `deploy/helm/`
*   **Deployment profiles:** `profiles/*.env` (official, dev, cscs)
*   **Operational scripts:** `scripts/ops/` (bash -- deploy, teardown, health-check, build-images, lab, etc.)
*   **Virtual test lab:** `scripts/ops/lab.sh` (AlmaLinux server VM + PXE client VMs via libvirt)
*   **RPM packaging:** `ochami-from-scratch.spec` + `Makefile` rpm targets (package name: `openchami`)
*   **MCP server:** `ochami/mcp/` (standalone Python, stdlib only)
*   **Cloud-init config:** `cloud-init/` (cluster defaults, compute group cloud-config pushed to metadata service)
*   **Docs:** `README.md`, `AGENTS.md`, `docs/architecture/`, `docs/plans/ROADMAP.md`
*   **Tests:** `tests/` (pytest via `make test`)

## Architecture
*   OCI images are built from Dockerfiles in `images/<service>/Dockerfile` using buildah or docker. Go services that vendor glibc-linked libraries (cloud-init with DuckDB, PCS with confluent-kafka) use `golang:*-bookworm` builders; others use `golang:*-alpine`.
*   Deployment profiles (`profiles/*.env`) define version refs, image prefix, and optional registry. Available profiles: `official` (default, `localhost/*`), `dev` (moving main refs), `cscs` (CSCS JFrog, `cscs-` prefix).
*   Deployment artifacts (docker-compose.yml, .container files, values.yaml) are directly maintained in `deploy/`. These are the source of truth -- there are no generators.
*   Quadlet `.container` files reference binaries at `/usr/local/bin/` for locally-built Go services and `/bin/` for CSCS Nix-built images (`jfrog.svc.cscs.ch/...`). System packages (kea) use `/usr/sbin/`. Keep `Exec=` paths in sync with the actual image binary locations.
*   Runtime configs require secret interpolation at deploy/activation time via `envsubst`; config templates keep placeholders until deploy time.
*   Runtime operations (deploy, teardown, health checks, node registration) are handled by bash scripts in `scripts/ops/`.
*   The **virtual test lab** (`make lab`) creates an AlmaLinux 9 server VM running the RPM + quadlet deployment, with PXE client VMs on a dedicated libvirt network (`ochami-lab-net`, 192.168.200.0/24). State is stored in `~/.local/state/openchami/lab/`.
*   The **Docker Compose PXE path** (`make deploy METHOD=compose`) runs services on the host with libvirt network `ochami-pxe-net`, bridge `virbr-ochami`, and `scripts/ops/create-test-vms.sh` for test VM bootstrap.
*   Local OCI image builds are orchestrated by `scripts/ops/build-images.sh`; `make build-images` and `make deploy` accept `SMD_SRC`, `BSS_SRC`, `PCS_SRC`, `CLOUD_INIT_SRC`, and `KEA_SYNC_SRC` to build specific services from local checkouts.
*   `make teardown METHOD=compose` removes compose containers and volumes and restores paused libvirt DHCP networks, but it does **not** delete libvirt test VMs or their qcow2 disks.
*   **Boot images** for PXE nodes are built in the sibling `image-building/openchami-image-building` repository using `mkosi`. It produces a compressed cpio archive (openSUSE Leap 15.6) that the kernel unpacks directly into RAM. NetworkManager handles DHCP in userspace (wicked is disabled via systemd preset). Cloud-init contacts the metadata service for per-node configuration (hostname, DNS, packages). The push script deploys artifacts to the OpenCHAMI HTTP server and registers BSS boot parameters.
*   **Cloud-init configuration** lives in `cloud-init/` and is pushed to the in-memory metadata service (port 27777) via its admin API. Data must be repopulated after any service restart. The compute group cloud-config handles DNS override via NetworkManager, zypper GPG key auto-import, and package installation.
*   Architecture overviews and ADRs belong under `docs/architecture/`, not a top-level `ARCHITECTURE/` directory.
*   The MCP server (`ochami/mcp/`) is self-contained with zero external dependencies (stdlib only).
*   There is no Python CLI -- the only Python entry point is `ochami-mcp`.
*   Four deployment paths are supported: virtual lab (`make lab`), compose (`make deploy METHOD=compose`), quadlets/RPM (`make rpm`), and helm (`make deploy METHOD=minikube`).

## Mandates
1.  **Roadmap Driven:** Always check `docs/plans/ROADMAP.md` before starting work. Mark tasks as `[x]` only after verification.
2.  **Test First:** Create/Update tests before implementing features.
3.  **Architecture Docs Location:** Keep architecture notes and ADRs under `docs/architecture/`.
4.  **Style:** Follow existing coding style. Bash scripts use `set -euo pipefail` and source `lib/common.sh`.
5.  **Dockerfile / Quadlet Consistency:** When changing binary install paths in Dockerfiles, update the corresponding `Exec=` lines in `deploy/quadlets/containers/*.container` and vice versa.
6.  **Operational Safety:**
    *   **Non-Interactive:** Use flags to suppress prompts (e.g., `apt-get -y`).
    *   **No Watch Modes:** Never run commands that block forever unless explicitly requested as a background daemon.
    *   **Compose PXE Host Assumptions:** The compose PXE path expects host tools such as Docker, libvirt, `envsubst`, and passwordless `sudo` for bridge/libvirt network preparation.
7.  **Git:**
    *   **Commit Message Standard:** Use Conventional Commits (`type(scope): description`).
        *   Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`.
        *   Example: `feat(lab): add client VM PXE boot support`
    *   **Descriptive:** The message must clearly explain *what* changed and *why*. Avoid vague messages like "fix".
    *   Never commit broken code.

## Verification
*   Always run the build/test suite before finishing a turn.
*   Command: `make test`
*   Image builds: `make build-images` or `make build-images PROFILE=cscs`
*   RPM packaging: `make rpm` (official) or `make rpm PROFILE=cscs` (CSCS JFrog)
*   Lab status: `make lab-status`
*   GitHub Actions is out of scope for this repository. Keep verification local unless explicitly requested otherwise.

---
> Source: [aescoubas/ochami-from-scratch](https://github.com/aescoubas/ochami-from-scratch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
