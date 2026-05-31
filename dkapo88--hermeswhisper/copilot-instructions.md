## 005-security-config

> Security and configuration guidance

# Security & Configuration

- Never commit secrets; prefer `*.xcconfig` and use Keychain at runtime.
- Review `Signing & Capabilities` and `*.entitlements` for least privilege; enable Hardened Runtime.
- Avoid private APIs; audit third‑party dependencies periodically.
- Keep model/API keys in the Keychain and load via dedicated services (e.g., `KeychainService`).

---
> Source: [dkapo88/hermeswhisper](https://github.com/dkapo88/hermeswhisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
