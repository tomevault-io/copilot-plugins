## pactopus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pactopus is a cross-distribution personal package management DSC (Desired State Configuration) tool designed to standardize tool sets across multiple Linux machines and environments. It manages installation and configuration of software packages as composable sets and integrates with dotfiles management.

## Architecture

The project is designed as a CLI bash utility with the following key components:

1. **Main CLI**: `pactopus` command with structure `pactopus [COMMAND] <OPTIONS> arguments`
2. **Backend**: Ansible playbooks for package installation across different distributions
3. **Configuration Storage**: Package tracking in `$XDG_CONFIG_HOME/pactopus/`

## Supported Platforms

Rolling Release (prefer native package managers):
- Arch Linux
- Fedora Workstation/Server
- CentOS Stream
- OpenSUSE Tumbleweed

Stable Release (prefer binaries/alternate sources):
- Debian, Ubuntu and derivatives
- RHEL, Alma Linux, Rocky Linux
- Amazon Linux 2
- Alpine Linux
- OpenSUSE

## Development Commands

Since the project uses Ansible and bash with a Makefile build system:

```bash
# Build/Install from source
make
make install

# Run tests
make test

# Lint code
make lint

# Main CLI commands
pactopus help
pactopus install [package-sets...]
pactopus update
```

## Key Implementation Details

1. **Package Sets**: Composable configuration sets defined in `docs/pactopus-initial-package-lists.md` (Remote Server, Minimal Workstation, Full Workstation)
2. **Dotfiles Integration**: Automatically clones/updates https://github.com/jdubba/dotfiles/ and runs `dotfiles install`
3. **Idempotent Operations**: Both install and update commands must be idempotent
4. **Platform Detection**: Must detect distribution type and choose appropriate installation strategy
5. **Package Name Mapping**: Handle package name variations across distributions (e.g., `bat` vs `batcat`)

## Testing Strategy

When implementing new features:
1. Test across multiple distribution families (Debian-based, RHEL-based, Arch, SUSE)
2. Verify idempotency of install/update operations
3. Check package name mappings work correctly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jdubba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
