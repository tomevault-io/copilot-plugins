## fssh

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

fssh is a macOS SSH private key manager that provides Touch ID or OTP (One-Time Password) authentication for SSH connections. It acts as an SSH agent that securely stores and manages encrypted private keys, eliminating the need to enter passphrases repeatedly while maintaining security.

**Core Purpose**: Replace plaintext SSH keys with encrypted storage, unlock via biometric (Touch ID) or TOTP, and provide an interactive shell for managing SSH hosts.

## Build and Development Commands

```bash
# Build the binary
go build ./cmd/fssh

# Install to /usr/local/bin
go build -o /usr/local/bin/fssh ./cmd/fssh

# Run tests (if any)
go test ./...

# Initialize master key (Touch ID mode - default)
./fssh init

# Initialize with OTP mode (for devices without Touch ID)
./fssh init --mode otp --seed-unlock-ttl 3600

# Import SSH private key
./fssh import -alias mykey -file ~/.ssh/id_rsa --ask-passphrase

# List imported keys
./fssh list

# Start SSH agent (secure mode, Touch ID/OTP per signature)
./fssh agent --unlock-ttl-seconds 600

# Start SSH agent (convenience mode, decrypt all on startup)
./fssh agent --require-touch-id-per-sign=false

# Export private key
./fssh export -alias mykey -out /path/to/key.pem --ask-passphrase

# Interactive shell for SSH connections
./fssh shell
# or simply
./fssh

# Remove a key
./fssh remove -alias mykey

# Rotate master key and re-encrypt all keys
./fssh rekey

# Check system status
./fssh status

# Generate SSH config entries
./fssh config-gen

# Align sshd config for RSA-SHA2 support
./fssh sshd-align
```

## Architecture Overview

### Authentication Layers

The project supports two authentication modes via a unified `AuthProvider` interface:

1. **Touch ID Mode** (default for macOS with Touch ID):
   - Master key stored in macOS Keychain
   - Biometric unlock via `internal/macos/touchid_darwin.go`
   - Hardware-backed security

2. **OTP Mode** (for devices without Touch ID):
   - Master key derived from password-encrypted OTP seed
   - TOTP verification (RFC 6238) via authenticator apps
   - Seed cached with TTL, supports recovery codes
   - Configuration stored in `~/.fssh/otp/config.enc`

Mode selection is stored in `~/.fssh/auth_mode.json` and determined by `internal/auth/auth.go:GetAuthProvider()`.

### Encryption Architecture

**Master Key Flow**:
```
User Auth (Touch ID/OTP) → Master Key → HKDF per-file key → AES-256-GCM → Encrypted Private Key
```

- **Master Key**: 32-byte key from Keychain (Touch ID) or password-derived (OTP)
- **Per-file Keys**: Derived using HKDF with unique salt per private key
- **Encryption**: AES-256-GCM with unique nonce, fingerprint as AAD
- **Storage**: JSON files in `~/.fssh/keys/*.enc`

See `internal/crypt/crypt.go` for crypto primitives and `internal/store/store.go` for key storage.

### SSH Agent Implementation

**Two Operating Modes**:

1. **Secure Mode** (`require_touch_id_per_sign: true`):
   - Implemented in `internal/agent/secure_agent.go`
   - Decrypts keys on-demand per SSH signature
   - Optional TTL cache for master key unlock
   - Every signature triggers `AuthProvider.UnlockMasterKey()`

2. **Convenience Mode** (`require_touch_id_per_sign: false`):
   - Uses standard `golang.org/x/crypto/ssh/agent.Keyring`
   - Decrypts all keys once at agent startup
   - Keys kept in memory until agent stops

The agent implements the SSH agent protocol via `golang.org/x/crypto/ssh/agent.Agent` interface, supporting:
- RSA-SHA2-256/512 signatures (via `SignWithFlags`)
- Public key listing
- Agent extensions

### Interactive Shell

`cmd/fssh/shell.go` provides an interactive prompt powered by `github.com/peterh/liner`:
- Parses `~/.ssh/config` via `internal/sshconfig/sshconfig.go`
- Tab completion for host names, IPs, and numeric IDs
- Commands: `list`, `search <term>`, `connect <host>`, `help`, `exit`
- Non-command input defaults to SSH connection

### Configuration Management

**User Configuration** (`~/.fssh/config.json`):
```json
{
  "socket": "~/.fssh/agent.sock",
  "require_touch_id_per_sign": true,
  "unlock_ttl_seconds": 600,
  "log_level": "info",
  "log_format": "plain"
}
```

Loaded by `internal/config/config.go`, used as defaults for CLI flags.

**Auto-start**: `contrib/com.fssh.agent.plist` for macOS LaunchAgent.

## Key Code Locations

### Authentication
- `internal/auth/auth.go` - `AuthProvider` interface and mode selection
- `internal/auth/touchid.go` - Touch ID implementation
- `internal/auth/otp.go` - OTP implementation
- `internal/otp/totp.go` - TOTP generation/verification
- `internal/otp/config.go` - OTP seed encryption/storage
- `internal/otp/recovery.go` - Recovery code generation/validation

