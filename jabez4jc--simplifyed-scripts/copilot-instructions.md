## simplifyed-scripts

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive collection of bash scripts for managing OpenAlgo trading platform instances on Ubuntu/Debian servers. The scripts automate multi-instance deployment, configuration, monitoring, backup, and updates.

## Architecture

**Core Components:**

1. **quick-setup.sh** - Single instance setup script
   - Automated complete setup in one command
   - Configures 4GB swap automatically
   - Includes all system packages, dependencies, SSL, Nginx, systemd service
   - Interactive prompts for domain, broker, and credentials
   - Best for single instance deployments and quick testing

2. **multi-install.sh** - Main orchestration script
   - Validates system prerequisites (Python 3, pip, uv)
   - Manages system-wide dependencies (nginx, certbot, firewall)
   - Creates isolated instance directories at `/var/python/openalgo-flask/openalgo<N>/`
   - Generates unique configurations per instance (ports, domains, databases)
   - Creates systemd services and Nginx reverse proxy configs
   - Handles SSL certificate generation via Let's Encrypt

3. **update_swap_4gb.sh** - Fixed swap utility
   - Creates or replaces fixed 4GB swap space to prevent OOM during broker authentication

4. **oa-configure-swap.sh** - Flexible swap utility
   - Interactive or command-line driven swap configuration (1-512 GB)
   - Validates disk space before allocation
   - Displays current swap configuration and filesystem usage
   - Includes confirmation prompts and safe reconfiguration

5. **oa-restart.sh** - Instance management (manual)
   - Discovers running instances via systemd
   - Provides interactive menu for restarting single or all instances
   - Auto-reloads Nginx after restart

5a. **setup-daily-restart.sh** - Automated restart scheduler
   - Sets up cron job for daily automatic restart at 8 AM IST
   - Creates restart script at `/usr/local/bin/openalgo-daily-restart.sh`
   - Creates log file at `/var/log/openalgo-daily-restart.log`
   - Verifies/sets system timezone to Asia/Kolkata
   - Provides easy modification commands for restart time

6. **oa-uninstaller.sh** - Cleanup utility
   - Removes instances with full cleanup (service, directories, SSL certs, nginx config)
   - Includes confirmation prompts to prevent accidental deletion

7. **oa-health-check.sh** - Monitoring utility
   - Multi-category health checks (service, ports, configuration, databases, filesystem, logs, connectivity)
   - System-wide health assessment (Nginx, firewall, swap, load)
   - Exit codes for automation (0=healthy, 1=warning, 2=critical)
   - Supports single instance, all instances, or system-only checks

8. **oa-backup.sh** - Backup & restore utility
   - Quick backups (env + databases + configs) with optional GPG encryption
   - Full backups (complete instance archive)
   - Selective restore with current data preservation
   - Automatic cleanup of old backups (configurable retention)
   - Supports single instance, all instances, or specific backup operations

9. **oa-update.sh** - Smart update utility
   - Version-aware .env merging using `ENV_CONFIG_VERSION` field
   - Selective updates (only merge .env when version changes)
   - Pre-update automatic backup
   - Dependency updates via `uv sync`
   - Dry-run mode to preview updates
   - Rollback capability to pre-update backup

10. **make-executable.sh** - Setup utility
   - Finds all `.sh` files in repository automatically
   - Makes them executable with single command
   - Reports success/failure for each script
   - Provides summary and lists all available scripts
   - Simplifies initial setup process

## Key Implementation Patterns

**Instance Isolation:**
- Each instance uses a unique port range (Flask: 5000+N, WebSocket: 8765+N, ZMQ: 5555+N)
- Separate SQLite databases per instance (openalgo{N}.db, latency{N}.db, logs{N}.db)
- Unique session/CSRF cookie names (session{N}, csrf_token{N}) to prevent cross-instance pollution
- Individual systemd services (openalgo{N}) with separate Unix sockets

**Broker Integration:**
- Validates broker names against hardcoded list (30 supported brokers)
- Special handling for XTS-based brokers that require market data API credentials
- Broker credentials injected via `.env` file during installation

**Configuration Management:**
- Uses `.env` file from cloned OpenAlgo repository as template
- Sed-based replacements to update environment variables
- Order of replacements matters (domain → ports → credentials → keys)

**Error Handling:**
- `check_status()` function aborts on any command failure
- Logging via `log_message()` function with color codes
- All logs saved to `logs/install_multi_TIMESTAMP.log`

## Key Features

