## mnemify

> > Read this before touching any code. This file is a **map to the code**, not

# AGENTS.md — Mnemify Orientation

> Read this before touching any code. This file is a **map to the code**, not
> a substitute for it. It stays intentionally short; the code and its module
> docstrings are the source of truth. If anything here conflicts with the
> code, the code wins.

---

## Where things live

| Area | Path | Notes |
|---|---|---|
| Harvester (Tier 0) | `backend/src/harvester/` | One package per source (`notion/`, `confluence/`, `jira/`, `obsidian/`, `slack/`, `github/`, `_google/`). Plugin contract is `SourcePlugin` in `harvester/__init__.py`: `health_check()`, `list_documents(since)`, `fetch_document(ref)`, optional `mark_harvested()`; register with `register_plugin(name, factory)`. Only the packages in `ENABLED_SOURCES` are imported — see `src/sources.py`. |
| Terrain compiler (Tier 1) | `backend/src/terrain/` | File map below. |
| FastAPI + SSE (Tier 2) | `backend/src/api/` | `__init__.py` builds the app and mounts the built frontend (`paths.web_dist_dir()`); with no build it logs a WARNING and serves a "frontend not built — run `sh setup.sh`" page at `/` rather than booting API-only. Harvest/compile progress streams over SSE from an in-process event bus (`event_bus.py`, `compile_bus.py`) with a replay ring buffer for reconnecting tabs. Schedules (`/api/schedules`) run on an in-process APScheduler — single uvicorn worker only. |
| On-device embeddings | `backend/src/api/routes_embeddings.py`, `terrain/utils/local_embedder.py` | Claude engines with no `OPENAI_API_KEY`: `start_compile` refuses with `code="openai_key_missing"` + `local_embeddings_eligible`; the frontend `LocalEmbeddingsDialog` (mounted in `DashboardLayout`, reached through `useStartCompile`) explains the trade-off (runs locally, English-only, coarser regions), links to Settings, and on consent calls `POST /embeddings/local/prepare`, which downloads bge-small into `<home>/.mnemify/models` and saves it as the `embedding_model` default. The model never downloads silently — `LocalEmbeddingClient` loads with `local_files_only`. The compile dialog (`AiModePicker`) has an **Embeddings: OpenAI / On this computer** switch (OpenAI disabled without a key); a per-run pick that differs from the saved default is persisted by `start_compile` so Ask and schedules stay in the same vector space. With a key set but the on-device default never downloaded, the refusal is `local_model_missing` + `openai_key_set` and the dialog offers OpenAI instead of the download. |
| Chat (`/api/ask`) | `backend/src/api/routes_ask.py`, `ask_retrieval.py`, `ask_chunks.py`, `ask_expansion.py`, `ask_providers.py`, `ask_agent.py` | Flow: `understand_query()` → embed → chunk search over `terrain.db` vectors → graph walk/rerank → bundle (≤15 items) → expand to raw chunk text → SSE `retrieval_debug → citations → delta* → citations_used → done`. The chat-LLM key arrives per request in `Authorization: Bearer` and is never persisted; query embeddings use the model stamped on `graph_node_vectors` (OpenAI via the server's `OPENAI_API_KEY`, or the on-device model) so vectors match the compile. Frontend side: `frontend/web/src/ask/`. |
| CLI | `backend/src/cli.py` | `mnemify harvest / terrain build / up / stop / status / inspect / normalize / purge / debug / reset / migrate-home / login`. Run via `uv run mnemify …` from `backend/`. `up` is single-instance (an already-running server means "print the URL, open the browser, exit 0") and falls forward up to ten ports if `--port` is taken; `stop` POSTs the shutdown route using `<home>/server.port`; `migrate-home` moves a legacy `backend/` layout into the platform home. |
| On-disk paths | `backend/src/paths.py` | **Every** path the app persists to resolves here — `home()`, `data_dir()`, `yaml_file()`, `env_file()`, `logs_dir()`, `server_pid_file()`, `server_port_file()`, `web_dist_dir()`. Home precedence: `MNEMIFY_HOME` → legacy (`backend/` already holds `.mnemify/`, `mnemify.yaml` or `.env`) → platform app-data dir. `MNEMIFY_ENV_FILE` / `MNEMIFY_YAML_FILE` still win for those two files. |
| Shipped connectors | `backend/src/sources.py` | `ENABLED_SOURCES = ("notion", "confluence", "obsidian")` — what the release exposes. The other five plugins stay in the tree, tested, just never registered; `MNEMIFY_SOURCES=notion,jira` turns any subset back on for one process. Gates the registry's imports and `GET /api/connections`, not the plugin code. |
| Process lifecycle | `backend/src/api/lifecycle.py`, `idle.py`, `routes_system.py` | The app quits itself. `lifecycle` holds the `uvicorn.Server` handle (`request_shutdown()`); `idle` is the ASGI activity stamp + watchdog (health and SSE reconnects don't count, the browser's 30 s heartbeat does, a running compile/harvest/ask or *any* enabled schedule blocks it); `routes_system` is `POST /api/system/heartbeat`, `/shutdown` and `/open-home` (reveals `paths.home()` in the OS file manager; Settings → Data shows the path). **Every** non-GET `/api` request must carry `X-Mnemify-Client` (`ClientGuardMiddleware` in `api/__init__.py`, next to the Host guard): a foreign page can make the browser POST to localhost, but cannot add a custom header without a CORS preflight the app never grants. `apiFetch` sends it on every call; `mnemify stop` and the smoke tests send `cli`. `POST /api/reset` additionally needs `{"confirm": "RESET"}` in the body (the UI makes the user type it). |
| Secrets | `backend/src/api/routes_secrets.py`, `credential_store.py` | `GET/PUT/DELETE /api/secrets[/{name}]` over an allowlist (`SECRET_ALLOWLIST`); values go in and never come back out — `GET` returns `set` plus a masked hint. `credential_store` owns the atomic `0600` write to `<home>/.env` and the `os.environ` refresh that makes a saved key live without a restart. |
| Install + launch | `setup.sh` / `setup.bat`, `mnemify.sh` / `mnemify.bat`, `scripts/` | Repo-root entry points; `scripts/` holds the implementations (`setup.ps1`, `mnemify.ps1`, `make-launcher.sh`/`.ps1`). `setup.sh` is idempotent — re-running it *is* the update path. Icons in `assets/icons/`. |
| React app | `frontend/web/src/` | `brainMap/` is the self-contained R3F 3D module (`BrainMap.tsx`, `scene/`, `chrome/`, `store.ts`); `app/` is the dashboard shell (`routes.tsx`, `pages/`, `components/`, `api/`, `sse/`, `data/`). Stack: React 18 + Vite 5 + TS strict, Tailwind with CSS-var theme, React Router v6, TanStack Query v5. Vite proxies `/api/*` to `:8783`. |
| Tests | `backend/tests/` (`uv run pytest -q`), `frontend/web/` (`npm test`, `npx tsc -b`) | Live-network harvester tests skip without credentials. |

**On-disk state — everything under `MNEMIFY_HOME`, outside the repo:**

Home is `MNEMIFY_HOME` if set, else `backend/` when that directory already
holds state (the legacy dev layout — this is why the team's checkouts keep
working), else the platform app-data dir: `~/Library/Application Support/Mnemify`
· `%LOCALAPPDATA%\Mnemify` · `$XDG_DATA_HOME/mnemify`. `src/paths.py` is the
only place that decides.

| Under home | What |
|---|---|
| `.mnemify/` | All harvested + compiled data (table below) |
| `mnemify.yaml` | Source config, schedules, `compile:`, `data_retention:`, `server:` — written by the wizards and Settings through `api/yaml_writer.py` (ruamel round-trip, comments preserved) |
| `.env` | Secrets, mode `0600`, written atomically by `api/credential_store.py` |
| `logs/` | Launcher + server logs |
| `server.pid`, `server.port` | The running instance; written by `mnemify up`, removed on exit, read by the launcher's already-running check and by `mnemify stop` |

| Inside `.mnemify/` | What |
|---|---|
| `raw/<source>/<shard>/<id>.<ext>` | Raw fetched bytes, byte-for-byte |
| `normalized/<source>/<shard>/<id>.md` | Clean-markdown sidecar per document (body only; metadata lives in the manifest) |
| `harvest-manifest.db` | SQLite: one row per document + `harvest_runs` |
| `harvest-log.jsonl` | Append-only audit log of harvest events |
| `terrain.json` | v2 BrainMap: region → tag tree, graph nodes/edges, entities, attention signals |
| `mocknotes.json` | Note registry (one per source doc) for citations and panels |
| `render-data.json` | v3 hex render-data the 3D map reads — additive-only, no graph/embedding fields |
| `terrain.db` | SQLite cache: features, embeddings, names, positions, compiled notes, chunk vectors, compile runs |

Both SQLite stores carry a `PRAGMA user_version` (`harvester/manifest.py`,
`terrain/utils/store.py`): set on create and on upgrade, and checked on open —
a file written by a *newer* Mnemify fails loudly instead of corrupting
quietly, which is what protects someone juggling two checkouts or rolling
back a download.

---

## What This Product Is

A **Knowledge Map that is becoming a grounded chatbot.** Users connect
Notion / Confluence / Obsidian — the three connectors this build ships; Jira,
Slack, GitHub, Gmail and Calendar are complete and in the tree but only
reachable via `MNEMIFY_SOURCES` (see `src/sources.py`). The pipeline turns harvested documents
into (a) a 3D hex terrain — a spatial browse surface — and (b) a layered
knowledge graph the chatbot retrieves over. Both are derived from the same
compiled brain map; neither is primary over the other going forward.

**The three pillars:**
1. **Legibility** — every connection can explain *why* it exists (edge
   provenance: `extracted` vs `inferred` vs `ambiguous`).
2. **Multi-path triangulation** — the same answer reachable through entities,
   themes, and notes, so the system can corroborate itself.
3. **Local-first** — the user's brain and their API key never leave their
   machine.

**Key invariants that still hold:**
- **Same document, multiple region appearances.** A cross-cutting document
  gets its primary region plus secondary `region_assignments` above a
  similarity threshold — implemented in `clusterer.assign_multi_region()`.
- **Emergent structure, not fixed taxonomy.** Regions/tags/depth emerge from
  HDBSCAN on real embeddings, not hardcoded keyword lists.
- **Uniform node schema** for the region tree (`id, name, position, height,
  chunk_ids, children`) — leaves have `chunk_ids` and empty `children`.
- **LLM names/synthesizes what emerged; it does not determine structure.**
  Clustering is HDBSCAN; naming and compiled-note synthesis are LLM calls
  layered on top, cached by content-hash fingerprint.
- **`render-data.json` is additive-only.** Graph/entity/embedding work
  (Stage 4+) lives in `terrain.json`, never leaks into the v3 hex bake.
- **No module binds a data path at import.** Call `paths.data_dir()` /
  `paths.env_file()` / … at use time. A module-level `DATA_DIR = ...` freezes
  whatever `MNEMIFY_HOME` happened to be when the import ran, which breaks
  test isolation and every path override.
- **Every secret the app needs has a UI write path** — `/api/secrets`
  (Settings → AI & Models) or a connection wizard. Never add a key whose only
  home is a hand-edited `.env`, and never document editing that file as the
  way to set one.

---

## Pipeline Overview

```
Harvest → Normalize → Chunk (per-source) → Extract → Embed → Cluster
  → Merge/Region-refine → Name/Compile-notes → Promote entities
  → Build graph (Leiden + confidence) → Context-embed (GraphSAGE-lite)
  → Layout → Emit terrain.json → Bake v3 render-data.json
```

Three tiers:

| Tier | Package | LLM? |
|---|---|---|
| 0 — Harvester | `backend/src/harvester/` | No — deterministic |
| 1 — Compiler | `backend/src/terrain/` | Yes (or `--ai-mode local`) |
| 2 — Surface | `backend/src/api/` + `frontend/web/` | No — BYOK |

---

## `backend/src/terrain/` — current file map

```
terrain/
├── pipelines/compiler.py     TerrainCompiler — drives the whole compile:
│                              reader → chunk → extract → embed → cluster →
│                              region_merger → namer → _promote_entities →
│                              _build_graph_view (Leiden) → graph_embed →
│                              layout → emit → validate → bake_v3
├── preprocessing/
│   ├── reader.py              Loads active manifest rows + normalized markdown
│   ├── extractor.py           Local-mode deterministic feature extraction
│   └── chunkers/               Per-source ChunkerRegistry dispatch (Stage 1):
│       markdown_default.py, notion.py, obsidian.py, confluence.py, registry.py
├── agents/
│   ├── openai_clients.py      OpenAI-backed extractor, namer, compiled-note
│                              synthesis, entity blurbs, query understanding
│   ├── claude_cli.py           `ai_mode="claude"` transport — subscription
│                              auth via the local `claude` CLI, not metered API
│   └── claude_clients.py      Shims the OpenAI prompt/parse contract onto claude_cli
├── utils/
│   ├── models.py               Single source of truth: TerrainChunk, ChunkFeatures,
│                              ClusterTreeNode, GraphNode/GraphEdge/GraphView, Entity,
│                              BrainMap, AttentionSignal, etc.
│   ├── embedder.py             EmbeddingClient (real OpenAI `embed`/`embed_batch`)
│                              + LocalHashEmbeddingClient (explicit local/offline mode)
│   ├── local_embedder.py       LocalEmbeddingClient — on-device bge-small via fastembed
│                              (no OPENAI_API_KEY; consent-gated download behind
│                              /api/embeddings/local; picked by embedding_model name)
│   ├── clusterer.py             TerrainClusterer — HDBSCAN + recursive _build_node,
│                              nearest-centroid noise absorption, assign_multi_region,
│                              container-backbone clustering (cluster_with_containers)
│   ├── containers.py           Source-native folder path extraction (Obsidian, etc.)
│   ├── region_merger.py        Post-cluster region merge/refine pass
│   ├── namer.py                 ClusterNamer (local heuristic fallback) —
│                              OpenAIClusterNamer (agents/) does the real LLM naming
│   ├── canonicalize.py         Entity-label fuzzy canonicalization (Stage 4.5)
│   ├── attention.py            AttentionSignal / recency / action-item detection
│   ├── graph_embed.py          Stage 4.6 — mean-of-neighbors context_embedding
│   ├── layout.py                Deterministic circular packing, nested_positions()
│   ├── emitter.py               Atomic terrain.json + mocknotes.json writer
│   └── store.py                 SQLite cache (.mnemify/terrain.db): features,
│                              embeddings, names, positions, compiled notes
├── _bake_v3.py                  v2→v3 hex bake core: force-directed region layout,
│                              warped-Voronoi leaf assignment, tag summits, diffusion
└── render_v3.py                  bake_v3() entry point + call sequence
```

**Do not rely on any older description of this tree** (flat `chunker.py` /
`compiler.py` at package root, no `agents/` split) — that shape predates the
v0.5 rewrite (see `git log` around `cbe1b0f`, `V0.5`, `V0.6 Improvements`).

---

## Current status (v0.5/v0.6) — what's real vs pending

The embedder, clusterer, and namer described in older internal notes as
"broken" (fake hash embeddings, HDBSCAN overridden by keyword matching, no
LLM naming) are **fixed** — `EmbeddingClient.embed()` calls real
`text-embedding-3-small`/`-large`, `clusterer.py` has no keyword override
branches, and `OpenAIClusterNamer` does real LLM naming + compiled-note
synthesis. Don't re-diagnose these; if something in this area looks wrong,
check `git log -p` on the file first — it's probably a known, already-fixed
edge case, or a documented pending gap (below).

**Pending:**
- Reference-affinity signals beyond wikilinks/mentions (internal URLs, Notion
  page mentions, Confluence `ac:link`, Jira issue links) — only Obsidian
  really benefits from the sparse-note clustering boost today.
- Entity canonicalization threshold (0.85) undermerges risk for `person`
  labels — needs tightening or a stricter person-specific rule.
- Per-workspace entity-type taxonomy is hardcoded
  (`person/product/project/customer/concept`) — Stage 5.5 generalizes this.
- Confidence-score calibration for `inferred` edges — currently borrows the
  Jaccard weight axis as a trust axis; not validated against a labeled set.
- Citation → hex-map click-through: works for **tag** citations only; entity
  and region citations are non-clickable; note citations need frontend
  `openDoc` wiring. Recommended approach: resolve entity → home tag.
- Compile-time emission counters (`{stage, regions, tags, entities, edges}`)
  for observability — not yet reported over SSE.

---

## What Not To Do

- **Do not touch `_bake_v3.py` / `render_v3.py`** without reading the
  module and constant comments first — the geometry has documented
  failure modes (e.g. small regions annihilated by warp amplitude) that look
  like bugs but are load-bearing constants.
- **Do not add config files or settings objects** for the clustering/bake
  constants (`MAX_LEAF_CHUNKS`, `HEXES_PER_TAG`, `warp_amp`, etc.). Named
  module constants only, until real-corpus tuning data exists.
- **Do not implement compatibility bridges** to a pre-emergent-tree model. If
  HDBSCAN cluster assignments are missing, raise loudly rather than fall back.
- **Do not let graph/entity/embedding fields leak into `render-data.json`.**
  Stage 4+ output stays in `terrain.json`; `_bake_v3.py` explicitly strips it.
- **Do not revive removed frontend components** (e.g. a standalone
  `ArcsToggle.tsx`, a `Legend.tsx`, `ActivityFeed.tsx` — all removed in the V2
  BrainMap redesign).
- **Do not wholesale-adopt GraphRAG/LightRAG/graphify.** The strategy is
  to cherry-pick one pattern
  at a time into the existing hex-terrain pipeline, never swap the pipeline.

---

## Is Claude Code actually reading this file?

Only if a `CLAUDE.md` at the repo root points here. Claude Code auto-loads
`CLAUDE.md` at session start; it does **not** auto-load `AGENTS.md` on its
own. There is no `CLAUDE.md` in the repo today — add a one-line one that
says "Read AGENTS.md" if you want it picked up automatically.

---

*Last updated: 2026-09-18. The former `docs/` folder was removed; this file
and the per-package READMEs are the only prose docs.*

---
> Source: [mnemify-ai/mnemify](https://github.com/mnemify-ai/mnemify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
