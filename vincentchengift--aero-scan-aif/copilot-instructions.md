## aero-scan-aif

> A distributed blockchain-backed knowledge graph for tracking MQ-9 Reaper parts through the USAF supply chain. Three independent backends share a 7-node CometBFT blockchain for consensus. Each backend projects chain events into its own ArcadeDB graph database. A Next.js dark-HUD frontend connects via Socket.IO.

# AERO-SCAN — MQ-9 Reaper Defense Supply-Chain Prototype

## What This Is
A distributed blockchain-backed knowledge graph for tracking MQ-9 Reaper parts through the USAF supply chain. Three independent backends share a 7-node CometBFT blockchain for consensus. Each backend projects chain events into its own ArcadeDB graph database. A Next.js dark-HUD frontend connects via Socket.IO.

## Architecture (DO NOT rediscover — use directly)

```
CometBFT chain (7 validators: node-1..node-7)
    ↓ blocks
backend-1 (port 8001) → arcadedb-1 (port 2480)
backend-2 (port 8002) → arcadedb-2 (port 2481)
backend-3 (port 8003) → arcadedb-3 (port 2482)
    ↓ Socket.IO
Next.js frontend (served from /out as static + SSR)
```

- **Docker compose file**: `docker-compose.multi.yml` (7-node + 3-backend)
- **Single-node dev**: `docker-compose.yml`
- **Build**: `docker compose -f docker-compose.multi.yml up -d --build`
- **Volume mounts**: `./app:/app/app` — edits to `app/*.py` are live-reloaded by uvicorn

## File Map (DO NOT scan — read these directly)

### Backend (`app/`)
| File | Purpose | Key functions |
|------|---------|---------------|
| `api.py` | FastAPI app entry, mounts routers | |
| `db.py` | ArcadeDB HTTP REST adapter | `query_sql()`, `run_sql()` |
| `admin_api.py` | Admin endpoints (seed, tamper, emit-event) | `tamper_events()` L1180, `seed_events()`, `seed_parts_via_chain()` |
| `admin.html` | Admin UI (Alpine.js) | `tamperEvents()`, `seedEvents()`, `emitLiveEvent()` |
| `socket_server.py` | Socket.IO server, integrity scan loop, AI agent chat | `_integrity_scan_loop()`, `emit_ledger_snapshot()`, `_build_kg_nodes()` |
| `agent_planner.py` | AI agent: PLAN→EXECUTE→SYNTHESIZE | `QUERY_CATALOG`, `_x_find_neighbors()`, `_x_manufacturer_parts()`, `_x_site_parts()` |
| `integrity.py` | KG↔chain integrity verification | `run_shallow_scan()`, `run_deep_verification()`, `recalculate_event_hash()` |
| `ledger_indexer.py` | Pulls blocks from CometBFT, stores BlockEvent, projects Event | `_pull_and_store_block()`, `_upsert_event()`, `_fill_gaps()` |
| `ledger_publisher.py` | Publishes events TO the chain | `make_event()`, `broadcast_tx()` |
| `event_generator.py` | Generates realistic part lifecycle events | `generate_events()`, `generate_supplier_events()` |
| `seed_data.py` | Seeds KG vertices + edges from CSV | |
| `ticket_generator.py` | Generates fake supply-chain tickets | |

### Frontend (`out/`)
- Pre-built Next.js static export, served by FastAPI
- Block store in `out/_next/static/chunks/0ks364ybl.ia5.js` — `tn=50` (max blocks cached)
- Socket events: `ledger:block`, `integrity:status`, `nodes:data`, `nodes:integrity:update`, `events:batch`

### Scripts
| File | Purpose |
|------|---------|
| `scripts/seed_parts_via_chain.py` | Seeds ~991 parts via blockchain transactions |
| `scripts/generate_usaf_supply_chain.py` | Generates CSV data files |

## KG Schema

### Vertex types
`Part`, `Manufacturer`, `Supplier`, `AircraftPartsStore`, `AirForceBase`, `AirLogisticsComplex`

### Node ID prefixes
`PRT-`, `MFG-`, `SUP-`, `APS-`, `AFB-`, `ALC-`

### Edge types
`MANUFACTURES`, `AUTHORIZED_FOR`, `SUBCONTRACTS`, `DELIVERED_BY`, `STORED_AT`, `INSTALLED_AT`, `IN_DEPOT`, `SHIPS_BETWEEN`, `SERVICES`, `CAN_MAINTAIN`