**Version-Aware .env Updates (oa-update.sh):**
- Reads `ENV_CONFIG_VERSION = 'X.Y.Z'` from both old and new `.sample.env`
- If versions match: skips merge (code-only updates)
- If versions differ: intelligently merges configuration
- Preserves instance-specific settings: ports, broker, credentials, keys, cookies
- Includes custom variables not in template
- Falls back to MD5 hash comparison if version field missing

**Health Check Exit Codes (oa-health-check.sh):**
- 0: All checks passed (healthy)
- 1: One or more warnings detected
- 2: Critical issues found
- Enables automation and monitoring integration

**Backup Encryption (oa-backup.sh):**
- Uses GPG with AES256 cipher for .env files
- Falls back to plain text if GPG unavailable
- Preserves file permissions and ownership
- Creates pre-restore backup of current data

## Common Tasks

**Add support for a new broker:**
1. Add broker name to `valid_brokers` variable in multi-install.sh
2. If XTS-based, add to `xts_brokers` variable
3. Test broker credential validation

**Debug installation failures:**
1. Check latest log: `tail -f logs/install_multi_*.log`
2. Verify systemd service: `sudo systemctl status openalgo<N>`
3. Check Nginx config: `sudo nginx -t`
4. View Flask app logs: `sudo journalctl -u openalgo<N> -n 50`

**Monitor instance health:**
1. Run health check: `sudo ./oa-health-check.sh all`
2. Check specific instance: `sudo ./oa-health-check.sh openalgo1`
3. Integrate with monitoring: Use exit codes for alerting

**Backup before major changes:**
1. Quick backup: `sudo ./oa-backup.sh all quick`
2. Full backup: `sudo ./oa-backup.sh all full`
3. List backups: `sudo ./oa-backup.sh list`

**Update instances safely:**
1. Dry-run first: `sudo ./oa-update.sh dry-run`
2. Update single: `sudo ./oa-update.sh openalgo1`
3. Update all: `sudo ./oa-update.sh update-all`
4. Rollback if needed: `sudo ./oa-update.sh rollback /path/to/backup openalgo1`

**Modify port allocation strategy:**
Update the port calculation formulas in the instance loop (lines 272-274 in multi-install.sh)

## Testing Considerations

- Scripts require root access; local testing limited to syntax checking with `shellcheck`
- `.env` template comes from OpenAlgo repository; verify template variables before sed replacements
- Domain validation uses regex; test with various formats (subdomains, international domains)
- Nginx config uses variables in heredoc; ensure proper escaping of `$` characters
- `ENV_CONFIG_VERSION` field must exist in both old and new `.sample.env` for version comparison
- Health check tests systemd, ports, and files; requires running OpenAlgo instances for full validation
- Backup encryption requires GPG; falls back to plain text gracefully if unavailable
- Update script tests git commands; requires valid git repository with origin remote

## External Dependencies

- **OpenAlgo repository**: Cloned from https://github.com/marketcalls/openalgo.git
- **Python packages**: uv (installed via snap), gunicorn, eventlet, synced via `uv sync`
- **System tools**: nginx, certbot, systemd, timedatectl, sed, awk, grep, curl, ss, df, du
- **Optional tools**: gpg (for backup encryption), git (for updates)

## Implementation Notes

**Error Handling:**
- All scripts use explicit `check_status()` or error checking to fail fast
- Backups are always created before destructive operations
- Service state is preserved and restored on failures
- Colored output distinguishes status (green=success, yellow=warning, red=error)

**Configuration Management:**
- `.env` files contain sensitive credentials (API keys) - encrypted backups recommended
- Instance ports calculated as BASE + instance_number (allows easy scaling)
- Session/CSRF cookie names made unique per instance to prevent cross-instance pollution
- Timezone validated as IST (Asia/Kolkata) for Indian stock market compatibility

**Security:**
- WebSocket/ZMQ ports bound to localhost; external access only through Nginx SSL
- Firewall configured to allow SSH, HTTP, HTTPS only
- File permissions restrict instance directories to www-data user
- Keys directory (700 permissions) holds sensitive authentication data
- All scripts require sudo; no security bypass mechanisms

**Version Management:**
- OpenAlgo devs increment `ENV_CONFIG_VERSION` ONLY when `.env` structure changes
- This enables smart updates: code-only changes skip .env processing
- Version mismatch detection prevents configuration drift across instances

---
> Source: [jabez4jc/Simplifyed-Scripts](https://github.com/jabez4jc/Simplifyed-Scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
