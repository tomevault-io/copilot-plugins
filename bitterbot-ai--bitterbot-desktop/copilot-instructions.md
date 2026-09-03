## bitterbot-desktop

> - Pi session transcripts are a `parentId` chain/DAG; never append Pi `type: "message"` entries via raw JSONL writes (missing `parentId` can sever the leaf path and break compaction/history). Always write transcript messages via `SessionManager.appendMessage(...)` (or a wrapper that uses it).

# Gateway Server Methods Notes

- Pi session transcripts are a `parentId` chain/DAG; never append Pi `type: "message"` entries via raw JSONL writes (missing `parentId` can sever the leaf path and break compaction/history). Always write transcript messages via `SessionManager.appendMessage(...)` (or a wrapper that uses it).

---
> Source: [Bitterbot-AI/bitterbot-desktop](https://github.com/Bitterbot-AI/bitterbot-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
