## nixos

> > **Status**: Draft - Comprehensive Repository Reorganization

# NixOS Repository Restructuring Plan

> **Status**: Draft - Comprehensive Repository Reorganization
> **Created**: 2025-11-01
> **Objective**: Optimize organization, improve accessibility, enhance maintainability

---

## Executive Summary

Este documento estabelece um plano detalhado de reestruturação completa do repositório NixOS. O objetivo é criar uma estrutura modular, intuitiva e escalável que facilite o desenvolvimento, manutenção e colaboração, enquanto mantém alta segurança e compatibilidade.

---

## Current State Analysis

### Repository Structure Overview
```
/etc/nixos/
├── flake.nix (6KB)                    # ✅ Entry point
├── laptop-offload-client.nix (11KB)   # ⚠️ Misplaced - should be in modules
├── .claude/                           # ✅ Claude Code config
├── docs/ (164KB)                      # ✅ Recently organized
├── scripts/ (48KB)                    # ✅ Good location
├── modules/ (592KB, 61 files)         # ⚠️ Needs optimization
├── hosts/ (192KB)                     # ⚠️ Single host, underutilized
├── lib/ (20KB)                        # ⚠️ Could be expanded
├── overlays/ (16KB)                   # ✅ Good structure
├── sec/ (20KB)                        # ⚠️ Confusing separation from modules/security
├── secrets/ (20KB)                    # ✅ SOPS encrypted
├── nixtrap/ (1.8MB, 39 files)         # 🔴 CRITICAL: nested git repo, legacy code
├── reports/ (76KB)                    # ⚠️ Purpose unclear
└── journal/ (32KB)                    # ⚠️ Purpose unclear
```

### Key Metrics
- **Total .nix files in modules**: 61 files
- **Total .nix files in nixtrap**: 39 files (legacy/duplicate)
- **Largest directory**: nixtrap (1.8MB) - contains nested git repository
- **Documentation**: Well organized in docs/
- **Scripts**: 4 utility scripts in scripts/

---

## Identified Issues

### 🔴 Critical Issues

1. **Nested Git Repository (nixtrap/)**
   - Contains separate git repository (git@github.com:VoidNxSEC/nixtrap.git)
   - Should be git submodule or extracted/archived
   - Creates confusion and maintenance overhead
   - 1.8MB of potentially duplicate/legacy code

2. **Security Module Fragmentation**
   - Security configs split between `modules/security/` and `sec/hardening.nix`
   - Unclear hierarchy and override semantics
   - Final override in `sec/hardening.nix` should be better documented

### ⚠️ High Priority Issues

3. **Root-Level Module (laptop-offload-client.nix)**
   - 11KB module sitting in root directory
   - Should be in `modules/services/` or `modules/system/`
   - Breaks organizational pattern

4. **Inconsistent Module Entry Points**
   - Only 5 modules have `default.nix`: shell, network/dns, services, services/users, programs
   - Should standardize: every category should have `default.nix` aggregator
   - Makes imports more verbose and error-prone

5. **Unclear Purpose Directories**
   - `reports/` (76KB) - what reports? Build reports? Audit reports?
   - `journal/` (32KB) - personal notes? System logs? Build logs?
   - Need clear documentation or removal

6. **Module Organization Gaps**
   - No clear separation between user-space and system-level configs
   - Some modules mix concerns (system/services.nix vs services/default.nix)
   - Browser configs split: `modules/browsers/` vs `modules/applications/`

### 💡 Optimization Opportunities

7. **Underutilized Library System**
   - `lib/` only contains packages.nix and shells.nix
   - Could add: helper functions, builders, common patterns
   - Missing: validation functions, type definitions, reusable builders

8. **Single Host Configuration**
   - Only `hosts/kernelcore/` exists
   - Structure prepared for multi-host but underutilized
   - Could better separate host-specific vs shared configs

9. **Documentation Distribution**
   - Most docs in `docs/` but `.claude/CLAUDE.md` contains critical info
   - Should have clear README.md in root
   - Module-level documentation missing

