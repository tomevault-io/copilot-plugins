## bpm-codingagentconfigcopy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`cac` (Coding Agent Config) is a production-grade CLI tool for managing versioned ZIP-based configuration bundles for AI coding assistants. It supports centralized storage via Gokapi backend or local filesystem, enabling configuration portability across hosts and users.

Supports: Claude Code, Codex CLI, Gemini CLI, and Mistral Vibe.

## Quick Start

```bash
# Install
./install.sh

# Configure backend
cp .env.example ~/.config/cac/.env
chmod 600 ~/.config/cac/.env
# Edit ~/.config/cac/.env with backend settings

# Use
cac push                    # Bundle and upload current user's config
cac pull                    # Download and apply globally newest bundle
cac pull [BUNDLE_ID]        # Download and apply specific bundle
cac list                    # List available bundles
cac test                    # Test AI tool API connectivity
```

## Repository Structure

```
bpm-CodingAgentConfigCopy/
├── bin/
│   └── cac                      # Main CLI entrypoint
├── lib/
│   ├── backend_gokapi.sh        # Gokapi REST API integration (7-day max TTL)
│   ├── backend_local.sh         # Local filesystem backend
│   ├── bundle.sh                # ZIP creation/extraction logic
│   ├── check.sh                 # Credential verification with caching
│   ├── config.sh                # Configuration loading (.env)
│   ├── env.sh                   # AI tool environment management (install/update/status)
│   ├── logging.sh               # Logging utilities (info, warn, error, verbose)
│   ├── security.sh              # Permission checks, zip-slip protection
│   ├── tools.sh                 # Tool-specific file mappings
│   ├── update.sh                # Self-update logic (scope detection, version check)
│   └── utils.sh                 # Shared utilities (JSON parsing, retry, filters)
├── tests/
│   ├── run_tests.sh             # Test runner
│   ├── test_bundle.sh           # Bundle tests
│   ├── test_security.sh         # Security validation tests
│   ├── test_integration.sh      # End-to-end tests
│   ├── test_env_settings.sh     # Claude Code settings.json merge tests
│   └── test_update.sh           # Self-update module tests
├── install.sh                   # Bootstrap installer
├── uninstall.sh                 # Clean removal script
├── cpaiagentconfig.sh           # Legacy single-host copy script
├── .env.example                 # Configuration template
└── README.md                    # User documentation
```

## Architecture

### CLI Commands

| Command | Description |
|---------|-------------|
| `cac push [--user USER] [--skip-check]` | Create ZIP bundle from user configs and upload to backend |
| `cac pull [BUNDLE_ID] [--tool TOOL] [--user USER]` | Download and apply bundle (globally newest, filtered, or specific) |
| `cac get` | Silent alias for `cac pull` (backward compatibility) |
| `cac list [--tool TOOL] [--user USER]` | List available bundles with optional filtering |
| `cac check [TOOL] [--user USER]` | Verify AI tool credentials work (real API calls) |
| `cac test [--user USER]` | Alias for check (backward compatibility) |
| `cac update [--check]` | Self-update cac to the latest version |
| `cac env status [--parseable]` | Show AI tool installation status |
| `cac env install [TOOL] [--global] [--yes] [--tmux]` | Install AI tool environments |
| `cac env update [TOOL]` | Update installed AI tools |

### Library Modules

- **config.sh**: Loads `.env` configuration, validates backend settings, checks file permissions
- **security.sh**: User access checks, file permission validation, zip-slip protection, secure temp directories
- **tools.sh**: Maps AI tools to their configuration files (dual registry: credentials + settings), collects/counts existing files
- **bundle.sh**: ZIP creation with correct naming convention, secure extraction with backups
- **backend_local.sh**: Local filesystem storage operations (upload, download, list, get_newest)
- **backend_gokapi.sh**: Gokapi REST API operations (upload, download, list, get_newest, delete); enforces 7-day max TTL
- **check.sh**: Credential verification via real API calls; 5-minute cache, 10-second timeout per provider
- **env.sh**: AI tool environment management; install/update/status for Claude, Codex, Gemini, Mistral Vibe, continuous-claude; `--tmux` flag sets `teammateMode` in settings.json
- **logging.sh**: Structured logging (info, warn, error, verbose, spinner)
- **update.sh**: Self-update logic; detects install scope (user/global), compares local vs remote version, downloads and re-runs install.sh
- **utils.sh**: Shared utilities (JSON field extraction, retry with backoff, filter parsing, command context)

### Bundle Naming Convention

```
CodingAgentConfig_<HOST>_<USER>_<YYMMDD-HHMMSS>.zip
```

### Credential Files (portable across hosts)

| Tool | Files |
|------|-------|
| Claude Code | `.claude.json`, `.claude/.credentials.json` |
| Codex CLI | `.codex/auth.json` |
| Gemini CLI | `.gemini/oauth_creds.json`, `.gemini/google_accounts.json`, `.gemini/settings.json`, `.gemini/state.json`, `.gemini/installation_id`, `.config/gcloud/application_default_credentials.json` |
| Mistral Vibe | `.vibe/.env`, `.vibe/config.toml` |

### Settings Files (host+user-specific, NOT portable)

| Tool | Files |
|------|-------|
| Claude Code | `.claude/settings.json` |

Settings are bundled with `cac push` but only extracted by `cac pull` when the bundle's hostname AND username match the target. This prevents host-specific config (e.g. `teammateMode: "tmux"`) from being overwritten by bundles from other hosts.

## Installation Paths

