## vendor-jarvisos

> > Persistent memory and architectural spec for JarvisOS.

# AGENTS.md — JarvisOS Project Spec
> Persistent memory and architectural spec for JarvisOS.
> Read at the start of every session. Update after every sprint.
> Last updated: April 2026

---

## Project Vision

JarvisOS is a privacy-first Android OS built on LineageOS that runs LLM inference,
RAG, and tool-calling as privileged Android system services — baked into the OS,
not running as apps. No cloud dependencies. All inference on-device via Cactus.

---

## Team

- **Kevin** — Lead, Android system layer, RAG architecture, project vision
- **Sam** — NLP engineer, Cactus integration
- **Desmond** — In-house app development (integrates with Tool Registry)
- **Chindu** — Contributor, PR workflow
- **Gerald** — Security and configuration

---

## Repository Structure

| Repo | GitHub | Path (local) | Branch |
|------|--------|--------------|--------|
| Android framework fork | ocansey11/android_frameworks_base | `frameworks/base/` | `lineage-21.0` |
| JarvisOS vendor overlay | ocansey11/vendor_jarvisos | `vendor/jarvisos/` | `main` |
| Cactus inference engine | ocansey11/cactus | `vendor/cactus/` | `main` |

**Local manifest:** `.repo/local_manifests/jarvos.xml`
- Removes LineageOS default `frameworks/base`, replaces with `ocansey11/android_frameworks_base`
- Adds `ocansey11/vendor_jarvisos` → `vendor/jarvisos`
- Adds `ocansey11/cactus` → `vendor/cactus` (`sync-s=false`)

**Build environment:**
- WSL2 Ubuntu 24.04 on Windows
- lineage-22.2 re-sync pending — must do at uni on fast connection
- Current base: lineage-21.0. All JarvisOS code is pure Java — no device-specific deps

---

## Architecture Overview

```
User / App
    |
    | AIDL (Binder IPC)
    v
JarvisService.java  (com.android.server.jarvis)
    |
    +-- ToolDispatcher          tool path (runs first on every query)
    |       |
    |       +-- ToolScannerService   discovers tools from installed APKs
    |       +-- AppRecord / ToolRecord   ObjectBox entities
    |       +-- CactusWrapper.embed + indexQuery (tools index)
    |       +-- sendBroadcast → app BroadcastReceiver → ResultReceiver
    |
    +-- JarvisFileObserver      watches Documents/ Downloads/ Pictures/
    |       |
    |       v
    |   IndexQueue              BlockingQueue (cap 500)
    |       |
    |       v
    |   JarvisIndexWorker          WorkManager (15min, charging only)
    |       |
    |       +-- TextExtractor        file → raw text
    |       +-- ChunkingStrategy     text → chunks
    |       +-- CactusWrapper.embed  chunk → float[]
    |       +-- ObjectBox            persist SourceFile, DocumentChunk
    |
    +-- processQuery()
             |
             v
        MetadataSearch (Stage 1 — ObjectBox keyword/score)
             |
             v
        CactusWrapper.embed + indexQuery (Stage 2 — semantic)
             |
             v
        CactusWrapper.complete → response string
```

---

## Tech Stack

- **Base OS:** LineageOS 21.0 (Android 14) — pending upgrade to 22.2
- **Inference engine:** Cactus (forked) — `vendor/cactus/`
- **Vector DB:** ObjectBox 4.0.3 with HNSW
- **Embedding / chat model:** Qwen / nomic-embed-text
- **Tool selection model:** FunctionGemma (270M, zero-shot only)
- **Build system:** Soong (Android.bp)
- **Languages:** Java (services), C++ (JNI/Cactus)

---

## File Map

Server service code in `frameworks/base/services/core/java/com/android/server/jarvis/`:
Public API layer in `frameworks/base/core/java/android/jarvis/`:

```
jarvis/                              ← renamed from rag/ this session
├── JarvisService.java              ✅ System service entry point
├── IJarvisService.aidl             ✅ Binder interface (server-side copy)
├── Android.bp                   ✅ Build config — library: services.jarvis
│
├── core/
│   ├── JarvisStore.java         ✅ ObjectBox singleton
│   ├── ModelRegistry.java       ✅ Named (modelHandle, indexHandle) pairs
│   ├── IndexQueue.java          ✅ Singleton BlockingQueue, cap 500
│   ├── JarvisManager.java          ✅ Public API manager
│   └── JarvisException.java        ✅
│
├── inference/
│   └── CactusWrapper.java       ✅ JNI bridge — only entry point to Cactus
│
├── indexing/
│   ├── JarvisIndexWorker.java      ✅ WorkManager background indexer
│   ├── JarvisFileObserver.java  ✅ Watches Documents/Downloads/Pictures
│   ├── TextExtractor.java       ✅ File → raw text (.txt .md .csv .pdf .docx)
│   └── ChunkingStrategy.java    ✅ Sentence-boundary splitting + overlap
│
├── search/
│   └── MetadataSearch.java      ✅ Stage 1 — ObjectBox keyword scoring (6 passes)
│
├── model/                       ✅ ObjectBox entities
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
└── tools/                       ✅ Phase 4 — Tool Registry
    ├── AppRecord.java           ✅ ObjectBox: one per installed app
    ├── ToolRecord.java          ✅ ObjectBox: one per tool, ToOne<AppRecord>
    ├── ToolScannerService.java  ✅ Scans APKs on install, embeds tools
    └── ToolDispatcher.java      ✅ Resolves + fires tools via broadcast

android/jarvis/                      ← public API layer ✅
├── IJarvisService.aidl             ✅ package android.jarvis
├── IToolRegistry.aidl           ✅ package android.jarvis
├── JarvisManager.java              ✅ package android.jarvis
└── JarvisException.java            ✅ package android.jarvis
```

---

## Development Phases

### ✅ Phase 0 — Foundation
LineageOS setup, AIDL interfaces, manifest entries, vendor overlay initialized.

### ✅ Phase 1 — RAG Service Architecture
JarvisService registered in SystemServer. Binder IPC via AIDL. JarvisFileObserver →
IndexQueue → JarvisIndexWorker pipeline. TextExtractor, MetadataSearch, CactusWrapper.
ObjectBox entities defined (8 entities).

### ✅ Phase 2 — Core RAG Pipeline
Full pipeline: hash check → TextExtractor → ChunkingStrategy → embed → indexAdd →
ObjectBox persist. MetadataSearch all 6 passes wired. JarvisManager + IJarvisService
package fixed to `android.app.rag`. isIndexed() across full Binder stack.

### ✅ Phase 3 — Model Registry
ModelRegistry singleton — named map of `{ name → (modelHandle, indexHandle, dim, indexDir) }`.
Two entries registered at boot: "rag" (documents) and "tools" (tool embeddings).
Same model file, separate index directories — indexes never mixed.
JarvisIndexWorker updated to use ModelRegistry.getReady("rag") per task.

### ✅ Phase 4 — Tool Registry
**Goal:** Android-native tool discovery and dispatch. Apps expose tools via manifest
`<receiver>` + `<meta-data>`. JarvisOS scans on install, embeds descriptions, routes
queries to the right app via broadcast.

**Done:**
- `AppRecord.java` — ObjectBox entity, one per app. `ToMany<ToolRecord>`.
- `ToolRecord.java` — ObjectBox entity, one per tool. `ToOne<AppRecord>`. `rawDefinition`
  field (toolName + description + params) is what gets embedded.
- `ToolScannerService.java` — scans all installed packages on boot + listens for
  `ACTION_PACKAGE_ADDED/REMOVED/REPLACED`. Upserts AppRecord + ToolRecord. Embeds
  `rawDefinition` into "tools" Cactus index.
- `ToolDispatcher.java` — semantic search (embed query → HNSW → top-5 ToolRecords) →
  metadata fallback → builds OpenAI-compatible toolsJson → CactusWrapper.complete()
  selects tool → broadcast to app receiver → ResultReceiver + CountDownLatch (10s timeout).