10. **No Testing Infrastructure**
    - No `tests/` directory
    - No integration test framework
    - Checks in flake.nix are minimal (fmt, iso, vm, docker-app)

---

## Restructuring Plan

### Phase 1: Critical Path (Week 1)

#### 1.1 Address Nested Git Repository

**Options for nixtrap/:**
- **Option A (Recommended)**: Convert to git submodule
  ```bash
  git rm -rf nixtrap/
  git submodule add git@github.com:VoidNxSEC/nixtrap.git nixtrap
  ```
- **Option B**: Archive and extract useful modules
  ```bash
  mv nixtrap/ archive/nixtrap-$(date +%Y%m%d)/
  # Extract useful modules to modules/
  ```
- **Option C**: Keep as separate repository reference in docs

**Decision Matrix**:
- Use **Option A** if nixtrap is actively developed separately
- Use **Option B** if nixtrap is legacy and contains useful modules
- Use **Option C** if nixtrap is pure reference material

#### 1.2 Relocate Root-Level Modules

```bash
# Move laptop-offload-client.nix to proper location
mv laptop-offload-client.nix modules/services/laptop-offload-client.nix

# Update flake.nix import
# FROM: ./laptop-offload-client.nix
# TO:   ./modules/services/laptop-offload-client.nix
```

#### 1.3 Clarify Security Module Hierarchy

**Current Structure**:
```
modules/security/*.nix  (imported first)
sec/hardening.nix      (imported last - final override)
```

**Proposed Structure**:
```
modules/security/
├── default.nix              # Import all security modules
├── profiles/
│   ├── base.nix            # Base security (current modules/security/*.nix)
│   ├── hardened.nix        # Hardened profile (current sec/hardening.nix)
│   ├── paranoid.nix        # Maximum security
│   └── development.nix     # Relaxed for dev
├── boot.nix
├── kernel.nix
├── network.nix
└── ...
```

**Benefits**:
- Clear security profile selection
- Better documentation
- Easier to understand override hierarchy
- Can switch profiles per host

### Phase 2: Structural Improvements (Week 2)

#### 2.1 Implement Module Default.nix Standard

Every module category should have `default.nix` that imports all submodules:

```nix
# modules/applications/default.nix
{ ... }:
{
  imports = [
    ./firefox-privacy.nix
    ./brave-secure.nix
    ./vscodium-secure.nix
    ./vscode-secure.nix
  ];
}
```

**Affected Categories**:
- `modules/applications/` - CREATE
- `modules/browsers/` - CREATE (or merge into applications)
- `modules/containers/` - CREATE
- `modules/development/` - CREATE
- `modules/hardware/` - CREATE
- `modules/ml/` - CREATE
- `modules/security/` - ENHANCE
- `modules/system/` - CREATE
- `modules/virtualization/` - CREATE
- `modules/network/` - ENHANCE (already has dns/default.nix)

**Result**: Simplified flake.nix imports
```nix
# FROM:
./modules/applications/firefox-privacy.nix
./modules/applications/brave-secure.nix
./modules/applications/vscodium-secure.nix
./modules/applications/vscode-secure.nix

# TO:
./modules/applications
```

#### 2.2 Reorganize Module Categories

**Merge Redundant Categories**:
```
modules/browsers/ → modules/applications/browsers/
```

**Create Missing Categories**:
```
modules/core/           # Core system configs (currently scattered)
  ├── boot/
  ├── kernel/
  ├── users/
  └── filesystem/

modules/desktop/        # Desktop environment configs
  ├── gnome/
  ├── kde/
  └── wayland/

modules/profiles/       # Complete system profiles
  ├── workstation.nix
  ├── server.nix
  ├── development.nix
  └── minimal.nix
```

**Proposed Final Structure**:
```
modules/
├── core/              # Core system functionality
├── hardware/          # Hardware-specific configs
├── security/          # Security hardening (with profiles/)
├── network/           # Networking (DNS, VPN, bridges)
├── services/          # System services
├── applications/      # User applications (browsers, editors)
├── development/       # Development environments
├── ml/                # Machine learning infrastructure
├── containers/        # Docker, Podman, NixOS containers
├── virtualization/    # VMs, QEMU, libvirt
├── desktop/           # Desktop environment configs
├── shell/             # Shell configuration, aliases
└── profiles/          # Complete system profiles
```

