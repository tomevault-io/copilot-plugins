## as-aix

> - `src/main.rs` — CLI entry point (clap 4 derive), scan/extract subcommands, interactive mode

# AS-AIX — AI Query History Forensic Extractor

## Architecture

- `src/main.rs` — CLI entry point (clap 4 derive), scan/extract subcommands, interactive mode
- `src/output.rs` — CSV output formatting (14 columns + NaturalLanguage)
- `src/scanner.rs` — Triage directory scanning for AI artifacts
- `src/tools/mod.rs` — Shared types: AiEntry, AiArtifact, AiTool enum, activity detection, linearization
- `src/tools/claude_code.rs` — Claude Code extraction (history.jsonl + session JSONL files)
- `src/tools/chatgpt.rs` — ChatGPT extraction (SQLite databases)

## Key Decisions

- All timestamps normalized to `DateTime<Utc>`, output always in UTC
- Claude Code timestamps: Unix milliseconds (history.jsonl) or ISO 8601 (session JSONL)
- ChatGPT timestamps: Unix seconds or ISO 8601 depending on schema version
- Content field truncated to 500 chars in CSV output for readability
- Session JSONL parsing skips non-message entries (file-history-snapshot, etc.)
- SQLite databases copied to temp dir before reading (avoids lock conflicts)
- Cross-platform path handling for Windows (C:\Users\) and macOS/Linux (~/.claude/)

---
> Source: [acquiredsecurity/as-aix](https://github.com/acquiredsecurity/as-aix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
