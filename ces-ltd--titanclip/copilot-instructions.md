## titanclip-security

> Security-sensitive areas — secrets, auth, adapters, IPC


# Security

- **Never** log, echo, or paste raw secrets, API keys, JWTs, or vault payloads. Follow existing redaction and encryption patterns for credential vault and agent auth.
- Treat **LLM adapters**, **Express routes**, and **Electron IPC** as trust boundaries: minimal surface area, clear error handling, no “temporary” bypasses.
- **Context files** shipped to agents (`AGENTS.md`, `.paperclip.md`) may be scanned for injection; keep onboarding copies in `server/src/onboarding-assets/` consistent with product expectations when editing those assets intentionally.

---
> Source: [CES-Ltd/TitanClip](https://github.com/CES-Ltd/TitanClip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