#### 2.3 Expand Library System

**Current**:
```
lib/
├── packages.nix       # Docker images, packages
└── shells.nix         # Development shells
```

**Proposed**:
```
lib/
├── default.nix        # Export all library functions
├── packages.nix       # Docker images, packages
├── shells.nix         # Development shells
├── builders/          # Custom builders
│   ├── docker.nix     # Docker build helpers
│   ├── vm.nix         # VM build helpers
│   └── iso.nix        # ISO build helpers
├── helpers/           # Utility functions
│   ├── security.nix   # Security helper functions
│   ├── network.nix    # Network helper functions
│   └── strings.nix    # String manipulation
└── types/             # Custom types and validators
    └── hardware.nix   # Hardware type definitions
```

### Phase 3: Documentation & Testing (Week 3)

#### 3.1 Create Comprehensive Documentation

**Root Level**:
```
/etc/nixos/
├── README.md                    # CREATE - Repository overview
├── CONTRIBUTING.md              # CREATE - Contribution guidelines
├── ARCHITECTURE.md              # CREATE - System architecture
└── docs/
    ├── README.md                # CREATE - Documentation index
    ├── modules/                 # CREATE - Module documentation
    │   ├── security.md
    │   ├── networking.md
    │   └── development.md
    ├── guides/                  # ORGANIZE existing docs
    │   ├── BINARY-CACHE-SETUP.md
    │   ├── DESKTOP-TROUBLESHOOTING.md
    │   ├── VMCTL-USAGE.md
    │   └── ...
    └── reference/               # CREATE - Reference docs
        ├── flake-structure.md
        ├── module-options.md
        └── security-profiles.md
```

**Module-Level Documentation**:
Each module should have inline documentation:
```nix
{ config, lib, pkgs, ... }:

# Documentation comment block
# Purpose: Brief description
# Dependencies: List of dependencies
# Security impact: Security considerations
# Example usage: How to enable/configure

{
  options.kernelcore.feature = {
    enable = lib.mkEnableOption "feature description";
    # ... options with descriptions
  };

  config = lib.mkIf config.kernelcore.feature.enable {
    # ... implementation
  };
}
```

#### 3.2 Establish Testing Infrastructure

```
tests/
├── integration/               # Integration tests
│   ├── vm-tests/             # NixOS VM tests
│   ├── container-tests/      # Container tests
│   └── network-tests/        # Network configuration tests
├── unit/                     # Unit tests for lib functions
├── security/                 # Security validation tests
│   ├── cve-checks/
│   ├── hardening-tests/
│   └── audit-tests/
└── helpers/                  # Test helper functions
```

**Enhance flake.nix checks**:
```nix
checks.${system} = {
  fmt = ...;                  # Current
  iso = ...;                  # Current
  vm = ...;                   # Current
  docker-app = ...;           # Current

  # NEW
  security-tests = ...;       # Security validation
  module-tests = ...;         # Module unit tests
  integration-tests = ...;    # Integration tests
  documentation = ...;        # Doc building/validation
};
```

#### 3.3 Clarify/Remove Unclear Directories

**Action Items**:

