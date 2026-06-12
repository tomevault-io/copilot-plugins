## fedora-atomic

> This is a custom Fedora Atomic OS image built with BlueBuild.

# AGENTS.md

## Project Overview

This is a custom Fedora Atomic OS image built with BlueBuild.

**Key Characteristics:**

- **Not a traditional application** - This builds bootable system images, not deployable code
- **Declarative configuration** - Image is defined entirely through YAML recipe files
- **OSTree native containers** - Uses Fedora Atomic's immutable infrastructure approach

## Technology Stack

- **BlueBuild** - Tool for building custom Fedora Atomic images
- **Bash** - Used for scripts
- **YAML** - Recipe and module configuration files
- **GitHub Actions** - CI/CD for automated builds
- **Sigstore cosign** - Image signing

## Schemas (Source of Truth)

All YAML files follow BlueBuild schemas. Reference these for structure and conventions:

- Recipe: `https://schema.blue-build.org/recipe-v1.json`
- Modules: `https://schema.blue-build.org/module-stage-list-v1.json`

Use ONLY those schemas as others are nested specific ones.

## Build Commands

This project has **no local build/test commands** - images are built via GitHub Actions.

**Local validation:**

- Validate YAML syntax against BlueBuild schemas using `check-jsonschema <yaml_file> --schemafile <schema_in_yaml_file> --base-uri https://schema.blue-build.org`
  - <yaml_file> is any yaml file that is either recipe or module
  - <schema_in_yaml_file> is a schema defined in a file in a comment tag `# yaml-language-server: $schema=`
  - All yaml files MUST pass schema validation
- Use `bluebuild` CLI if installed: `bluebuild build recipes/recipe-bazzite.yml`

**CI/CD:**

- Builds trigger automatically on push to main
- Images published to: `ghcr.io/michal-grzelak/bazzite-custom-dx`

## Code Style Guidelines

- **Conventional commits** - Use standard commit message format
- **YAML formatting** - Consistent indentation, validated against schemas
- **Modular design** - Prefer reusable module configurations

## Important Notes

- **Repository structure is being refactored** - Do not hardcode specific file paths
- **No secrets in configs** - All sensitive data (signing keys) are handled via GitHub secrets
- **Immutable infrastructure** - Changes are baked into images, not applied at runtime

## Security Considerations

- Images are signed with cosign. Public key available at `cosign.pub`.
- Never commit sensitive configuration to YAML files
- All builds run in GitHub Actions with isolated environments

## Pattern repo

- Dev tooling is based on this repo: https://github.com/ublue-os/aurora
- Aurora repo uses: https://github.com/get-aurora-dev/common for common system_files, flatpak and logos
- Current repo partly imitates DEV setup from Aurora
- Whenever I ask about verification or checking with Aurora, you MUST verify with those repositories in scope:
  - Dev packages installed in Aurora and those in this repo. If any packages are missing in current repo -> list the gap between and ASK if they should be installed (DON'T do that without asking)
  - Outdated packages installed in current repo -> list them and justify change
  - Outdated scripts, config files, setups for packages -> list them and update configuration in current repo

---
> Source: [michal-grzelak/fedora-atomic](https://github.com/michal-grzelak/fedora-atomic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