### Cryptography
- `internal/crypt/crypt.go` - HKDF, AES-GCM encryption/decryption
- `internal/store/store.go` - Private key record storage and retrieval
- `internal/keychain/keychain.go` - macOS Keychain integration

### SSH Agent
- `internal/agent/server.go` - Agent server startup and mode dispatch
- `internal/agent/secure_agent.go` - On-demand decryption agent

### CLI Commands
- `cmd/fssh/main.go` - Command dispatch and flag parsing
- `cmd/fssh/shell.go` - Interactive shell
- `cmd/fssh/otp_init.go` - OTP initialization logic
- `cmd/fssh/config_gen.go` - SSH config generator
- `cmd/fssh/align.go` - sshd configuration alignment

### Utilities
- `internal/sshconfig/sshconfig.go` - `~/.ssh/config` parser
- `internal/log/log.go` - Structured logging
- `internal/macos/touchid_darwin.go` - macOS biometry CGo bindings

## Important Implementation Details

### Private Key Format
- All keys converted to **PKCS#8 DER** internally
- Supports RSA, ECDSA, Ed25519
- Import: Handles encrypted/unencrypted PEM via `ssh.ParseRawPrivateKey()`
- Export: PKCS#8 PEM with optional AES-256 encryption

### File Storage Structure
```
~/.fssh/
├── agent.sock                # SSH agent socket
├── config.json               # User configuration
├── auth_mode.json            # Authentication mode (touchid/otp)
├── keys/                     # Encrypted private keys
│   ├── mykey.enc             # Per-key encrypted file
│   └── server.enc
└── otp/                      # OTP mode only
    ├── config.enc            # Encrypted OTP seed + config
    └── recovery_codes.enc    # Encrypted recovery codes
```

### OTP Mode Security
- Password must be ≥12 characters
- TOTP uses standard algorithms (SHA1/SHA256/SHA512), 6 or 8 digits
- OTP seed encrypted with password-derived key (scrypt or similar)
- Seed cached in memory with configurable TTL
- Master key cached per signature with separate TTL
- 10 single-use recovery codes for password reset

### TTL Behavior
- **OTP Mode**: Two-tier caching
  - `seed-unlock-ttl`: How long OTP seed stays in memory (password entry frequency)
  - `unlock-ttl-seconds`: How long master key stays cached (TOTP entry frequency)
- **Touch ID Mode**: Single-tier caching
  - `unlock-ttl-seconds`: How long Touch ID unlock is cached

### RSA-SHA2 Support
The agent advertises RSA-SHA2-256/512 support via:
- `SignWithFlags()` implementation in `secure_agent.go`
- `Extension("ext-info-c")` returning signature flags bitmask
- Uses `ssh.AlgorithmSigner` interface for algorithm selection

## Common Development Patterns

### Adding a New Command
1. Add case to `cmd/fssh/main.go:main()` switch
2. Implement `cmdYourCommand()` function in `main.go` or separate file
3. Parse flags with `flag.NewFlagSet()`
4. Call appropriate internal packages

### Modifying Authentication Flow
- Both Touch ID and OTP implement `AuthProvider` interface
- `UnlockMasterKey()` is the primary entry point
- Cache management happens inside provider implementations
- Test mode switching by modifying `~/.fssh/auth_mode.json`

### Working with Encrypted Keys
```go
// Load and decrypt
mk, _ := keychain.LoadMasterKey()              // or provider.UnlockMasterKey()
rec, _ := store.LoadDecryptedRecord(alias, mk)
// rec.PKCS8DER contains raw private key

// Parse and use
priv, _ := x509.ParsePKCS8PrivateKey(rec.PKCS8DER)
signer, _ := ssh.NewSignerFromKey(priv)
```

### Logging
Use structured logging via `internal/log/log.go`:
```go
log.Info("message", map[string]interface{}{"key": "value"})
log.Debug("detail", fields)
log.Warn("warning", fields)
log.Error("error", fields)
```

## Testing the Agent

```bash
# Terminal 1: Start agent
./fssh agent --unlock-ttl-seconds 600

# Terminal 2: Set socket and test
export SSH_AUTH_SOCK=~/.fssh/agent.sock
ssh-add -l  # List keys from agent
ssh user@host  # Test actual connection
```

## Security Considerations

- Never log master keys, seeds, passwords, or recovery codes
- Always use `0600` permissions for encrypted files
- Touch ID prompts use macOS LAContext with biometry policy
- OTP mode should recommend FileVault full-disk encryption
- Convenience mode keeps keys in memory - document security tradeoff
- Recovery codes are single-use and should be marked as used after consumption

## Documentation

- `README.md` - English user guide
- `README_CN.md` - Chinese user guide
- `docs/OTP-QUICKSTART.md` - OTP mode quick start
- `docs/otp-authentication.md` - OTP design document
- `docs/otp-sdd.md` - OTP software design document
- `docs/otp-implementation.md` - OTP implementation details

---
> Source: [Mister-leo/fssh](https://github.com/Mister-leo/fssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