1. **reports/** (76KB)
   - Review contents
   - If audit reports: move to `docs/audits/`
   - If build reports: move to `docs/builds/` or delete (regenerable)
   - If obsolete: archive or delete

2. **journal/** (32KB)
   - Review contents
   - If personal notes: move to `.journal/` (private) or `docs/notes/`
   - If build logs: delete (regenerable)
   - If important: organize into proper docs

### Phase 4: Advanced Optimizations (Week 4+)

#### 4.1 Multi-Host Architecture

**Proposed Structure**:
```
hosts/
├── common/                   # Shared configs across all hosts
│   ├── base.nix             # Base configuration
│   ├── hardware.nix         # Common hardware settings
│   └── users.nix            # Common user settings
├── profiles/                # Host profiles
│   ├── workstation.nix     # Workstation profile
│   ├── server.nix          # Server profile
│   └── laptop.nix          # Laptop profile
├── kernelcore/              # Current main host
│   ├── configuration.nix
│   ├── hardware-configuration.nix
│   └── home/
└── future-host/             # Template for new hosts
    └── configuration.nix
```

**Benefits**:
- Easy to add new hosts
- Shared configs in one place
- Profile-based configuration
- Reduced duplication

#### 4.2 CI/CD Enhancement

**Current**:
```
.gitlab-ci.yml               # GitLab CI
.github/                     # GitHub Actions (partial)
```

**Proposed Enhancements**:
```
.github/
├── workflows/
│   ├── ci.yml              # ENHANCE - Build, test, check
│   ├── security.yml        # CREATE - Security scans
│   ├── docs.yml            # CREATE - Doc generation
│   └── release.yml         # CREATE - Release automation
└── actions/                # CREATE - Custom actions
    ├── nix-build/
    └── security-scan/

ci/                          # CREATE - CI scripts
├── build-all.sh
├── test-security.sh
└── generate-docs.sh
```

#### 4.3 Secret Management Enhancement

**Current**:
```
secrets/
├── ssh-keys/
└── api-keys/ (SOPS encrypted)

modules/secrets/
├── sops-config.nix
└── api-keys.nix
```

**Proposed Enhancement**:
```
secrets/
├── README.md               # CREATE - Secret management guide
├── sops-config/           # ORGANIZE
│   ├── .sops.yaml         # MOVE from root
│   └── age-keys/
├── encrypted/             # ORGANIZE - All encrypted secrets
│   ├── api-keys/
│   ├── ssh-keys/
│   └── certificates/
└── templates/             # CREATE - Secret templates
    └── api-key-template.yaml

scripts/
├── add-secret.sh          # CURRENT
├── rotate-secrets.sh      # CREATE
└── validate-secrets.sh    # CREATE
```

#### 4.4 Development Workflow Improvements

**Create Development Tools**:
```
tools/                     # CREATE - Development tools
├── dev                    # Master dev script
├── module-generator/      # Generate new modules from template
├── host-generator/        # Generate new host configs
└── docs-generator/        # Generate module documentation

templates/                 # CREATE - Templates
├── module/               # Module template
├── host/                 # Host template
└── service/              # Service template
```

**Usage**:
```bash
# Generate new module
./tools/dev new module my-feature

# Generate new host
./tools/dev new host laptop

# Build and test
./tools/dev build
./tools/dev test

# Generate documentation
./tools/dev docs
```

---

## Implementation Strategy

### Execution Phases

#### Phase 1: Critical Path (Days 1-7)
- [ ] Decide on nixtrap/ approach (submodule/archive/reference)
- [ ] Implement chosen nixtrap/ solution
- [ ] Move laptop-offload-client.nix to modules/
- [ ] Restructure security modules with profiles
- [ ] Update flake.nix imports
- [ ] Test rebuild: `sudo nixos-rebuild switch`

#### Phase 2: Structural (Days 8-14)
- [ ] Create default.nix for all module categories
- [ ] Merge modules/browsers/ into modules/applications/
- [ ] Create new module categories (core, desktop, profiles)
- [ ] Expand lib/ with builders and helpers
- [ ] Update flake.nix with simplified imports
- [ ] Test rebuild and validate functionality

#### Phase 3: Documentation & Testing (Days 15-21)
- [ ] Create root-level documentation (README, CONTRIBUTING, ARCHITECTURE)
- [ ] Organize docs/ with clear structure
- [ ] Add inline documentation to all modules
- [ ] Create tests/ infrastructure
- [ ] Enhance flake.nix checks
- [ ] Review and organize reports/ and journal/
- [ ] Generate module documentation

#### Phase 4: Advanced (Days 22+)
- [ ] Implement multi-host architecture
- [ ] Enhance CI/CD pipelines
- [ ] Improve secret management
- [ ] Create development tools
- [ ] Create module/host templates
- [ ] Performance optimization
- [ ] Security audit

### Rollback Strategy

**Before Each Phase**:
```bash
# Create git tag for rollback point
git tag -a phase-N-pre -m "Before Phase N changes"
git push origin phase-N-pre

# Create backup
sudo nixos-rebuild build
```

**If Issues Occur**:
```bash
# Rollback git changes
git reset --hard phase-N-pre

# Rollback system
sudo nixos-rebuild switch --rollback
```

### Success Metrics

**After Phase 1**:
- ✅ No nested git repositories
- ✅ All modules in proper locations
- ✅ Clear security profile hierarchy
- ✅ System rebuilds successfully

**After Phase 2**:
- ✅ All module categories have default.nix
- ✅ Flake.nix imports reduced by >50%
- ✅ No redundant module categories
- ✅ Expanded lib/ with useful helpers

**After Phase 3**:
- ✅ Comprehensive README.md
- ✅ All modules documented
- ✅ Test infrastructure in place
- ✅ CI checks enhanced
- ✅ Clear purpose for all directories

**After Phase 4**:
- ✅ Multi-host architecture ready
- ✅ Enhanced CI/CD pipelines
- ✅ Development tools available
- ✅ Template system for rapid development

---

## Repository Scan: Reorganization Opportunities

### Automated Scan Checklist

When performing the comprehensive repository scan, look for:

#### Code Quality
- [ ] Duplicate code across modules
- [ ] Modules that should be merged
- [ ] Modules that should be split
- [ ] Unused imports
- [ ] Dead code
- [ ] TODO/FIXME comments needing action

#### Security
- [ ] Hardcoded secrets (should be in SOPS)
- [ ] Insecure configurations
- [ ] Missing security options
- [ ] Deprecated security practices
- [ ] Conflicts between security modules

#### Organization
- [ ] Files in wrong locations
- [ ] Inconsistent naming conventions
- [ ] Missing module options documentation
- [ ] Circular dependencies
- [ ] Over-complex module hierarchies

#### Performance
- [ ] Inefficient Nix expressions
- [ ] Unnecessary rebuilds
- [ ] Large derivations that could be split
- [ ] Missing binary cache utilization

#### Documentation
- [ ] Undocumented modules
- [ ] Outdated documentation
- [ ] Missing usage examples
- [ ] Unclear module purposes

### Scan Execution Plan

```bash
# 1. Find duplicate code
find /etc/nixos/modules -name "*.nix" -exec md5sum {} \; | sort | uniq -w32 -D

# 2. Find large modules (candidates for splitting)
find /etc/nixos/modules -name "*.nix" -exec wc -l {} \; | sort -rn | head -20

# 3. Find modules with no options (might need restructuring)
grep -L "mkOption\|mkEnableOption" /etc/nixos/modules/**/*.nix

