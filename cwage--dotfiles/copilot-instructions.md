## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal dotfiles repository focused on systemd user services and X11 session configuration. The repository contains:

- **systemd user services**: Email tunnel services (socat-based) and audio system configurations (pipewire/pulseaudio)
- **X11 session configuration**: Custom xsession desktop entry
- **Minimal structure**: This is a lightweight dotfiles setup without complex build systems or package managers

## ⚠️ CRITICAL: Dotfiles Safety Policy

**These are personal dotfiles that directly affect the user's system configuration. Exercise extreme caution:**

- **NO SWEEPING CHANGES**: Never make large-scale modifications or refactoring
- **SMALL CHANGES ONLY**: Only make minimal, targeted changes explicitly requested by the user
- **ALWAYS REQUIRE TESTING**: User must test any changes before committing
- **ASK BEFORE ACTING**: Confirm the scope and impact of any proposed changes
- **PRESERVE FUNCTIONALITY**: These files control active system services - breaking them affects the user's workflow

When making changes:
1. Make only the specific change requested
2. Explain what the change does and potential impacts
3. Ask user to test the change before committing
4. Never assume broader improvements are wanted

## Architecture

### Systemd User Services
- **Email tunnels**: `socat-imap-tunnel.service` and `socat-smtp-tunnel.service` create TCP tunnels for email services
  - IMAP tunnel: binds port 1143 on 10.10.15.120 → localhost:1143
  - SMTP tunnel: binds port 1025 on 10.10.15.120 → localhost:1025
- **Audio services**: pipewire and pulseaudio configurations with proper service dependencies and security restrictions
- **Service locations**: All services are in `systemd/user/` with appropriate symlinks in target directories

### File Structure
```
systemd/user/
├── *.service                    # Service definitions
├── default.target.wants/        # Services enabled by default
├── sockets.target.wants/        # Socket-activated services
└── pipewire.service.wants/      # Pipewire dependencies
```

## Common Commands

### Systemd User Service Management
```bash
# Enable and start a service
systemctl --user enable socat-imap-tunnel.service
systemctl --user start socat-imap-tunnel.service

# Check service status
systemctl --user status socat-imap-tunnel.service

# View service logs
journalctl --user -u socat-imap-tunnel.service

# Reload systemd configuration after changes
systemctl --user daemon-reload
```

### Installation/Deployment
This repository doesn't have automated installation scripts. Services should be:
1. Copied to `~/.config/systemd/user/`
2. Enabled with `systemctl --user enable <service>`
3. Started with `systemctl --user start <service>`

## Development Notes

- Services use specific IP binding (10.10.15.120) - likely for network isolation or VPN scenarios
- Audio services include security hardening (NoNewPrivileges, MemoryDenyWriteExecute, etc.)
- No build system or package management - changes are made directly to service files
- Services are designed for user-space operation, not system-wide

## Testing Services

Check if services are running properly:
```bash
# Test email tunnels
ss -tlnp | grep :1143  # IMAP tunnel
ss -tlnp | grep :1025  # SMTP tunnel

# Test audio services
systemctl --user status pipewire
systemctl --user status wireplumber
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cwage)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cwage)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