| Mode | Binary | Libraries | Config |
|------|--------|-----------|--------|
| System-wide (root) | `/usr/local/bin/cac` | `/usr/local/lib/cac/` | `/etc/cac/.env` |
| User-local | `~/.local/bin/cac` | `~/.local/lib/cac/` | `~/.config/cac/.env` |

## Banned Dependencies

- **Bun is BANNED.** Do not use Bun anywhere in this project — no `bun add`, no `bun update`, no `bun install`, no `/opt/claude-code` bun-based paths. Use `npm` for package management and official `curl | bash` installers for tool installation. Existing Bun code (Issue #56) is legacy debt being removed.

## Security Requirements

- `.env` must have 600 permissions; CLI refuses to run if too open
- Cross-user operations require root privileges
- ZIP extraction validates against path traversal attacks
- Extracted files set to 600 permissions
- Temporary files use secure permissions (700 directories, 600 files)

## Development

### Running Tests

```bash
# Install dependencies
sudo apt-get install zip unzip

# Run all tests
./tests/run_tests.sh

# Run specific suite
./tests/test_bundle.sh
./tests/test_security.sh
./tests/test_integration.sh
```

### Linting

```bash
shellcheck bin/cac lib/*.sh install.sh uninstall.sh tests/*.sh
```

### Key Functions

**bundle.sh:**
- `bundle_generate_filename(user)` - Creates standard filename
- `bundle_create(home_dir, output_file, tool)` - Creates ZIP bundle
- `bundle_extract(zip_file, home_dir, username)` - Extracts with security validation

**security.sh:**
- `security_check_user_access(target_user)` - Validates permissions
- `security_validate_zip(zip_file, target_dir)` - Zip-slip protection
- `security_mktemp_dir(prefix)` - Creates secure temp directory

**backend_local.sh / backend_gokapi.sh:**
- `backend_*_upload(bundle_file)` - Upload bundle
- `backend_*_download(bundle_id, output_file)` - Download bundle
- `backend_*_list([--tool TOOL] [--user USER])` - List bundles
- `backend_*_get_newest([--tool TOOL] [--user USER])` - Get newest matching

**check.sh:**
- `check_single_tool(tool, user)` - Verify one tool's credentials with caching
- `check_all_tools(user)` - Verify all configured tools
- `check_tool_claude/codex/gemini(user)` - Provider-specific verification

## Milestone-Based Issue Lifecycle (MANDATORY)

Every issue in this repository MUST follow the milestone-based lifecycle. No exceptions.

| Milestone | Set By | Meaning |
|-----------|--------|---------|
| `new` | Team Lead | Issue created, not yet planned |
| `planned` | Team Lead | Agent submitted a plan (posted as issue comment) |
| `plan-approved` | Team Lead + Codex | Both reviewed and approved the plan |
| `test-designed` | Team Lead | Agent submitted test design as issue comment |
| `test-design-approved` | Team Lead + Codex | Both approved test design |
| `implemented` | Team Lead | Code written, agent reports completion |
| `tested-success` | Team Lead | All tests pass |
| `tested-failed` | Team Lead | Tests fail — bounces back with documented reason |
| `test-approved` | Team Lead + Codex | Final automated gate — independent verification passed |
| `DONE` | **Human only** | Final sign-off. Agents NEVER set this. |

**Lifecycle flow:**
```
new -> planned -> plan-approved -> test-designed -> test-design-approved
  -> implemented -> tested-success / tested-failed -> test-approved -> DONE
```

**Rules (Non-Negotiable):**
1. One milestone at a time per issue — no skipping states
2. Dual approval required at every gate — Team Lead AND Codex must both approve
3. `DONE` is human-only — agents must NEVER set this milestone
4. One issue per discrete change — all phases documented as comments on that issue
5. Every Codex response posted as comment on the GitHub Issue

See `/my-team-milestones` skill for full details including Codex gate patterns and compact lifecycle variant.

## Multi-Agent Workflow

This project uses a multi-agent development workflow:

- **Claude** - Primary orchestrator and main executor
- **Codex** - Primary review and approval authority
- **Gemini** - Consensus and fallback reviewer

### Segregation of Duty (SoD)

**Default Rule:** No LLM may review or approve work it has performed itself.

**Controlled Exception:** Claude may self-review only if:
1. Work was NOT performed by Claude
2. Codex is rate-limited or unavailable
3. Gemini is rate-limited or unavailable
4. Exception is documented in the issue with "SEGREGATION OF DUTY EXCEPTION APPLIED"

### Consensus and Fallback Logic

- When Claude and Codex disagree, Gemini provides independent assessment
- Codex rate-limit fallback: gpt-5.1-codex-mini → Gemini → Claude (with SoD exception)
- Gemini rate-limit fallback: gemini-2.5-flash-lite → Codex → Claude (with SoD exception)
- Claude rate-limit: STOP - no release allowed without orchestrator

### Required Approvals Before Git Push

All must be documented in the issue:
1. PLAN AND AGENT/SKILL ASSIGNMENT APPROVED
2. IMPLEMENTATION APPROVED
3. TEST DESIGN APPROVED
4. TEST RESULTS APPROVED
5. DOCUMENTATION UPDATED AND CONSISTENT APPROVED

### Related Documentation

- [agent.md](agent.md) - Full multi-agent model, skill selection, SoD rules, approval workflow
- [gemini.md](gemini.md) - Consensus process, fallback logic, exception handling

---
> Source: [BPMspaceUG/bpm-CodingAgentConfigCopy](https://github.com/BPMspaceUG/bpm-CodingAgentConfigCopy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
