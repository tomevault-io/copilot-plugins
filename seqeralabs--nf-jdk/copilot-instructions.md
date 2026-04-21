## nf-jdk

> This is a multi-architecture Java JDK container project (`nf-jdk`) that builds two variants of Java containers with different memory allocators, specifically designed for backend execution in the Nextflow ecosystem.

# Build Structure Documentation

## Project Overview

This is a multi-architecture Java JDK container project (`nf-jdk`) that builds two variants of Java containers with different memory allocators, specifically designed for backend execution in the Nextflow ecosystem.

## Build Architecture

### Container Variants

1. **Base Container** (`Dockerfile`)
   - Image: `nf-jdk:corretto-{version}`
   - Standard JDK with basic utilities (tar, gzip, procps, which)
   - Includes `wait-for-it.sh` script

2. **Jemalloc Container** (`Dockerfile_jemalloc`)
   - Image: `nf-jdk:corretto-{version}-jemalloc`
   - Amazon Linux 2023 jemalloc package with native ARM64/AMD64 support
   - Environment: `LD_PRELOAD=/usr/lib64/libjemalloc.so.2`

### Multi-Architecture Support

- **Architectures**: linux/amd64, linux/arm64
- **Build Method**: Docker Buildx with multi-platform support
- **Jemalloc**: Uses AL2023 package manager (native ARM64/AMD64 support)

## Build Targets (Makefile)

- `make all`: Build and push all variants
- `make build`: Build all variants (base + jemalloc)
- `make build-base`: Standard Java container
- `make build-jemalloc`: Container with jemalloc
- `make push`: No-op (images pushed during build)

## Workflow Structure

### Unified Build (.github/workflows/build.yml)

**Triggers:**
- **Schedule**: Daily at 1:00 AM UTC (cron: "0 1 * * *")
- **Manual**: workflow_dispatch for on-demand builds
- **No push builds**: Removed automatic builds on code pushes

**Execution Flow:**
1. **get-versions**: Reads version list from Makefile via `make print-versions` and outputs JSON for the matrix.
2. **Parallel Container Matrix Builds** (matrix from get-versions output):
   - `build-base`: Matrix job - independent base containers
   - `build-jemalloc-container`: Matrix job - uses AL2023 package manager

**Matrix Strategy:**
- **Versions**: Defined in Makefile `VERSIONS`; workflow uses dynamic matrix from `get-versions.outputs.versions`.
- **Total Builds**: 6 container images per run (3 versions × 2 variants) with current default versions.
- **Architecture Support**: All variants support multi-arch (AMD64/ARM64)

## Release Process

### Container Image Publishing
- **Automatic**: Images are pushed during build with `--push` flag
- **No Git tagging**: Removed tag-and-push.sh script and post-build jobs
- **Registry**: cr.seqera.io/public
- **Image naming**: nf-jdk:corretto-{version}[-jemalloc]

### Version Management

- **Single source of truth**: Versions and base image digests are defined in the Makefile (`VERSIONS`, `CORRETTO_*` variables). The workflow reads the version list via `make print-versions` for the dynamic matrix.
- **Generic Dockerfiles**: `Dockerfile` and `Dockerfile_jemalloc` use `ARG BASE_IMAGE`; the Makefile passes the pinned base image per version via `--build-arg BASE_IMAGE=...`.
- **Renovate**: Bumps base image version tags and digests in the Makefile (custom manager matches `VAR = image:tag@sha256:digest`). Major version bumps are disabled; minor/patch versions and digest updates are automatic. Adding a new JDK version requires only Makefile edits (add to `VERSIONS`, add `CORRETTO_*` pin and `base_image_*` mapping).
- **VERSION file**: Used as default for local `make build` when `version=` is not set.

## Development Commands

### Testing Builds Locally
```bash
# Build all variants
make build

# Build specific variant
make build-jemalloc
```

### Manual Builds
```bash
# Trigger build workflow manually via GitHub Actions UI
# or using GitHub CLI:
gh workflow run build.yml

# Check workflow status
gh run list --workflow=build.yml --limit=5
```

## Key Dependencies

- **Registry**: cr.seqera.io/public
- **Base Image**: Amazon Corretto JDK on Amazon Linux 2023; digests pinned in Makefile (`CORRETTO_*` variables) for Renovate digest bumps.
- **Required Secrets**: SEQERA_CR_USERNAME, SEQERA_CR_PASSWORD
- **Build Tools**: Docker Buildx, native compilation scripts

## Performance Optimizations

1. **Package Manager Integration**: Jemalloc uses AL2023 native packages (no compilation needed)
2. **Parallel Execution**: Independent builds run concurrently
3. **Docker Layer Caching**: Buildx optimization for multi-platform builds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seqeralabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
