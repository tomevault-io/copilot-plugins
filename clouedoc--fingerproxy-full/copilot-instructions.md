## fingerproxy-full

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go-based HTTP/2 reverse proxy that embeds TLS fingerprinting information in headers before forwarding requests to an origin server. It's built on top of the `github.com/wi1dcard/fingerproxy` library and adds custom header injectors for detailed TLS analysis.

## Development Commands

### Building the Project
```bash
go build .
```

### Running in Development Mode
```bash
go run . -listen-addr :8443 -forward-url https://httpbin.org
```

For auto-restart during development (requires nodemon):
```bash
nodemon --signal SIGTERM --exec "go run . -listen-addr :8443 -forward-url https://httpbin.org" --ext go
```

### Testing the Server
```bash
curl "https://localhost:8443/headers" --insecure
```

### Dependency Management
```bash
go mod tidy    # Update dependencies and clean up go.mod
go mod verify  # Verify dependencies
```

## Architecture

### Core Structure
- `main.go` - Single-file application that customizes the fingerproxy library
- Uses the fingerproxy framework to handle HTTP/2 reverse proxying
- Extends default functionality with custom TLS fingerprinting headers

### Key Components

#### Header Injectors
The application defines custom header injectors that extract and format TLS information:

- **JA4 Debug Info** (`X-JA4-Fingerprint-Debug`) - Detailed JA4 fingerprint analysis
- **TLS Cipher Suites** - Both numeric codes and human-readable names
- **TLS Extensions** - List of TLS extension IDs
- **Client Hello Debug** - Raw TLS ClientHello information
- **Signature Algorithms** - Both raw hex codes and BoringSSL-compatible names
- **HTTP/2 Headers Order** - Comma-separated list of header names in order
- **Header Values JSON** - JSON array of all header values

#### TLS Analysis Functions
- `JA4FingerprintDebug()` - Parses and formats JA4 fingerprint data
- `tlsSignatureAlgs()` / `tlsSignatureAlgsNames()` - Extract signature algorithms
- `tlsCipherSuitesStrings()` / `tlsCipherSuitesCodes()` - Extract cipher suite information
- `tlsExtensions()` - Extract TLS extension information
- `tlsClientHello()` - Parse complete ClientHello record
- `headersOrder()` / `headersValuesJson()` - Extract HTTP/2 header information

### Dependencies
- `github.com/wi1dcard/fingerproxy` - Core reverse proxy functionality
- `github.com/dreadl0ck/tlsx` - TLS parsing and analysis
- `github.com/pkg/errors` - Error handling utilities

## Testing

No test files are currently present in the codebase. To add tests:
```bash
go test ./...  # Run all tests
go test -v .   # Run tests with verbose output
```

## Deployment

The application can be deployed as a SystemD service. Example configuration is provided in the README.md file for production deployment with TLS certificates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/clouedoc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
