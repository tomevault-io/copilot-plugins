## workspace

> This repository builds a containerized virtual desktop environment  (Workspace) for the DTaaS (Digital Twin as a Service) platform. The container provides multiple services through a web-based interface including KasmVNC for desktop access, Jupyter notebooks, VS Code   Server, and Firefox browser.

# Copilot Instructions for Containerized Virtual Desktop Environment

## Repository Overview

This repository builds a containerized virtual desktop environment  (Workspace) for the DTaaS (Digital Twin as a Service) platform. The container provides multiple services through a web-based interface including KasmVNC for desktop access, Jupyter notebooks, VS Code   Server, and Firefox browser.

## Project Structure

### `.` - Project root
Contains files more related to the repository than the project artifact. This includes various linting configurations, the README, the LICENSE, the CODE_OF_CONDUCT and so on. Apart from this, it also contains:
- `scripts/` - Contains various scripts useful in the development and maintanence of the repository.
- [`workspaces/`](#workspaces-main-artifact-directory) - Contains files related more to the project artifact than the repository infrastructure.


### `workspaces/` - Main artifact directory
Contains all files related to the Workspace image artifact.
- [`src/`](#workspacessrc-workspace-image-source-files) - Source files needed to build the image.
- `test/` - Files related to testing the image.
- `Dockerfile.*` - The dockerfile(s) used in building the image. (must be linted with hadolint)

#### `workspaces/src/` - Workspace image source files
- `admin/` - Admin service for workspace service discovery (FastAPI application)
- `install/` - Files used as part of the image build process.
- `resources/` - Static files injected into the image during building.
- `startup/` - Files used during image startup, bootstrapping the workspace, configuring it dependent on container runtime environment variables.

**Script Requirements:**
- All bash/zsh scripts must be linted with shellcheck
- All Python scripts must be linted with pylint/flake8
- Scripts must follow proper style conventions for their respective languages
- Include error handling and proper exit codes

**Startup Script Requirements:**
- Must be executable and properly handle signals
- Should support graceful shutdown
- Follow the same linting requirements as installation scripts

#### `workspaces/test/dtaas/` - DTaaS specific testing
Contains files needed to test the integration of the workspace into existing DTaaS infrastructure.
- `certs/` - Holds certificates and related files.
- `config/` - Holds configuration files for the Docker compositions along with example files.
- `dynamic/tls.yml` - Dynamic injected configuration for the Traefik reverse proxy used by some of the compositions.
- `files/` - Persistent storage structure, mounted as part of composition setup.
- `compose.*` - Docker compose files for different workspace usecases. (must be validated with docker compose config)
- `TRAEFIK*.md` - Guides for using and setting up the different multiuser Docker compositions.
- `CONFIGURATION.md` - General guide for configuring the DTaaS-like compositions.

## Development Guidelines

### Code Quality Standards

#### Shell Scripts (bash/zsh)
- Use shellcheck for linting
- Follow Google Shell Style Guide
- Include proper shebang lines
- Use `set -e` for error handling
- Quote variables appropriately
- Use meaningful variable names

#### Python Scripts
- Use pylint or flake8 for linting
- Follow PEP 8 style guide
- Include docstrings for functions and modules
- Use type hints where appropriate

#### Python Projects (admin service)
- Use Poetry for dependency management
- Run tests with pytest before committing
- Maintain test coverage above 75%
- Ensure all tests pass: `poetry run pytest --cov`
- Lint with pylint: `poetry run pylint src tests`

#### Dockerfile
- Use hadolint for linting
- Pin specific versions for base images and packages
- Minimize layers by combining RUN commands where logical
- Use multi-stage builds if appropriate
- Clean up package manager caches
- Use non-root user when possible

#### Docker Compose
- Validate with `docker compose config`
- Use version 3.x syntax
- Define explicit service dependencies
- Use environment variables for configuration
- Include volume mounts for persistent data

### File Modifications

When modifying files:
1. **Installation scripts** - Ensure all dependencies are installed and versions are pinned
2. **Configuration files** - Validate syntax and compatibility with service versions
3. **Dockerfile** - Run linting and ensure build succeeds
4. **compose.\*yml** - Validate configuration before committing

### Testing

Before committing changes:
- Lint all modified scripts
- Test Docker image builds successfully
- Verify services start correctly in the container
- Check that persistent directories are properly mounted

### Common Tasks

#### Adding a New Software Component
1. Create installation script in `workspaces/src/install/<component>/`
2. Add configuration to `workspaces/src/resources/<component>/` if needed
3. Update startup scripts (`workspaces/src/startup/`) with neccessary bootstrapping.
4. Update Dockerfile to call the installation script
5. Update README.md with component information

#### Modifying the Admin Service
1. Make changes to `workspaces/src/admin/src/admin/`
2. Update tests in `workspaces/src/admin/tests/`
3. Run tests: `cd workspaces/src/admin && poetry run pytest --cov`
4. Run linting: `poetry run pylint src/admin tests`
5. Update documentation in README.md and DOCUMENTATION.md
6. Rebuild and reinstall: `poetry build && poetry install`

#### Modifying Startup Behavior
1. Edit or add scripts in `workspaces/src/startup/``
2. Ensure scripts are executable
3. Test container startup and shutdown
4. Verify all services initialize correctly

## Linting Commands

When suggesting or making changes, ensure these commands pass:

```bash
# Shell scripts
shellcheck workspaces/src/**/*.sh

# Dockerfile
hadolint workspaces/Dockerfile.ubuntu.noble.xfce

# Docker Compose
docker compose -f workspaces/test/dtaas/compose.yaml config
docker compose -f workspaces/test/dtaas/compose.traefik.yaml config
docker compose -f workspaces/test/dtaas/compose.traefik.secure.yaml config
docker compose -f workspaces/test/dtaas/compose.traefik.secure.tls.yaml config

# Python scripts (if any)
pylint **/*.py
flake8 **/*.py

# Admin service (Python FastAPI project)
cd workspaces/src/admin
poetry run pytest --cov=admin --cov-report=term-missing
poetry run pylint src/admin tests
```

## Best Practices

- Always use absolute paths in scripts
- Handle errors gracefully with proper exit codes
- Log important operations for debugging
- Use environment variables for configurable values
- Document any non-obvious logic with comments
- Test changes in a clean container environment

---
> Source: [INTO-CPS-Association/workspace](https://github.com/INTO-CPS-Association/workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