- `JarvisService.java` — ToolDispatcher instantiated. processQuery() runs tool path first,
  falls through to RAG if resolveAndDispatch() returns null.
- `ToolDefinition.java` — tombstoned.

**Phase 4 complete.** ObjectBox _ stubs written (AppRecord_, ToolRecord_,
SourceFile_, Folder_, AccessLog_, TaskMemory_) + MyObjectBox stub. Committed
as `31f59143`. getCursorFactory() stubs throw — replace with processor-generated
versions when objectbox-generator deps land in prebuilts (see Android.bp TODO).

### ✅ Phase 5 — Agentic Loop
JarvisExecutor state machine. LangGraph-validated design, Claude Code leak patterns.

**Done:**
- `agent/JarvisExecutor.java` — loop runner: Plan→[Retrieve?]→[Tool?→Plan]→Respond
- `agent/RouterNode.java` — deterministic routing via Gemma 4 `<|tool_call|>` token check
- `agent/PlanNode.java` — first-turn planning, uses "primary" or "rag" ModelRegistry entry
- `agent/RetrieveNode.java` — MetadataSearch Stage 1 + HNSW Stage 2, appends to accumulatedContext
- `agent/ToolNode.java` — parses Gemma 4 `<|tool_call|>`..`<|end_tool_call|>` JSON, calls dispatchByName()
- `agent/RespondNode.java` — final CactusWrapper.complete() with accumulated context
- `model/AgentSession.java` + `AgentSession_.java` — ObjectBox entity + stub; persisted after every node
- `model/AgentTurn.java` + `AgentTurn_.java` — ObjectBox entity + stub; full turn history for Phase 6
- `ToolDispatcher.dispatchByName()` — named dispatch bypassing semantic search (for ToolNode)
- `JarvisService.processQuery()` → routes through JarvisExecutor
- `MyObjectBox` updated to register AgentSession + AgentTurn

**Max turns:** 5 hard ceiling. KV cache growth and mobile thermal throttling make this non-negotiable.
**Gemma 4 prerequisite (Sam):** pull upstream llama.cpp into Cactus, confirm `<|tool_call|>` output works.

### ✅ Phase 6 — Memory Consolidation + Multimodal + Sub-agents

**Done:**
- `agent/DreamWorker.java` — nightly WorkManager (24h, charging only). Reads DONE+unconsolidated
  AgentSessions, extracts user facts via model, merges into `UserContext.facts` (cap 50).
  Inspired by KAIROS autoDream (Claude Code internal architecture).
- `agent/SubAgentExecutor.java` — child loop for sub-tasks. maxTurns=2, inherits parent
  accumulatedContext, linked via parent session tag.
