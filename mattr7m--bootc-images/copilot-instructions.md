## bootc-images

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fedora bootc image definitions using a base/derivative pattern. Three container images are built with Podman from Containerfiles, layered on `quay.io/fedora/fedora-bootc:43`.

## Image Hierarchy and Build Dependencies

```
fedora-bootc:43
  └── base          (images/base/)
        ├── nidus   (images/nidus/) — AMD Strix Halo AI workstation
        └── dev     (images/dev/)   — Development workstation with Claude Code
```

Derivative images receive the base image via `ARG BASE_IMAGE` build arg. The Makefile enforces this: `build-nidus` and `build-dev` both depend on `build-base`.

## Build Commands

```bash
make build-all                        # Build all images (base → nidus + dev)
make build-base                       # Build base only
make build-nidus                      # Build nidus (auto-builds base)
make build-dev                        # Build dev (auto-builds base)
make lint                             # Hadolint all Containerfiles via podman
make clean                            # Remove built images
make push-all REGISTRY=ghcr.io/user   # Push to registry (REGISTRY required)
sudo make iso-nidus                   # Build Anaconda ISO (requires root)
```

Default `REGISTRY` is `localhost`. Override with `REGISTRY=`, `BASE_TAG=`, `NIDUS_TAG=`, `DEV_TAG=`.

## Architecture Conventions

- **Each image directory** (`images/<name>/`) contains: `Containerfile`, `README.md`, `config.toml.example`, and a `config/` directory for filesystem overlays.
- **Filesystem overlays** in `config/` mirror the root filesystem and are copied into the image with `COPY config/ /`. For example, `images/nidus/config/usr/lib/udev/rules.d/99-amd-kfd.rules` lands at `/usr/lib/udev/rules.d/99-amd-kfd.rules`.
- **All Containerfiles** must end with `RUN bootc container lint` for validation.
- **Podman is not in base** — it's installed in derivative images (nidus, dev) that need it.
- **Nidus runs AI workloads in toolbox containers**, not baked into the host image. GPU access is via udev rules granting the `render` group device permissions.
- **Install-time config** (`config.toml`) is gitignored — it contains password hashes and SSH keys. Each image has a tracked `config.toml.example` that serves as the template. The `config.toml` is manually created on the build system by copying the example. **When changing the config.toml format, update `config.toml.example` in each image directory, the root `README.md`, and the per-image `README.md` files.**
- **Anaconda + pre-existing users**: Anaconda silently skips `[[customizations.user]]` for users that already exist in the image. Credentials (password, SSH key) and hostname are instead applied via a kickstart `%post` script in `[customizations.installer.kickstart]`. See any `config.toml.example` for the format.
- **Scripts** (`scripts/build.sh`, `scripts/push.sh`, `scripts/build-iso.sh`) are helpers invoked by the Makefile or directly. All use `set -euo pipefail`.

## Adding a New Derivative Image

1. Create `images/<name>/Containerfile` with `ARG BASE_IMAGE` and `FROM ${BASE_IMAGE}`
2. Add `config/` directory for filesystem overlays, `config.toml.example` for install-time customization template
3. Add `build-<name>`, `push-<name>`, and `iso-<name>` targets to the Makefile (follow existing patterns)
4. Add the image to the `lint` target's hadolint commands
5. Document in `images/<name>/README.md`

---
> Source: [mattr7m/bootc-images](https://github.com/mattr7m/bootc-images) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
