## devrunner

> This repository contains Docker container images for Chaitin MonkeyCode developer workflows. The structure is organized as follows:

# Repository Guidelines

## Project Structure & Module Organization

This repository contains Docker container images for Chaitin MonkeyCode developer workflows. The structure is organized as follows:

- `docker/base/bookworm/` - Base Debian bookworm-slim image with essential development tools
- `docker/devbox/bookworm/` - All-in-one devbox with Go, Node.js, Python, and common packages (extends base)
- `docker/frontend/node20/` - Node.js 20 frontend development image (extends base)
- `docker/golang/1.25-bookworm/` - Go 1.25 development image (extends base)
- `scripts/build.sh` - Environment-driven build script for all images
- `.github/workflows/ci.yaml` - CI/CD pipeline for automated builds and pushes
- `README.md` - Project documentation and usage instructions

Each stack directory contains a single `Dockerfile` defining that specific image.

## Build, Test, and Development Commands

### Building Images Locally
```bash
# Base image
STACK=base VERSION=bookworm ./scripts/build.sh

# Devbox all-in-one image
STACK=devbox VERSION=bookworm ./scripts/build.sh

# Frontend Node.js image
STACK=frontend VERSION=node20 ./scripts/build.sh

# Go development image
STACK=golang VERSION=1.25-bookworm ./scripts/build.sh
```

### Pushing to Registry
```bash
PUSH=true REGISTRY=ghcr.io/chaitin/monkeycode-runner STACK=base VERSION=bookworm ./scripts/build.sh
```

### Running Images
```bash
# Base image
docker run --rm -it ghcr.io/chaitin/monkeycode-runner/base:bookworm bash

# Devbox all-in-one image
docker run --rm -it ghcr.io/chaitin/monkeycode-runner/devbox:bookworm bash

# Frontend image
docker run --rm -it ghcr.io/chaitin/monkeycode-runner/frontend:node20 node --version

# Go image
docker run --rm -it ghcr.io/chaitin/monkeycode-runner/golang:1.25-bookworm bash
```

### CI/CD Pipeline
The GitHub Actions workflow automatically:
- Builds all images on pull requests (no push)
- Builds and pushes images on main branch merges
- Supports multi-architecture builds (amd64/arm64)
- Tags images with version, branch, and `latest` as appropriate

## Coding Style & Naming Conventions

### Dockerfile Conventions
- Use `debian:bookworm-slim` as base for the foundational image
- All derived images extend the base image
- Include proper OCI labels (`org.opencontainers.image.*`)
- Use `ARG DEBIAN_FRONTEND=noninteractive` for non-interactive builds
- Set `WORKDIR /workspace` for consistency
- Architectural support for amd64 and arm64

### Directory Naming
- Stack directories use lowercase: `base/`, `frontend/`, `golang/`
- Version naming follows tool conventions: `bookworm`, `node20`, `1.25-bookworm`
- Dockerfile paths: `docker/{stack}/{version}/Dockerfile`

### Shell Script Style
- Use `#!/usr/bin/env bash` shebang
- Enable strict mode: `set -euo pipefail`
- Environment variables with sensible defaults
- Clear error messages to stderr

## Testing Guidelines

### Local Testing
Test images locally before pushing:
```bash
# Build and test basic functionality
STACK=base VERSION=bookworm ./scripts/build.sh
docker run --rm -it ghcr.io/chaitin/monkeycode-runner/base:bookworm bash -c "git --version && python3 --version"

# Test specific tools
docker run --rm -it ghcr.io/chaitin/monkeycode-runner/frontend:node20 node --version
docker run --rm -it ghcr.io/chaitin/monkeycode-runner/golang:1.25-bookworm go version
```

### CI Testing
- All images must build successfully in GitHub Actions
- Matrix builds test all stacks concurrently
- Multi-architecture support is verified automatically

## Commit & Pull Request Guidelines

### Commit Message Format
Follow conventional commit format observed in the project:
```
Merge pull request #3 from xiaomakuaiz/feature/frontend-node

Add Node frontend stack and update CI
```

Feature branches should use descriptive names:
- `feature/frontend-node`
- `feature/golang-update`
- `fix/base-image-security`

### Pull Request Requirements
- Title should clearly describe the change
- Description should explain the purpose and impact
- Include testing instructions for new features
- Ensure all images build successfully in CI
- Update README.md if adding new stacks or changing usage
- Target the `main` branch for all changes

### Security Considerations
- Verify checksums for all downloaded binaries
- Use minimal base images (`bookworm-slim`)
- Clean up package manager caches (`rm -rf /var/lib/apt/lists/*`)
- Run as non-root user when appropriate (current images use root for development flexibility)

## Development Workflow

1. Create feature branch from `main`
2. Add new Dockerfile in appropriate `docker/{stack}/{version}/` directory
3. Update `scripts/build.sh` if needed (usually not required)
4. Update CI matrix in `.github/workflows/ci.yaml` for new stacks
5. Update `README.md` with usage instructions
6. Test locally with provided commands
7. Submit pull request following guidelines above

## Registry Configuration

Default registry: `ghcr.io/chaitin/monkeycode-runner`
Image naming: `{registry}/{stack}:{version}`
Examples:
- `ghcr.io/chaitin/monkeycode-runner/base:bookworm`
- `ghcr.io/chaitin/monkeycode-runner/devbox:bookworm`
- `ghcr.io/chaitin/monkeycode-runner/frontend:node20`
- `ghcr.io/chaitin/monkeycode-runner/golang:1.25-bookworm`

---
> Source: [chaitin/DevRunner](https://github.com/chaitin/DevRunner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