# 4. Find TODO/FIXME comments
grep -r "TODO\|FIXME\|XXX\|HACK" /etc/nixos/modules --include="*.nix"

# 5. Check for hardcoded secrets
grep -r "password\s*=\|apiKey\s*=\|secret\s*=" /etc/nixos/modules --include="*.nix"

# 6. Find unused imports (complex - manual review)
# Manual review required

# 7. Analyze module dependencies
nix flake show --json | jq '.nixosConfigurations.kernelcore.modules'
```

---

## MCP Server Implementation (2025-11-06)

### Overview

**Location**: [`modules/ml/unified-llm/mcp-server/`](modules/ml/unified-llm/mcp-server/)
**Status**: ✅ Production Ready
**Version**: 2.0.0
**Purpose**: Model Context Protocol server for IDE integration (Roo Code/Cline/Claude Desktop)

### Key Components

1. **MCP Server** (TypeScript)
   - 12 tools total (6 security + 6 knowledge management)
   - JSON-RPC 2.0 compliant
   - stdio transport (no network port)
   - SQLite knowledge database with FTS5 search

2. **Tools Available**:
   - **Security**: provider_test, security_audit, rate_limit_check, build_and_test, provider_config_validate, crypto_key_generate
   - **Knowledge**: create_session, save_knowledge, search_knowledge, load_session, list_sessions, get_recent_knowledge

3. **Documentation**:
   - Health Report: [`docs/MCP-SERVER-HEALTH-REPORT.md`](docs/MCP-SERVER-HEALTH-REPORT.md)
   - Quick Reference: [`.claude/mcp-server-implementation.md`](.claude/mcp-server-implementation.md)
   - Architecture: [`docs/MCP-KNOWLEDGE-STABILIZATION.md`](docs/MCP-KNOWLEDGE-STABILIZATION.md)
   - Implementation: [`docs/MCP-KNOWLEDGE-EXTENSION-PLAN.md`](docs/MCP-KNOWLEDGE-EXTENSION-PLAN.md)

4. **Testing**:
   - Health Check Script: [`scripts/mcp-health-check.sh`](scripts/mcp-health-check.sh)
   - Automated testing framework
   - All 12 tools verified operational

### Integration

**Roo Code/Cline Configuration**:
```bash
~/.config/VSCodium/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json
```

**Quick Setup**:
```bash
mkdir -p ~/.config/VSCodium/User/globalStorage/saoudrizwan.claude-dev/settings/
cp modules/ml/unified-llm/mcp-server-config.json \
   ~/.config/VSCodium/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json
