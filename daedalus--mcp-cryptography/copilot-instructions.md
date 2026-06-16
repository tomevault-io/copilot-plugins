## mcp-cryptography

> MCP server exposing cryptography library functionality including encryption, hashing, and key derivation.

# MCP Cryptography

MCP server exposing cryptography library functionality including encryption, hashing, and key derivation.

## When to use this skill

Use this skill when you need to:
- Encrypt/decrypt data (symmetric and asymmetric)
- Generate cryptographic keys
- Hash data
- Create/verify digital signatures
- Derive keys from passwords
- Work with X.509 certificates

## Tools

- **Key Generation**: `generate_symmetric_key`, `generate_rsa_key`, `generate_x25519_key`
- **Symmetric Encryption**: `fernet_encrypt`, `fernet_decrypt`, `aes_encrypt`, `aes_decrypt`
- **Asymmetric Encryption**: `rsa_encrypt`, `rsa_decrypt`, `rsa_sign`, `rsa_verify`
- **Hashing**: `hash_sha256`, `hash_sha512`, `hash_sha1`
- **HMAC**: `hmac_sign`, `hmac_verify`
- **Key Derivation**: `pbkdf2_derive`, `scrypt_derive`
- **X.509 Certificates**: `generate_self_signed_cert`, `parse_certificate`
- **Utilities**: `generate_random_bytes`, `generate_random_base64`

## Install

```bash
pip install mcp-cryptography
```

---
> Source: [daedalus/mcp-cryptography](https://github.com/daedalus/mcp-cryptography) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
