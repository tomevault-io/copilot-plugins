## unification

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**Unification** is a multi-server infrastructure automation system born from real operational pain: **150+ SSH configuration failures** across a 3-server ecosystem. The core insight: 80% of problems stemmed from context limitations (repeating the same mistakes). Solution: intelligent automation with self-discovery, idempotency, and comprehensive testing of "impossible scenarios."

**3-PC Ecosystem Architecture**:
- **HAS** (192.168.0.58, 100.90.137.86): Home Automation Server - "Heart" (orchestration, ZEN Coordinator port 8020)
- **LLMS** (192.168.0.41, 100.68.65.121): LLM Server - "Brain" (Ollama on port 11434)
- **Workstation** (192.168.0.10, Aspire-PC): Development machine - "Hands"
- **Network**: Local 192.168.0.x/24 + Tailscale VPN 100.x.x.x, SSH port 2222 (non-standard)

---

## Development Commands

### Setup & Installation
```bash
# Install package in development mode
pip install -e .

# Install dependencies
pip install -r requirements.txt

# Install dev dependencies
pip install flake8 pytest
```

### Running the System
```bash
# Interactive wizard (main entry point)
python master_wizard.py

# Dry-run mode (safe exploration without making changes)
python wizards/workstation_setup.py --dry-run
```

### Testing
```bash
# Run all tests
python -m unittest discover tests

# Run specific test module
python -m unittest tests.impossible_scenarios.test_ssh_chaos

# Run single test case
python -m unittest tests.impossible_scenarios.test_ssh_chaos.TestSSHChaos.test_port_confusion_nightmare
```

### Linting
```bash
# Run flake8 (CI uses this)
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
flake8 . --count --max-complexity=10 --max-line-length=127 --statistics
```

### CI/CD
GitHub Actions runs on push/PR to main:
- Python 3.9, 3.10, 3.11 matrix
- Installs package with `pip install -e .`
- Lints with flake8
- Runs pytest

---

## Code Architecture

### Core Design Philosophy

**"Test the impossible, automate the repeatable, validate everything"**

1. **Impossible**: Test scenarios that actually occurred (port confusion, permission hell, network partitions, context loops)
2. **Repeatable**: Workstation setup with dry-run and idempotency
3. **Validate**: Comprehensive checks before/after every operation

### Component Interaction Flow

```
master_wizard.py (orchestrator)
    ↓
    ├─→ tools/system_detector.py      # Discover hardware, OS, capabilities
    ├─→ tools/dependency_resolver.py  # Cross-distro package management
    ├─→ tools/network_scanner.py      # Find ecosystem servers
    └─→ tools/config_validator.py     # Validate SSH, tmux, shell configs
         ↓
    wizards/workstation_setup.py (only implemented wizard)
         ↓
    [gather → analyze → plan → confirm → execute → validate]
```

### Key Modules

#### `master_wizard.py` - Entry Point
- Bilingual menu (Czech/English via MenuOption dataclass)
- 5 setup scenarios + 3 operations (ecosystem integration, post-reinstall recovery, health check)
- Orchestrates all tools and wizards
- Dry-run support throughout

#### `tools/system_detector.py` - System Intelligence
```python
SystemInfo dataclass:
  - OS detection via /etc/os-release
  - CPU thermal capabilities (Q9550-specific detection)
  - Virtualization detection (hypervisor, Docker, Podman)
  - Security audit (sudo, SSH, firewall, SELinux/AppArmor)
  - Health checks (disk <10%, memory >90%, CPU load)
  - JSON export
```

#### `tools/dependency_resolver.py` - Package Management
```python
PackageManager enum: APT, YUM, DNF, Pacman, APK, Brew, PIP, Conda, Snap
Package dataclass: OS-specific name mapping
  - 23+ common packages (git, python3, docker, nodejs, etc.)
  - Conflict detection (yum vs dnf)
  - Installation order optimization
  - Time/space estimates via InstallationPlan
```

#### `tools/network_scanner.py` - Network Discovery
```python
ServerInfo dataclass with service identification
  - Parallel scanning (ThreadPoolExecutor, max_workers=50/20)
  - Known ecosystem patterns (60% port match threshold):
    * orchestration: [2222, 8123, 3000, 9000, 8020]
    * llm_server: [2222, 8080, 11434]
  - SSH detection (ports 22 and 2222)
  - NetworkTopology JSON export
```

#### `tools/config_validator.py` - Health Validation
```python
ValidationResult dataclass (ok/warning/error)
  - SSH config validation (file, permissions, server status)
  - Tmux setup validation
  - Shell config validation (.shell_common sourcing)
  - Network validation (localhost, DNS, interfaces)
  - EcosystemValidation with recommendations
```

#### `wizards/workstation_setup.py` - Implementation Example
```python
WorkstationConfig dataclass
Phases:
  1. gather requirements
  2. analyze system
  3. create plan
  4. confirm with user
  5. execute with idempotency
  6. validate completion

Package groups:
  - REQUIRED: git, python3, tmux, curl, htop, ssh
  - DEVELOPMENT: docker, nodejs, gcc, make
  - AI_TOOLS: ollama-llm (conditional)

Features:
  - Dry-run support (--dry-run)
  - Error recovery (SSH config revert on failure)
  - Idempotent checks (don't reinstall if already configured)
```

### Architectural Patterns

1. **Bilingual by Design**: All wizards accept `language` parameter, all strings have en/cz variants
2. **Dataclass-Heavy**: Structured data over dicts (SystemInfo, Package, ServerInfo, ValidationResult, etc.)
3. **Graceful Degradation**: Every detection has try/except with fallback
4. **Logging Everything**: All wizards → file + stdout simultaneously
5. **Dry-run First-Class**: Every modification supports `dry_run=True`
6. **OS-Agnostic Package Naming**: `Package(name="git", apt="git", yum="git", pacman="git", ...)`
7. **Parallel Network Scanning**: ThreadPoolExecutor for speed
8. **Idempotency Checks**: Verify state before modification

