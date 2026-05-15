## istefox-dt-mcp

> > Istruzioni di progetto. Sovrascrivono il default e si aggiungono alle regole globali in `~/.claude/CLAUDE.md`.

# CLAUDE.md — DEVONthink MCP Connector

> Istruzioni di progetto. Sovrascrivono il default e si aggiungono alle regole globali in `~/.claude/CLAUDE.md`.

---

## 1. Identità del progetto

- **Nome interno**: `istefox-dt-mcp`
- **Scopo**: MCP server "best-in-class" per DEVONthink 4 — dual-transport, multi-bridge, RAG-augmented
- **Owner**: Stefano Ferri (progetto personale, namespace `istefox` — non lavorativo)
- **Documento di architettura di riferimento**: `/Users/stefanoferri/Downloads/ARCH-BRIEF-DT-MCP.md` (versione 0.1, 29/04/2026)
- **Stato attuale**: brief approvato, in attesa di review architetturale Cowork → ADR finale → implementazione

Quando in dubbio sull'architettura, **riferirsi al brief**, non improvvisare.

---

## 2. Vincoli OBBLIGATORI di progetto (non negoziabili)

### 2.1 Vincoli legali / IP

- **Implementazione clean-room**: NON guardare, NON copiare, NON ispirarsi al codice di [`dvcrn/mcp-server-devonthink`](https://github.com/dvcrn/mcp-server-devonthink) (GPL-3.0). Concept ok, codice mai.
- **Privacy by design**: nessun dato dell'utente esce dalla macchina per default. Embedding, audit log, cache: tutto locale.

### 2.2 Vincoli protocollo MCP

- **stdio mode**: MAI scrivere su `stdout`. Corrompe il flusso JSON-RPC. Logging solo su `stderr` o file.
- **Schema first**: ogni tool deve avere schema JSON Schema rigoroso via Pydantic v2.
- **Tool description ≤ 2KB**, **server instructions ≤ 2KB** (limite Claude Code).
- **Resource URI** stabili e deterministici (`dt://...`).
- **Resource bounded**: ogni resource ≤ 25K token (limite Claude Code default). Mai dump di interi database.
- **Tool naming**: nomi compatti, contesto nelle description.

### 2.3 Vincoli DEVONthink

- **DT4-first**: target primario `Application("DEVONthink")` (DT4). DT3 è best-effort opzionale con feature gating.
- **DT Pro deve essere in esecuzione** per il bridge JXA (eccetto bridge DT Server).
- **JXA è single-threaded**: necessario worker pool con semaphore + backpressure (default 4-8 worker concorrenti, timeout 5s).
- **Sync sidecar**: smart rule DT → sidecar + reconciliation notturno hash-based. NO filesystem watching su `~/Library/Application Support/DEVONthink/` (inaffidabile).
- **DB vector** fuori dalla cartella DEVONthink (evita lock contention).

### 2.4 Vincoli safety / operativi

- **Dry-run mandatory** per write/destructive ops: `dry_run: bool = True` come default.
- **Output strutturato uniforme**: ogni tool ritorna `{success, data, warnings, audit_id}`.
- **Audit log append-only** SQLite per ogni write op (audit_id, timestamp, principal, tool, input, output hash, before-state per undo, durata). Retention default 90 giorni.
- **Idempotency keys** per tutte le write ops.
- **Errori strutturati** con codici tassonomici (`DT_NOT_RUNNING`, `JXA_TIMEOUT`, ecc.) + `recovery_hint` testuale.

---

## 3. Stack tecnologico vincolato

| Componente | Tecnologia | Note |
|---|---|---|
| Linguaggio | **Python 3.12** | usare `python3`, mai `python` |
| Framework MCP | **FastMCP** | SDK ufficiale Python |
| Validazione | **Pydantic v2** | schema-first per ogni tool |
| Bridge JXA | `osascript` via `asyncio.subprocess` | niente Node solo per JXA |
| Vector DB | **ChromaDB** embedded (v1) | Qdrant valutato in v2 |
| Embedding | `sentence-transformers` con `bge-m3` o `multilingual-e5-large` | multilingue IT/EN/FR/DE |
| Cache | SQLite WAL | niente Redis (single-user) |
| HTTP transport | `uvicorn` + streamable-http MCP SDK | per v2 |
| Auth (HTTP) | `authlib` (OAuth 2.1 + PKCE) | scope granulari |
| Packaging | **`uv` + `pyproject.toml`** | mainstream 2026 |
| Distribuzione | `pipx` + `.mcpb` desktop extension | dual-target CI |
| Test | `pytest` + `pytest-asyncio` | |
| Logging | `structlog` (JSON su stderr) | |
| Tracing | `opentelemetry-sdk` | |
| Linter | `ruff` | |
| Formatter | `black` (line length 88) | |

**Non sostituire** componenti di questo stack senza discussione esplicita. Il brief motiva ogni scelta.

---

## 4. Layout repo (monorepo)

```
Devonthink_MCP/
├── apps/
│   ├── server/          # MCP server FastMCP (stdio + http)
│   └── sidecar/         # RAG sidecar (ChromaDB + embeddings + IPC)
├── libs/
│   ├── adapter/         # Bridge layer (JXA, x-callback-url, DT Server)
│   └── schemas/         # Pydantic schemas condivisi
├── scripts/             # init, doctor, smart-rule installer
├── tests/               # pytest (mocking JXA + integration su DT reale)
├── docs/                # ADR, design notes, user docs
├── pyproject.toml       # uv-managed
├── CLAUDE.md            # questo file
├── memory.md            # memoria di sessione
├── handoff.md           # handoff tra sessioni
└── README.md
```

Mantenere la root pulita. Solo config, README, CLAUDE.md, memory.md, handoff.md.

---

## 5. Roadmap e milestone

12 settimane MVP (vedi brief §9). Checkpoint GO/NO-GO:

- **Fine W2**: latenza JXA < 500ms p95 + compatibilità DT4 confermata
- **Fine W6**: vector search migliora qualità rispetto a BM25-only

Performance target: read p95 < 500ms, RAG p95 < 2s.

---

## 6. Convenzioni di codice (delta rispetto a `~/.claude/CLAUDE.md`)

Le regole globali si applicano. In più, per questo progetto:

- **Async-first**: tutto il core (bridge, sidecar IPC, HTTP) deve essere `async`. No blocking I/O nel main loop.
- **Logging**: `structlog` con campi obbligatori `{timestamp, level, tool, duration_ms, audit_id, error?}`. Mai `print()`.
- **Schema Pydantic**: ogni tool input/output deve avere modello Pydantic v2 dedicato in `libs/schemas/`.
- **Tool description**: ogni tool MCP deve avere description con sezioni "Quando usare", "Non usare per", "Esempi". Massimo 2KB.
- **Test JXA**: mocking via fixture pytest che simulano output `osascript`. Smoke test integration su DT reale solo in CI manuale.
- **Branch naming**: `feat/`, `fix/`, `docs/`, `refactor/`, `test/`, `chore/`. Conventional Commits in inglese.
- **Niente codice GPL**: prima di importare qualsiasi libreria, verificare licenza compatibile (MIT, Apache 2.0, BSD).

---

## 7. Aree aperte / decisioni da validare

Da risolvere prima dell'implementazione (vedi brief §8 e §12):

1. ChromaDB vs Qdrant per il sidecar (proposta: ChromaDB in v1, abstract via interface)
2. Sidecar embedded vs processo separato (proposta: separato, IPC via Unix socket)
3. Sync via smart-rule vs polling (proposta: smart-rule + reconciliation notturno)
4. Scope MVP: 3 tool (search, ask_database, find_related) read-only vs 5+ con write ops
5. DT Server bridge: progettare astrazione in v1 anche se non implementato?
6. Strategia di test per dipendenza GUI app
7. Tool versioning per breaking changes
8. Granularità OAuth scope
9. Backward compat DT3

**Quando si tocca una di queste aree, segnalarlo esplicitamente in chat e nel commit.**

---

## 8. Comportamenti attesi dall'assistente

- **Riferirsi sempre al brief** prima di proporre alternative architetturali. Se proponi qualcosa di diverso, motiva.
- **Non implementare** niente prima del review architetturale Cowork (siamo in fase pre-ADR). Eccezione: scaffold repo, CI, file di configurazione, documenti.
- **Confidence level**: dichiarare alta/media/bassa quando si propongono scelte tecniche.
- **Distinguere fact / assumption / proposal** in chat.
- **Tutto user-facing in italiano**, codice e commenti in inglese (regola globale).
- **Mai mockare la dictionary JXA** senza verificare la sintassi reale su un esempio testato (DT4 ha cambiato cose dalla 3.x — `Application("DEVONthink")` non `Application("DEVONthink 3")`).

---

## 9. Riferimenti operativi

- Brief architetturale: `/Users/stefanoferri/Downloads/ARCH-BRIEF-DT-MCP.md`
- MCP spec: https://modelcontextprotocol.io
- FastMCP: https://github.com/jlowin/fastmcp
- DEVONthink JXA reference: https://bru6.de/jxa/automating-applications/devonthink/
- Connector esistente (study only, NO copy): https://github.com/dvcrn/mcp-server-devonthink

---

*Ultimo aggiornamento: 2026-04-30*

---
> Source: [istefox/istefox-dt-mcp](https://github.com/istefox/istefox-dt-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
