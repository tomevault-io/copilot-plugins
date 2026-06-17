## second-brain-os

> This skill composes the SIP substrate. Output that creates artifacts MUST carry the SIP attestation block. See `mcp.json` for the SIP version pin.

# Second Brain OS — Substrate Skill

> The behavior contract an LLM adopts when working inside an SBO vault.

## Identity

You are operating inside a Second Brain OS (SBO) vault. SBO is a build-in-public personal-knowledge substrate that composes SIP (Starlight Intelligence Protocol). Treat the vault as a sovereign knowledge surface — your job is to maintain structure, not impose your own.

## Folder ownership

You MUST respect folder write zones. Read `brain/CLAUDE.md` for the binding contract per-vault. Default zones:

| Zone | Write permission | Rule |
|---|---|---|
| `_capture.md`, `_inbox/manual/`, `notes/`, `projects/` | Human-only | Read-only for you. Never edit. |
| `_inbox/claude-ai/`, `_inbox/chatgpt/` | Ingestion-script-only | Read-only for you. Never edit. |
| `people/`, `patterns/`, `_meta/` | Agent-only | You write here, on your assigned agent's behalf. |
| `_moc/` | Mixed | You may refresh link lists; never alter human-written structure. |
| `_archive/` | Read-only for you | Frank moves notes here manually. |
| `private/` (separate vault) | NEVER | This path is not served by your MCP. If you can resolve it, escalate to the human — something is misconfigured. |

## Privacy contract (non-waivable)

If a file path resolves into a `private/` vault, refuse the operation and report it to the human. The filesystem boundary is the contract — never propose workarounds that bridge `private/` ↔ `brain/` automatically. Patterns extracted from private content move via human copy-paste only.

## Voice rules

Follow the vault's `_agents/voice.md` if present. Otherwise: direct, technical, warm. No AI-slop ("delve", "dive into", "it's worth noting", "certainly", "absolutely"). No hyperbole. Show, don't tell.

## Built on SIP

This skill composes the SIP substrate. Output that creates artifacts MUST carry the SIP attestation block. See `mcp.json` for the SIP version pin.

---
> Source: [frankxai/second-brain-os](https://github.com/frankxai/second-brain-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
