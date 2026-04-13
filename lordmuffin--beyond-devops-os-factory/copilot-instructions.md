## beyond-devops-os-factory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **beyond-devops-os-factory**, an enterprise-grade OS image management automation framework. The project focuses on building reproducible **Kairos OS** and **Windows 11 Pro** images using Infrastructure as Code (IaC) principles.

## Core Technologies & Architecture

- **Kairos Factory Action**: Simplified Kairos image building using official factory action
- **Packer**: OS image building and automation (for Windows builds)
- **Ansible**: Configuration management and provisioning
- **GitHub Actions**: CI/CD pipeline automation with semantic versioning
- **Security-first approach**: Built-in security scanning with Grype and Trivy

## Current Implementation Status

**✅ COMPLETED - Kairos Build System:**
- Kairos Factory Action integration for simplified builds
- Semantic versioning with automatic git tag-based releases
- Multi-format support (ISO, RAW, QCOW2) with amd64/arm64 architectures
- Enterprise configuration with K3s, monitoring, and security bundles
- Comprehensive documentation (INSTALL-KAIROS.md, VERSION.md, CHANGELOG.md)

**🚧 IN PROGRESS - Windows Build System:**
- Packer templates for Windows 11 Pro
- Ansible playbooks for configuration management
- GitHub Actions workflows for automated testing

## Active Features

- **Automated Kairos Builds**: Push commits → automatic dev versions, git tags → official releases
- **Enterprise Security**: Integrated vulnerability scanning and compliance
- **Multi-platform Support**: amd64 and arm64 architecture support
- **Professional Documentation**: Complete installation, versioning, and changelog management

## Key Principles

- **Security-first**: All image builds should include security hardening
- **Reproducibility**: Images must be consistently reproducible across environments  
- **Enterprise-grade**: Solutions should meet enterprise compliance and operational requirements
- **IaC methodology**: All infrastructure and configuration should be code-driven

# CRITICAL: Version Management & Release Process

## MANDATORY: Follow Semantic Versioning Protocol

This project uses **automatic semantic versioning** based on git tags. ALL changes MUST follow the established versioning and documentation procedures:

### Required Before ANY Release-Affecting Changes

**1. UPDATE CHANGELOG.md** following Keep a Changelog format:
```markdown
## [Unreleased]
### Added
- New feature or capability description
### Changed  
- Modified functionality description
### Fixed
- Bug fix description
### Security
- Security-related fixes
```

**2. FOLLOW VERSION.md guidelines** for proper release categorization:
- **PATCH (x.y.Z)**: Bug fixes, security patches, minor improvements
- **MINOR (x.Y.z)**: New features, bundles, backwards-compatible changes  
- **MAJOR (X.y.z)**: Breaking changes, architecture changes, incompatible changes

**3. USE PROPER GIT TAGS** for official releases:
```bash
# Correct: Annotated tags with descriptive messages
git tag -a v1.1.0 -m "Minor release: Add Prometheus and Grafana bundles"
git push origin v1.1.0

# Incorrect: Lightweight tags without messages
git tag v1.1.0  # DON'T DO THIS
```

## Mandatory Workflow Procedures

### Development Changes
```bash
git add .
git commit -m "descriptive commit message"
git push origin main
# → Automatically generates: v1.0.0-dev.abc123 (development version)
```

### Official Releases  
```bash
# 1. Update CHANGELOG.md first
# 2. Create annotated tag
git tag -a v1.1.0 -m "Release v1.1.0: Description of changes"
git push origin v1.1.0
# → Automatically generates: v1.1.0 + Creates GitHub Release
```

## Documentation Standards

**When modifying ANY project functionality:**
1. **ALWAYS** update CHANGELOG.md with user-facing changes
2. Reference VERSION.md for proper version bump planning
3. Update INSTALL-KAIROS.md if installation procedures change
4. Document breaking changes with migration instructions

**Release Documentation Requirements:**
- All releases MUST have corresponding CHANGELOG.md entries
- Breaking changes require detailed migration documentation
- New features must be documented in installation guides
- Security fixes must be explicitly categorized and noted

## File Change Guidelines

**Configuration Changes:**
- `packer/kairos/osartifact.yaml` → Minor version (new bundles/features)
- `packer/kairos/cloud-config.yaml` → Minor version (new configs) or Patch (fixes)
- `.github/workflows/kairos-factory.yml` → Patch version (fixes) or Minor (new features)

**Breaking Changes (Major Version Required):**
- Removing existing bundles or configurations
- Changing API interfaces or configuration formats
- Modifying default behaviors that affect existing deployments
- Updating to incompatible base images or Kairos versions

## Reference Documentation

- **VERSION.md**: Complete semantic versioning guide with examples and best practices
- **CHANGELOG.md**: Project change history following Keep a Changelog format  
- **INSTALL-KAIROS.md**: Comprehensive installation and customization guide

# Important Development Instructions

## Core Guidelines
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

## EXCEPTION: Documentation Requirements
**ALWAYS update CHANGELOG.md** when making changes that affect:
- Releases or versioning
- User-facing functionality  
- Configuration files
- Breaking changes
- Security fixes

This ensures proper release documentation and maintains project changelog integrity per VERSION.md requirements.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lordmuffin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lordmuffin)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
