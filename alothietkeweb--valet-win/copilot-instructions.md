## valet-win

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Valet-Win is a Laravel Valet clone for Windows built in Go. It's a CLI application that manages local development environments by orchestrating Caddy web server and PHP-CGI processes to serve local sites.

## Common Development Commands

```bash
# Build the application
go mod tidy           # Download and organize dependencies
go build -o valet.exe # Build the executable

# Test the application (manual testing required - no test framework)
./valet.exe install   # Install and configure services
./valet.exe start     # Start services
./valet.exe park .    # Park current directory
./valet.exe list      # Verify parked sites
./valet.exe stop      # Stop services
```

## Core Architecture

The application follows a manager-based architecture with clear separation of concerns:

- **CLI Layer** (`cmd/`): Cobra command handlers that orchestrate operations
- **Service Managers** (`internal/`): Independent managers for each service
  - `caddy/`: Web server configuration and process management
  - `php/`: PHP-CGI process lifecycle management  
  - `sites/`: Park/link operations and site discovery
  - `certs/`: Self-signed SSL certificate generation
  - `config/`: YAML-based configuration management

## Configuration System

The application uses a single YAML configuration file at `%USERPROFILE%\.valet-win\config.yaml` that drives all operations:

- **Service Paths**: PHP and Caddy executable locations
- **Site Management**: Parked directories and linked sites
- **Network**: Domain (.test) and port (80/443) configuration
- **Generated Files**: Caddyfile, PHP config, certificates, and logs

## Key Implementation Details

1. **Process Management**: Direct OS process control using Go's `os/exec` for Caddy and PHP-CGI
2. **Template-Based Config**: Caddy configuration generated from Go templates in `internal/caddy/`
3. **Certificate Authority**: Self-signed CA and per-site certificates managed in `internal/certs/`
4. **Site Discovery**: Recursive directory scanning for park operations in `internal/sites/`

## External Dependencies

The application requires these external tools at runtime:
- **PHP 8.4** with CGI support (`php-cgi.exe`)
- **Caddy** Windows binary (`caddy_windows_amd64.exe`)

## Development Notes

- No testing framework is currently implemented
- No linting or code quality tools are configured
- Configuration validation happens at runtime during command execution
- All service management is Windows-specific using Windows process APIs

---
> Source: [alothietkeweb/valet-win](https://github.com/alothietkeweb/valet-win) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
