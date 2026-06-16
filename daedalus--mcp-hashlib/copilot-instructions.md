## mcp-hashlib

> MCP server exposing hashlib functionality.

# MCP Hashlib

MCP server exposing hashlib functionality.

## When to use this skill

Use this skill when you need to:
- Hash data with various algorithms
- Derive keys from passwords
- Work with cryptographic hashes

## Tools

- `hash_md5`, `hash_sha1`, `hash_sha256`, `hash_sha512` - Standard hashes
- `hash_sha224`, `hash_sha384` - SHA-2 variants
- `hash_sha3_224`, `hash_sha3_256`, `hash_sha3_384`, `hash_sha3_512` - SHA-3
- `hash_blake2b`, `hash_blake2s` - BLAKE2
- `hash_shake_128`, `hash_shake_256` - SHAKE XOF
- `hash_pbkdf2_hmac`, `hash_scrypt` - Key derivation
- `hash_file_digest` - Hash files
- `hash_new` - Create any algorithm
- `hash_info`, `hash_properties` - Get algorithm info

## Install

```bash
pip install mcp-hashlib
```

---
> Source: [daedalus/mcp-hashlib](https://github.com/daedalus/mcp-hashlib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