---

## Testing Philosophy

### The "SSH Hell" Story

Tests in `tests/impossible_scenarios/test_ssh_chaos.py` (306 lines) encode **actual failure modes** from the original 150+ SSH problems:

```python
# Real scenarios that happened:
test_port_confusion_nightmare()          # Port 22 vs 2222 conflicts
test_permission_hell()                   # 0644 vs 0600 key permissions
test_hostname_resolution_chaos()         # IP vs hostname vs FQDN mismatches
test_network_partition_during_scan()     # Mid-operation network failures
test_simultaneous_ssh_connections()      # Race conditions
test_context_limitation_scenarios()      # Repeated same mistake (context loop)
test_ecosystem_bootstrap_chicken_egg()   # Can't configure servers without SSH, can't SSH without config
test_network_storm_recovery()            # Network flooding recovery
test_complete_ssh_lockout_recovery()     # Total lockout scenarios
```

**Testing standards** (from AGENTS.md):
- Aim for 100% coverage
- Use temp dirs/mocks for filesystem/network ops
- Test naming: `test_*.py` mirrors module names
- Prove graceful failure handling

---

## Coding Standards

From `AGENTS.md`:

- **Language**: Python 3.8+
- **Style**: PEP 8, 4-space indent, max line 127 chars
- **Type hints**: Use where practical
- **Logging**: `logging` module (not print)
- **Naming**: Descriptive class/module names tied to scenarios

**Commit style**: Conventional Commits
```
fix(tests): correct mocking of socket context manager
feat(wizard): add LLM server setup
chore: update dependencies
```

**Security**:
- Never commit SSH keys, secrets, machine-specific configs
- `configs/` contains templates only
- Validate network changes with mocks first

---

## Documentation Standards

### Stavební deník (`docs/STAVEBNI_DENIK.md`)

Chronological infrastructure change log. Required format:

```markdown
## YYYY-MM-DD | Task name

**Stavbyvedoucí**: Who performed work
**Úkol**: What was addressed
**Stav před zahájením**: Initial state

### Provedené diagnostiky
[Details of work performed]

### Zjištěné problémy
[Issues discovered]

### Navrhované řešení
[How to fix]

### Další kroky - čeká na schválení
- [ ] Checklist of next steps

**Datum zpracování**: YYYY-MM-DD
**Čas zahájení**: HH:MM GMT
**Čas ukončení**: HH:MM GMT
**Celkem**: X minutes

**Závěr**: [Summary]
```

**NEVER include**: Weather section, Signature/Approval section (N/A for virtual systems)

### SSH Keys Registry (`docs/SSH_KEYS_REGISTRY.md`)

Central inventory of all SSH keys:
- Active production keys
- Legacy keys (for archival)
- Service keys (GitHub Actions, GCP)
- Access matrix table
- Deployment status
- Migration plan

**Current state**: Primary key is `unified_ecosystem_key_2025` (ED25519), 11 legacy keys pending consolidation.

---

## Development Status

### Implemented ✅
- master_wizard.py orchestration
- All 4 tools modules (system_detector, dependency_resolver, network_scanner, config_validator)
- workstation_setup.py wizard (production-ready)
- Comprehensive "impossible scenarios" test suite
- CI/CD with Python 3.9-3.11 matrix
- Bilingual docs (en/cz)

### Pending ⏳
- llm_server_setup.py wizard (placeholder)
- orchestration_setup.py wizard (placeholder)
- database_setup.py wizard (placeholder)
- monitoring_setup.py wizard (placeholder)
- Ecosystem integration (menu option 6)
- Post-reinstall recovery (menu option 7)

### Known Issues ⚠️

**CRITICAL (2025-12-06)**: Tailscale subnet routing 192.168.0.0/24 → blocks local SSH
- **Fix**: `tailscale up --advertise-routes="" --accept-routes=false` on HAS
- **Root cause**: Local network routed through VPN instead of direct connection
- **Documented in**: `docs/STAVEBNI_DENIK.md`

**Operational**:
- LLMS server offline 8 days
- MiniPC offline 24 days (dead battery)
- Workstation authorized_keys empty (0 bytes)

---

## Q9550-Specific Notes

Special handling for Intel Core 2 Quad Q9550 processor (Workstation CPU):
- Thermal zone detection (`/sys/class/thermal`)
- lm-sensors integration for monitoring
- Power management via cpufrequtils
- References separate PowerManagement project in `/home/milhy777/Develop/Production/PowerManagement`

---

## Related Projects

Part of larger ecosystem (per global `~/.claude/CLAUDE.md`):
- **PowerManagement**: Thermal & CPU management
- **SystemOptimization**: Monitoring & recovery
- **NetworkTools**: Network automation
- **ZEN Coordinator**: MCP services orchestration (port 8020, BETA)
- **MCP Services**: Filesystem (8001), Git (8002), Terminal (8003), Database (8004), Memory (8006), etc.

---

## Operational Context

**All infrastructure changes must be documented** in `docs/STAVEBNI_DENIK.md` following the prescribed format.

**SSH key changes** must update `docs/SSH_KEYS_REGISTRY.md` access matrix.

**Network topology** assumes:
- Local net: 192.168.0.x/24
- Tailscale VPN: 100.x.x.x
- Non-standard SSH port: 2222
- Current routing issue: Check stavební deník for latest status

**Testing ethos**: If it happened in production once, it gets a test. The 150+ SSH failures are encoded as test cases to prevent regression.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/milhy545) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
