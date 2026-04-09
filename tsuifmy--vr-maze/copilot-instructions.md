## vr-maze

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a VR Web development project that serves WebXR-enabled 3D content to VR headsets like Quest 2/3, Pico, HTC Vive, etc. The project consists of Python-based HTTP/HTTPS servers and a Three.js-based VR web application.

## Development Commands

### Starting the Development Server

**HTTP Server (Development):**
```bash
./start-server.sh          # Starts HTTP server on port 8080 with firewall rules
# or
python3 server.py          # Direct server start
```

**HTTPS Server (VR Production):**
```bash
./start-https-server.sh    # Starts HTTPS server on port 8443 with SSL certificates
# or  
python3 https_server.py    # Direct HTTPS server start
```

**NixOS Development Environment:**
```bash
nix-shell                  # Loads development shell with Python3 and cryptography
```

### SSL Certificate Management

```bash
python3 generate_cert.py   # Generate SSL certificates (preferred method)
# or
./generate-cert.sh         # Alternative bash-based certificate generation
```

### Port Configuration

- HTTP: Port 8080 (development)
- HTTPS: Port 8443 (VR production, required for WebXR API)
- Change ports by modifying `PORT` variable in respective Python files

## Architecture

### Server Architecture

**HTTP Server (`server.py`):**
- Basic HTTP server for development and testing
- CORS headers for cross-origin requests
- WebXR permission headers
- Auto-redirects root to `index.html`
- Custom request logging with client IP

**HTTPS Server (`https_server.py`):**
- Production-ready HTTPS server required for WebXR APIs
- Self-signed SSL certificate support
- Enhanced security headers (HSTS, CSP)
- Same CORS and WebXR headers as HTTP server
- Binds to all interfaces (0.0.0.0) for LAN access

### Frontend Architecture (`index.html`)

**VRWebApp Class Structure:**
- `scene`: Three.js scene container
- `camera`: PerspectiveCamera with VR positioning
- `renderer`: WebGL renderer with XR support enabled
- `controller1/2`: VR controller handling
- `objects[]`: Array of animated 3D objects

**3D Scene Components:**
- Floating animated cubes with rotation and vertical movement
- Orbiting transparent spheres
- Particle system with colored points
- Ground plane with shadows
- Ambient + directional lighting with shadow mapping

**VR Integration:**
- WebXR API integration for immersive VR sessions
- VR controller ray casting and interaction
- Fallback mouse/keyboard controls for desktop
- Cross-device VR headset support

### Certificate Generation

**Python Method (`generate_cert.py`):**
- Uses `cryptography` library for certificate generation
- Automatic local IP detection and SAN (Subject Alternative Names) inclusion
- 2048-bit RSA key generation
- Valid for 365 days

**Bash Method (`generate-cert.sh`):**
- OpenSSL-based certificate generation
- Network interface detection (wireless/wired priority)
- Creates temporary config file for certificate extensions

## NixOS Integration

The project includes NixOS-specific configurations:

- `shell.nix`: Development environment with Python3 and cryptography
- Firewall configuration examples in `nixos-firewall-config.md`
- Startup scripts handle temporary `iptables` rules for port access

## WebXR Requirements

- HTTPS is mandatory for WebXR API access
- Self-signed certificates require manual browser trust
- VR devices must be on same LAN as server
- Feature-Policy and Permissions-Policy headers properly configured

## Network Configuration

The servers automatically detect local IP addresses for LAN access. VR headsets can access the application using the detected IP address. Firewall ports are temporarily opened by startup scripts and cleaned up on exit.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tsuifmy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tsuifmy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