```

### Health Status

Last verified: 2025-11-06

- ✅ All dependencies current (MCP SDK 1.21.0, TypeScript 5.9.3)
- ✅ Build system functional
- ✅ All 12 tools operational
- ✅ Knowledge database initialized
- ✅ Full MCP protocol compliance
- ✅ Ready for production use

### Quick Reference

**Test Commands**:
```bash
# Run health check
bash scripts/mcp-health-check.sh

# Test MCP protocol
cd modules/ml/unified-llm/mcp-server
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | node build/index.js

# Rebuild server
cd modules/ml/unified-llm/mcp-server
npm run build
```

**Related Files**:
- Source: [`modules/ml/unified-llm/mcp-server/src/`](modules/ml/unified-llm/mcp-server/src/)
- Config: [`modules/ml/unified-llm/mcp-server-config.json`](modules/ml/unified-llm/mcp-server-config.json)
- Knowledge Base: [`.claude/mcp-server-implementation.md`](.claude/mcp-server-implementation.md)

---

## Best Practices Established

### Module Development
1. **Every module category must have default.nix**
2. **Use mkOption with description for all options**
3. **Document security implications in comments**
4. **Use mkDefault for overridable defaults**
5. **Use mkForce only in security profiles**
6. **Test in VM before committing**

### Directory Structure
1. **Related files grouped in subdirectories**
2. **Maximum 10 files per directory (create subdirs if more)**
3. **Clear, descriptive directory names**
4. **README.md in each major directory**

### Documentation
1. **Inline documentation in all modules**
2. **README.md for user-facing features**
3. **ARCHITECTURE.md for system design**
4. **Changelog for significant changes**

### Security
1. **Security modules imported last (highest priority)**
2. **Secrets only in SOPS-encrypted files**
3. **Security profiles documented with threat model**
4. **Regular security audits scheduled**

### Testing
1. **All PRs must pass nix flake check**
2. **Security changes must pass security tests**
3. **New modules must include usage examples**
4. **Breaking changes require migration guide**

---

## Next Steps

1. **Review this plan** with team/stakeholders
2. **Prioritize phases** based on immediate needs
3. **Create GitHub issues/project board** for tracking
4. **Begin Phase 1** after approval
5. **Schedule regular review meetings** during implementation

---

## Questions for Resolution

Before starting implementation, decide:

1. **nixtrap/ approach**: Submodule, archive, or reference?
2. **Security profile naming**: base/hardened/paranoid or different names?
3. **Module categories**: Approve proposed structure or modify?
4. **Documentation tooling**: mdBook, Sphinx, or static markdown?
5. **Testing framework**: NixOS tests, or custom framework?
6. **CI/CD platform**: Focus on GitHub Actions, GitLab CI, or both?

---

**Document Version**: 1.0.0
**Last Updated**: 2025-11-01
**Maintained By**: kernelcore
**Review Schedule**: Weekly during implementation, monthly after completion

---
> Source: [VoidNxSEC/nixos](https://github.com/VoidNxSEC/nixos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
