## trailfinder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Trailfinder is a Rust application for network device discovery and configuration parsing. The application can SSH into network devices, automatically identify their brand/type, parse their configurations, and maintain a JSON-based inventory. Features a comprehensive web interface with interactive topology visualization, HTTPS support, and RESTful API. Currently implements MikroTik, Cisco, and Ubiquiti parsers with comprehensive test coverage.

## Commands

### Building

- `cargo build --quiet` - Build the project
- `cargo run --quiet` - Run the main application (device discovery)
- `cargo run --quiet -- web` - Run the web server (HTTP on localhost:8000)
- `cargo run --quiet -- web --tls-cert cert.pem --tls-key key.pem` - Run HTTPS web server
- `cargo run --quiet -- web --help` - Show web server options including TLS configuration

### Testing

- `cargo test --quiet` - Run all tests (253+ tests with comprehensive coverage)
- `cargo test --quiet test_parse_mikrotik` - Run specific test
- `cargo test --quiet mikrotik` - Run all MikroTik parser tests
- `cargo test --quiet cisco` - Run all Cisco parser tests
- `cargo test --quiet lib_tests` - Run core library tests
- `cargo test --quiet main_tests` - Run CLI integration tests
- `cargo test --quiet ssh_tests` - Run SSH client tests

### Development

- `cargo check --quiet` - Fast compile check without producing binaries
- `cargo clippy --all-targets --quiet` - Lint checker
- `cargo fmt` - Format code

### Logging

The application uses structured logging via the `tracing` crate.

Examples:

- `cargo run` - Run with default INFO level logging
- `cargo run -- --debug` - Run with detailed debugging
- `cargo run --release` - Run release build with minimal output

## Architecture

The codebase follows a standard Rust library + binary structure:

- **src/lib.rs** - Core data models and trait definitions:
  - `Device` struct - Main data structure for network devices with interface references
  - `Interface` struct - Network interface with unique ID generation (`interface_id()` method)  
  - `Route` struct - Network routes that reference interfaces by ID
  - `ConfParser` trait - Extensible interface for device configuration parsers
  - Enums for `DeviceType`, `InterfaceType`, `RouteType`, and `Owner`

- **src/main.rs** - Main application logic for device discovery and identification

- **src/config.rs** - JSON-based configuration management:
  - `AppConfig` - Application configuration with device inventory
  - `DeviceConfig` - Per-device configuration including SSH details
  - `DeviceBrand` - Enum for supported device manufacturers

- **src/ssh.rs** - SSH client functionality:
  - `SshClient` - SSH connection with multiple authentication methods
  - `DeviceIdentifier` - Automatic device brand/type detection
  - SSH config integration - reads `~/.ssh/config` for connection details

- **src/brand/mikrotik.rs** - MikroTik-specific parser implementing `ConfParser`
- **src/brand/cisco.rs** - Cisco-specific parser implementing `ConfParser`
- **src/brand/ubiquiti.rs** - Ubiquiti-specific parser implementing `ConfParser`

- **src/web/mod.rs** - Web server and API implementation:
  - `web_server_command` - Main web server function with HTTP/HTTPS support
  - REST API endpoints for devices, topology, and path finding
  - OpenAPI/Swagger documentation generation
  - TLS certificate parsing and hostname extraction
  - HTTPS support with RSA and ECDSA keys (including prime256v1/P-256)

- **src/cli.rs** - Command-line interface with comprehensive subcommands:
  - Device discovery and identification commands
  - Web server startup with TLS configuration options
  - Add/remove device commands
  - Path finding and scanning functionality

## Device Configuration

The application uses a `devices.json` file to store device inventory. Each device entry includes:

- Hostname and IP address  
- SSH connection details (username, port, key path)
- Authentication preferences (ssh-agent, identity files)
- Brand and device type (auto-detected)
- Last interrogation timestamp for caching
- TLS configuration for HTTPS web server (optional):
  - Certificate file path (`tls_cert_file`)
  - Private key file path (`tls_key_file`)
  - Hostname override (`tls_hostname`)

## SSH Authentication

The application supports multiple SSH authentication methods in priority order:

1. **SSH Config Integration** - Automatically reads `~/.ssh/config` for:
   - Username from `User` directive
   - Identity files from `IdentityFile` directive  
   - ssh-agent usage based on `IdentitiesOnly` setting

2. **Manual Configuration** - Fallback to device-specific settings:
   - ssh-agent authentication (default: enabled)
   - SSH key file authentication
   - Password authentication (via `SSH_PASSWORD` environment variable)

The application tries SSH config first, then falls back to manual config if SSH config fails.

## Web Interface

The application includes a comprehensive web interface built with Axum and modern web technologies:

### Core Features
- **Interactive Topology Visualization** - D3.js-powered network maps with device relationships
- **Device Inventory Management** - Searchable device lists with filtering by type
- **Path Finding** - Network path discovery between devices and endpoints
- **RESTful API** - Complete API with automatic OpenAPI/Swagger documentation

### HTTPS/TLS Support
- **Multiple Key Formats** - RSA (PKCS#1, PKCS#8) and ECDSA (SEC1, PKCS#8) including prime256v1/P-256
- **Automatic Hostname Extraction** - Extracts hostnames from certificate SAN or CN fields
- **Flexible Configuration** - CLI arguments, environment variables, or config file options
- **Modern TLS** - Uses rustls with AWS-LC-RS crypto backend

### Web Interface Pages
- `/` - Homepage with navigation
- `/devices` - Device inventory with type filtering and search
- `/topology` - Interactive network topology visualization
- `/pathfinder` - Network path discovery tool
- `/api-docs` - OpenAPI/Swagger documentation
- `/api-docs/openapi.json` - OpenAPI specification

### API Endpoints
- `GET /api/devices` - List all devices with optional filtering
- `GET /api/devices/{id}` - Get specific device details
- `GET /api/topology` - Get network topology data
- `POST /api/pathfind` - Find paths between network endpoints

## Key Dependencies

### Core Dependencies
- `serde` - Serialization/deserialization with derive macros
- `serde_json` - JSON configuration file support
- `chrono` - Timestamp handling for identification caching
- `uuid` - Unique device ID generation
- `tokio` - Async runtime for web server and concurrent operations

### SSH and Network
- `russh` - SSH client functionality for device connection (replaces ssh2)
- `russh-keys` - SSH key management and parsing
- `ssh-agent` - SSH agent integration
- `cidr` - CIDR network parsing and manipulation

### Web Server and API
- `axum` - Modern async web framework for HTTP server
- `axum-server` - HTTPS/TLS support for axum with rustls
- `tower-http` - HTTP middleware (static files, CORS, tracing)
- `utoipa` - OpenAPI/Swagger documentation generation
- `utoipa-swagger-ui` - Swagger UI integration
- `askama` - Template engine for HTML rendering
- `askama_web` - Axum integration for askama templates

### TLS and Cryptography
- `rustls` - Modern TLS implementation with AWS-LC-RS crypto
- `rustls-pki-types` - TLS certificate and key type definitions with PEM parsing
- `x509-parser` - X.509 certificate parsing for hostname extraction

### CLI and Configuration
- `clap` - Command-line argument parsing with derive macros
- `dirs` - Cross-platform home directory detection
- `shellexpand` - Shell-style path expansion (~/ support)

## Testing Data

The project includes sample MikroTik and Cisco system responses in `src/tests/` that are used in comprehensive unit tests:
- `src/tests/mikrotik_*.txt` - MikroTik RouterOS command outputs
- `src/tests/cisco_*.txt` - Cisco IOS command outputs

## Test Coverage

The project maintains comprehensive test coverage across all major components:
- **CLI Integration Tests** - 25+ tests covering command parsing, config validation, error handling
- **SSH Client Tests** - 20+ tests covering connection management, authentication, device identification
- **Core Library Tests** - 32+ tests covering data models, serialization, error handling
- **Brand Parser Tests** - 43+ tests covering MikroTik (23) and Cisco (20) configuration parsing
- **Web API Tests** - Tests covering REST endpoints, OpenAPI documentation, and web interface
- **Total: 253+ tests** providing robust validation of network discovery and web interface functionality

Test coverage includes:
- Edge case handling and error conditions
- Configuration parsing for multiple device types
- Network topology discovery and neighbor detection
- Interface and routing table parsing
- Device identification and classification
- Web API endpoints and response formatting
- TLS configuration and certificate parsing
- HTTP/HTTPS server functionality

## Current Development Tasks

See TODO.md for the current development plan and task status.

## Development Guidelines

- no task is complete unless you can run `just check` and there's no errors or warnings
- when editing Cargo.toml always try to use cargo commands unless it's impossible
- clean up TODO.md when commiting checked-off tasks
- update TODO.md with tasks before and after they're done
- always use a todo list in TODO.md when working on complex tasks to track progress and remain on track
- if wanting to test the app, just use `cargo run` rather than `cargo build` then running the binary
- when using javascript you MUST use DOM manipulation to update UI objects, NEVER use .innerHTML or similar
- never try to use port 3000 to try and run the server, use a high port for testing

## Testing Philosophy

- Prefer comprehensive unit tests with edge case coverage over integration tests
- Test error conditions and malformed input handling
- Use real device output samples from `src/tests/` directory
- Maintain test coverage above 70% for all core modules
- Each brand parser should have 20+ tests covering all parsing functions
- always finish the task to the best of your ability, stopping half way through unless there is a major blocker is NEVER acceptable

---
> Source: [yaleman/trailfinder](https://github.com/yaleman/trailfinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