### Special tables (not vertices in KG sense)
`Event`, `Ticket`, `BlockHeader`, `BlockEvent`, `SyncState`

### BlockEvent columns
`height`, `idx`, `asset_id`, `hash`, `prev_hash`, `payload`, `status`
- NOT `event_hash`, NOT `tx_index` — these caused bugs before

### Event columns
`event_id`, `event_type`, `timestamp`, `asset_id`, `event_hash`, `prev_hash`, `height`, `chain_status`, `payload_json`, `description`, `seq_on_chain`, `integrity_status`, `from_node_id`, `to_node_id`, ...

## Environment
- DB password: `aeroscan2026`
- `CHAIN_RPC_URL`: e.g. `http://node-1:26657` (CometBFT RPC)
- `CHAIN_WS_URL`: e.g. `ws://node-1:26657/websocket`
- `OPENAI_API_KEY` / `OPENAI_MODEL`: for AI agent

## Integrity System
- **Shallow scan** runs every 5s in `_integrity_scan_loop()` (socket_server.py)
- Compares Event.payload_json hash against BlockEvent.hash
- Three states: `verified`, `syncing` (benign, ledger ahead of KG), `tampered` (alarm)
- Hash recipe: `sha256(json.dumps({"asset_id":..., "prev_hash":..., "payload":{...}}, separators=(",",":"), sort_keys=True))`
- Tamper endpoint: `POST /api/data/admin/tamper-events` — corrupts `payload_json` + `description` on random Event rows
- **No self-healing exists yet** — tampered events stay corrupted until manual fix or re-seed

## CometBFT RPC
- Each backend talks to its pinned validator: backend-1→node-1, backend-2→node-2, backend-3→node-3
- RPC endpoints: `/status`, `/block?height=H`, `/block_results?height=H`, `/validators`, `/net_info`
- **Known issue**: CometBFT config sets `laddr = "tcp://127.0.0.1:26657"` for RPC (localhost only). docker-compose.multi.yml has a `sed` entrypoint hack to patch it to `0.0.0.0:26657` — untested.

## AI Agent (agent_planner.py)
- Architecture: PLAN (LLM picks tools from QUERY_CATALOG) → EXECUTE (run SQL) → SYNTHESIZE (LLM composes answer)
- Key executors: `_x_find_neighbors()`, `_x_manufacturer_parts()`, `_x_site_parts()`, `_x_find_node()`, `_x_count_nodes()`, `_x_recent_events()`
- Synth prompt truncation: `[:8000]` chars
- Intent router: falls back to `kg_rag_answer()` only when planner returns no data
- Synth rules include: enumerate specific node_ids (never summarize away), use correct ID prefixes

## Gotchas (learned the hard way)
1. **Worktree vs main repo**: If Claude Code creates a worktree (`.claude/worktrees/`), Docker won't see those edits — it volume-mounts `./app:/app/app` from THIS directory. Always edit files here, not in a worktree.
2. **Three separate ArcadeDBs**: Each backend has its own DB. Tamper on one doesn't appear on others.
3. **Chain-based operations propagate naturally**: `seed-parts-via-chain` goes through blockchain, so all backends get the data. Direct DB writes (like tamper) only hit one.
4. **BlockEvent column names**: `hash` and `idx` — NOT `event_hash` or `tx_index`.
5. **Unicode path**: The repo is under `桌面` (Chinese for Desktop). Use PowerShell for path-dependent commands.
6. **Frontend is pre-built**: `out/` is a static Next.js export. JS is minified. Don't try to "build" it.

## Token Optimization (MANDATORY)
- **No directory scanning.** Read only files named above.
- **Code first, prose minimal.** Skip preambles and restating the question.
- **Batch reads.** If multiple files needed, read them in one round of parallel calls.
- **No echoing file contents** unless user asked for review.
- **Diffs only.** When modifying code, use Edit tool with minimal context.
- **One-pass edits.** Plan all changes before writing; avoid read-edit-read-verify cycles.
- **Skip confirmations** like "I've updated the file."
- **Error analysis:** top-level exception + immediate app frame only.
- Before modifying code: list target files + functions in ≤3 bullet points.
- **Don't re-read files** you just read in the same session.
- **Don't grep for things listed above** — column names, file paths, function names are documented.

---
> Source: [VincentChengIFT/aero-scan-aif](https://github.com/VincentChengIFT/aero-scan-aif) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
