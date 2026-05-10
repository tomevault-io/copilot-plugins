## vendor-jarvisos

> Read `vendor/jarvisos/AGENTS.md` immediately — full project spec and session log.

# JarvisOS — Claude Code Context

## Role: Senior Engineer
Read `vendor/jarvisos/AGENTS.md` immediately — full project spec and session log.
Read `vendor/jarvisos/WORKFLOW.md` — how Kevin works.
Read `vendor/jarvisos/HANDOFF.md` — what was done last session and what to do next.
Check `.jarvis/tasks/` for pending `TASK_<n>.md` files before doing anything else.

## Repos (GitHub org: ocansey11)
- `ocansey11/android_frameworks_base` — branch `lineage-21.0` — this repo
- `ocansey11/vendor_jarvisos` — branch `main` — all MDs live here
- `ocansey11/cactus` — branch `main` — inference engine

---

## What This Project Is

JarvisOS is an Android OS built on LineageOS that runs LLM inference, RAG, and
tool-calling as privileged Android system services — baked into the OS, not apps.
No cloud dependencies. All inference via Cactus (on-device ARM).

---

## Repository Layout

```
lineage/                                        ← BUILD ROOT (when running locally)
├── CLAUDE.md                                   ← you are here
├── vendor/jarvisos/                            ← canonical MD location (ocansey11/vendor_jarvisos)
│   ├── AGENTS.md                               ← READ THIS FIRST
│   ├── HANDOFF.md                              ← READ THIS SECOND
│   ├── WORKFLOW.md                             ← READ THIS THIRD
│   ├── ARCHITECTURE.md                         ← full service internals (for Sam / team)
│   ├── TOOL_REGISTRY.md                        ← Phase 4 design spec
│   ├── TOOL_DISPATCHER.md                      ← ToolDispatcher integration guide
│   ├── MEMORY_SCHEMA.md                        ← ObjectBox entity design
│   ├── AGENTIC_LOOP.md                         ← Phase 5 architecture
│   └── RESEARCH_CALM_TURBOQUANT.md             ← future research
│
├── frameworks/base/core/java/android/jarvis/   ← public API layer
│   ├── IJarvisService.aidl                        ← public Binder interface
│   ├── IToolRegistry.aidl                      ← public tool registry interface
│   ├── JarvisManager.java                         ← app-facing API
│   └── JarvisException.java
│
└── frameworks/base/services/core/java/com/android/server/jarvis/
    ├── JarvisService.java                         ← system service entry point
    ├── IJarvisService.aidl                        ← server-side Binder stub
    ├── Android.bp                              ← build config (services.jarvis)
    │
    ├── core/
    │   ├── JarvisStore.java                    ← ObjectBox singleton
    │   ├── ModelRegistry.java                  ← (modelHandle, indexHandle) pairs
    │   ├── IndexQueue.java                     ← BlockingQueue for indexing tasks
    │   ├── JarvisManager.java                     ← public API manager
    │   └── JarvisException.java
    │
    ├── inference/
    │   └── CactusWrapper.java                  ← JNI bridge to libcactus.so
    │
    ├── indexing/
    │   ├── JarvisIndexWorker.java                 ← WorkManager background indexer
    │   ├── JarvisFileObserver.java             ← watches Documents/Downloads/Pictures
    │   ├── TextExtractor.java                  ← file → raw text
    │   └── ChunkingStrategy.java               ← text → chunks
    │
    ├── search/
    │   └── MetadataSearch.java                 ← Stage 1 ObjectBox keyword search
    │
    ├── model/                                  ← ObjectBox entities
    │   ├── SourceFile.java
    │   ├── DocumentChunk.java
    │   ├── Chunk.java
    │   ├── Conversation.java
    │   ├── Message.java
    │   ├── Folder.java
    │   ├── UserContext.java
    │   ├── AccessLog.java
    │   └── TaskMemory.java
    │
    └── tools/                                  ← Phase 4 Tool Registry
        ├── AppRecord.java                      ← ObjectBox: one per installed app
        ├── ToolRecord.java                     ← ObjectBox: one per tool (ToOne<AppRecord>)
        ├── ToolScannerService.java             ← scans APKs on install, embeds tools
        └── ToolDispatcher.java                 ← resolves + fires tools
```

---

## Phase Status

| Phase | Status | Notes |
|-------|--------|-------|
| 0 | ✅ Done | LineageOS setup, AIDL, manifest |
| 1 | ✅ Done | JarvisService, Binder, FileObserver, IndexQueue |
| 2 | ✅ Done | Full RAG pipeline |
| 3 | ✅ Done | ModelRegistry — "rag" + "tools" handle pairs |
| 4 | 🔄 In progress | AppRecord/ToolRecord/ToolScannerService/ToolDispatcher written. IToolRegistry.aidl + Android.bp wiring still needed |
| 5 | 📅 Planned | Agentic loop — see AGENTIC_LOOP.md |
| 6 | 📅 Planned | Memory consolidation / DreamWorker |

---

## Architecture Rules (Non-Negotiable)

1. Public APIs only in `frameworks/base/core/` — NOT in services
2. `CactusWrapper` is the ONLY entry point to Cactus native
3. No Cactus `corpus_dir` — use primitives: `indexInit`, `indexAdd`, `indexQuery`, `embed`
4. "rag" and "tools" model entries use SEPARATE index dirs — NEVER mix indexes
5. No Kotlin in system_server
6. Small models (~270M) get zero-shot — no system prompts
7. Audio models produce transcripts only — not stored embeddings
8. `ToolDefinition.java` is a tombstone — replaced by AppRecord + ToolRecord

## Key Facts

- `ToolDispatcher.resolveAndDispatch()` returns null if no tool matches — falls through to RAG
- Tool broadcast timeout: 10 seconds
- ObjectBox query classes `AppRecord_` and `ToolRecord_` must still be wired into Android.bp
- lineage-22.2 re-sync pending (do at uni on fast connection) — current code is lineage-21.0, pure Java
- x86_64 emulators cannot load Cactus — ARM hardware only for native testing
- Latest commit: fde5f8eb — Phase 4 tools written, pushed to origin/lineage-21.0

---
> Source: [ocansey11/vendor_jarvisos](https://github.com/ocansey11/vendor_jarvisos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