- `IJarvisService.aidl` — `processQueryWithImage()` + `processQueryWithAudio()` added
- `JarvisService` implements both (text fallback until Sam's Cactus Gemma 4 pull)
- `JarvisManager` — `queryWithImage()` + `queryWithAudio()` public wrappers for apps
- `AgentSession` — `consolidated` flag added (DreamWorker gate)
- `UserContext` — `facts` (JSON array, cap 50) + `consolidatedAt` fields added

**Blocked on Sam:** multimodal methods fall back to text until Gemma 4 is in Cactus.

---

## Key Principles

1. `CactusWrapper` is the only entry point to Cactus — never call native from elsewhere
2. No Cactus `corpus_dir` — use primitives: `indexInit`, `indexAdd`, `indexQuery`, `embed`
3. "rag" and "tools" indexes are NEVER mixed — separate dirs, separate handle pairs
4. No Kotlin in system_server
5. Small models (~270M) = zero-shot only — no system prompt injection
6. Audio models produce transcripts only — not stored embeddings
7. ObjectBox owns metadata. Cactus owns vectors.
8. ToolDispatcher.resolveAndDispatch() returns null on no match — caller falls through to RAG
9. Public APIs only in `frameworks/base/core/` — not in services

---

## Known Gaps

| Item | Notes |
|------|-------|
| MetadataSearch semantic fallback | Zero ObjectBox candidates → Cactus never called. Fix: full vector scan fallback when Stage 1 confidence below threshold. Phase 4 item. |
| AppRecord_ / ToolRecord_ not in Android.bp | ObjectBox annotation processor generates these. Build will fail without them. Next task. |
| Curated tools not loaded | vendor/jarvisos/tools/*.json not yet read by ToolScannerService. |
| No re-embed on Cactus unavailable at install | Tools stored without embedding if Cactus not ready. No retry on next boot yet. |
| GraphRAG not implemented | Entity extraction, relationship mapping, graph traversal — Phase 5/6 item. |
| lineage-22.2 re-sync pending | Target device (Nothing Phone 2 / Pong) requires lineage-22.2+. Do at uni. |

---

## Session Log

| Date | What happened |
|------|--------------|
| Early 2026 | Phase 0 complete, AIDL + manifest work done |
| Hackathon | Won 2nd place Google DeepMind x Cactus — tool calling optimization, semantic chunking |
| Feb 2026 | dev.talk speaker slot confirmed |
| Mar 2026 session 1 | Reconnected, filesystem access established, AGENTS.md created, Phase 1 scoped |
| Mar 2026 session 2 | Phase 1 complete — JarvisService, FileObserver, IndexQueue, JarvisIndexWorker, TextExtractor, MetadataSearch, CactusWrapper, ObjectBox entities |
| Mar 2026 session 3 | Cactus fork explored — JarvisOS JNI bindings added, Android.bp written, jarvisos.mk created |
| Mar 2026 session 4 | Git emergency — cherry-picked commits onto correct branch, resolved conflicts, pushed |
| Mar 2026 session 5 | Phase 2 complete — full RAG pipeline, JarvisManager, IJarvisService fixed |
| Mar 2026 session 6 | Presentation work + protocol design. Tool Registry ObjectBox schema designed. |
| Mar 2026 session 7 | Phase 3 complete — ModelRegistry.java, JarvisIndexWorker updated, JarvisService wired. lineage-22.2 re-sync decision made. |
| Apr 2026 session 8 | Phase 4 in progress — AppRecord, ToolRecord, ToolScannerService rewritten, ToolDispatcher written, JarvisService updated. All MDs migrated to vendor/jarvisos. CLAUDE.md created for Claude Code remote. AGENTIC_LOOP.md written. Dispatch + Claude Code remote workflow established. |
| Apr 2026 session 9 | Package rename: com.android.server.rag → com.android.server.jarvis. All 27 server-side files written to new jarvis/ folder. HANDOFF + AGENTS updated. Layer 2 (android/rag/ → android/jarvis/) and git rm of old folder left for Claude Code. Claude Code installed in WSL. |
| Apr 2026 session 10 | Removed old rag/ folders (server/rag, android/rag). Layer 2 public API now clean at android/jarvis/. CLAUDE.md, AGENTS.md, HANDOFF.md, TASK_1.md updated to reference jarvis package only. |
| Apr 2026 session 11 | Cactus JNI bindings fixed (10 methods still referenced com.android.server.rag — UnsatisfiedLinkError blocker). Phase 5 complete: JarvisExecutor, 5 nodes, AgentSession/AgentTurn entities, ToolDispatcher.dispatchByName(). Pushed to both repos. |
| Apr 2026 session 11 (cont.) | Phase 6 complete: DreamWorker, SubAgentExecutor, multimodal AIDL stubs (processQueryWithImage/Audio), JarvisManager wrappers. UserContext.facts + consolidated flag added. |
| Apr 2026 session 12 | ModelRegistry.registerChatModel() + "primary" entry (Gemma 4). SystemToolExecutor with 16 in-process system tools. ToolScannerService.loadCuratedTools() reads vendor/jarvisos/tools/*.json. 14 curated JSON tool files. jarvisos.mk PRODUCT_COPY_FILES. Android.bp agent/*.java build fix. requires_confirmation gate in ToolDispatcher. 3 chaining demo scenarios in AGENTIC_LOOP.md. |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ocansey11) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
