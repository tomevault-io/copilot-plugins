## haiku-bfs-encryption

> Adding transparent AES-256-XTS encryption to Haiku's BFS filesystem with hardware-backed key management using YubiKey.

# Haiku BFS Encryption Project

## Overview

Adding transparent AES-256-XTS encryption to Haiku's BFS filesystem with hardware-backed key management using YubiKey.

## Architecture

See `docs/ARCHITECTURE.md` for full technical design.

**Key points:**
- Two-partition layout: /Haiku (plain) + /Data (encrypted)
- AES-256-XTS with AES-NI hardware acceleration
- Two-factor auth: YubiKey HMAC-SHA256 + passphrase
- Argon2id key derivation
- USB removal triggers secure shutdown
- Boot partition integrity verification

## Project Structure

```
haiku-bfs-encryption/
├── CLAUDE.md              # This file
├── .gitea/workflows/      # CI build & test workflow
│   └── build-test.yml
├── docs/
│   ├── ARCHITECTURE.md    # Full technical design
│   ├── GIT_WORKFLOW.md    # Git branching & release process
│   └── ROADMAP.md         # Implementation roadmap
├── userspace/
│   ├── crypto/            # AES-XTS, Argon2 wrappers
│   ├── yubikey/           # YubiKey HID communication
│   ├── mount_tool/        # Unlock and mount utility
│   └── watchdog/          # USB removal monitor
├── kernel/
│   └── bfs_crypt/         # BFS encryption hooks (later phase)
└── tests/
    └── crypto_tests/      # Test vectors and verification
```

## Git Workflow

See `docs/GIT_WORKFLOW.md` for full details.

- **`dev`** branch: active development
- **`main`** branch: stable releases only
- **Tags**: semantic versioning `v{MAJOR}.{MINOR}.{PATCH}`
- **CI**: Gitea Actions builds and tests every push (`.gitea/workflows/build-test.yml`)

## Current Phase

**Phase 1: Userspace Crypto Library**

Focus areas:
1. AES-256-XTS wrapper using OpenSSL/LibreSSL EVP API
2. Verify AES-NI is being used (benchmark should show 4+ GB/s)
3. Argon2id key derivation
4. NIST test vector validation

## Build Environment

- **Target machine**: `taurus.microgeni.synology.me` (Haiku x86_64, also ARM64 in future)
- **Dev machine**: macOS with **nix-darwin** (flakes enabled)
- **Dev shell**: `nix develop` provides OpenSSL, libargon2, pkg-config
- **Crypto**: OpenSSL/LibreSSL (available in Haiku)
- **Build**: Standard Makefile, `CC ?= cc` (works with both gcc on Haiku and clang via nix on macOS)
- **CI**: Gitea Actions runner on macOS, SSHs to Taurus for Haiku builds/tests
- **Workflow**: Prototype locally on macOS via nix, final build and test on Taurus (real Haiku hardware with AES-NI)

## Key Design Decisions

1. **XTS mode** (not CBC, GCM): Standard for disk encryption, handles partial blocks
2. **YubiKey over USB keyfile**: Secret never leaves hardware, can't be cloned
3. **Argon2id**: Memory-hard, resistant to GPU attacks
4. **Filesystem-level encryption**: More contained than block device layer

## Dependencies

```
OpenSSL/LibreSSL  - AES-256-XTS, HMAC-SHA256
libargon2         - Key derivation (may need to port/bundle)
Haiku USB Kit     - YubiKey HID communication
```

## Testing Strategy

1. **Unit tests**: Crypto primitives against known test vectors
2. **Integration**: File-backed container before touching real BFS
3. **Hardware**: Test with actual YubiKey before kernel work
4. **Security**: Verify key scrubbing, memory locking

## Useful Commands

```bash
# ── Local development (macOS with nix) ──
nix develop              # Enter dev shell
nix develop -c make      # Build
nix develop -c make test # Run tests
nix develop -c make clean

# ── On Taurus (Haiku) ──
ssh user@taurus.microgeni.synology.me
cd /Data/Code/Projects/haiku-bfs-encryption
make                     # Build (gcc, native AES-NI)
make test                # Run tests
make benchmark           # Verify AES-NI performance (target: 4+ GB/s)
openssl speed -evp aes-256-xts  # Reference benchmark
```

## Code Style

- Follow Haiku coding guidelines for kernel code
- Use explicit_bzero() for sensitive data
- Check all return values
- Lock sensitive memory with mlock()

## References

- Haiku BFS source: https://git.haiku-os.org/haiku/tree/src/add-ons/kernel/file_systems/bfs
- Architecture doc: docs/ARCHITECTURE.md

---
> Source: [microgeni/haiku-bfs-encryption](https://github.com/microgeni/haiku-bfs-encryption) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
