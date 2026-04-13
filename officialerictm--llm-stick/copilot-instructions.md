## llm-stick

> Project: Air-gapped “LLM Stick” for a nearly-blind, non-technical user.


Project: Air-gapped “LLM Stick” for a nearly-blind, non-technical user.
Mandates: 6-digit PIN gates everything; default = large-text UI (no auto-speech); optional Voice Mode (off by default).
Security slider: Standard (default) → Hardened → Paranoid; +2 reserved positions (disabled for now).
Air-gap policy: In Standard/Hardened the app runs even if adapters are active but MUST block all outbound networking internally; in Paranoid, adapters must be off or the app refuses to proceed.
Host filespace (read-only): Windows C:\OCRC_READONLY\ ; macOS ~/OCRC_READONLY/ ; Linux ~/OCRC_READONLY/
After any setting change: run enforcement self-check and print a 1–2 line audit (proceed or revert).
Deliverables must be path-based and runnable offline by any seasoned dev. No cloud calls, no telemetry.
At the beginning of every query, read docs/STATUS_BOARD.md and the linked docs first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/officialerictm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
