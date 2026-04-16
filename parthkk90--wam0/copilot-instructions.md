## wam0

> - Model: Phi-4-Mini 3.8B only. Never call external APIs (no GPT-4, no Claude API)

# PS-01 — The Loan Officer Who Never Forgets

## Non-negotiable constraints
- Model: Phi-4-Mini 3.8B only. Never call external APIs (no GPT-4, no Claude API)
- Data: All data stays on bank server. Mem0 self-hosted only
- Language: Hindi + English via AI4Bharat IndicASR v2
- PII: spaCy NER tokenisation runs BEFORE every mem0.add() call
- Hardware: CPU-only, 16GB RAM, 500GB SSD

## WAL rule
Always write to wal.jsonl BEFORE calling mem0.add(). Never skip this step.

## Stack
- Transcription: AI4Bharat IndicASR v2 (Docker)
- Memory: Mem0 open-source + ChromaDB
- SLM: Phi-4-Mini via Ollama
- API: FastAPI /session/start and /session/end
```

**Resource:** https://docs.anthropic.com/en/docs/claude-code/memory

---

### 2. MCP Servers — Give Claude Code tools to use
These let Claude Code directly operate your infrastructure instead of just writing code about it.

| MCP Server | What it does for PS-01 | How to add |
|---|---|---|
| `@modelcontextprotocol/server-filesystem` | Read/write `wal.jsonl`, configs, transcripts | `npm i -g @modelcontextprotocol/server-filesystem` |
| `@modelcontextprotocol/server-sqlite` | Query Mem0's SQLite graph store directly | `npm i -g @modelcontextprotocol/server-sqlite` |
| Docker MCP | Spin up/tear down per-session containers | https://github.com/docker/mcp-servers |
| Ollama (custom) | Call Phi-4-Mini inference directly | Point to `http://localhost:11434` |

Add them to `.claude/mcp_settings.json` in your project root.

**Resource:** https://docs.anthropic.com/en/docs/claude-code/mcp




---

### 3. Sub-agents — Parallel workers for your 5-day build
Claude Code can spawn sub-agents to work on multiple components simultaneously. For PS-01, define these in your `CLAUDE.md`:

- **WAL agent** — owns `wal.py`, writes and replays `wal.jsonl`, handles crash recovery
- **Tokeniser agent** — owns `tokeniser.py`, spaCy NER rules for PAN/Aadhaar/mobile
- **Compactor agent** — owns the Phi-4-Mini prompt for post-session summary rewriting
- **Test agent** — runs `pytest` after each component is built, reports failures back

**Resource:** https://docs.anthropic.com/en/docs/claude-code/sub-agents

---

### 4. Slash Commands — Reusable workflows
Create these as markdown files in `.claude/commands/`:
```
.claude/
  commands/
    session-start.md   → boot Mem0, run mem0.search(), inject context into system prompt
    session-end.md     → WAL write → mem0.add() → compact → verify
    demo-rajesh.md     → run all 4 Rajesh sessions in sequence for judge demo
    wal-replay.md      → on startup, find unsynced WAL entries and replay to Mem0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Parthkk90) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
