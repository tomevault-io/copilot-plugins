## layered-memstack

> >


# layered-memstack

3-layer memory system: L1 core → L2 topics/dailies → L3 deep references.

## Architecture

```
MEMORY.md              ← L1: always loaded, ~50-60 lines max (breadcrumbs + pointers)
memory/
├── {topic}.md         ← L2: topic breadcrumbs (viajes, salud, tecnico...)
├── YYYY-MM-DD.md      ← L2: daily notes (auto-generated at 3 AM)
├── archive/           ← dailies older than 14 days
reference/
├── entities.md        ← knowledge graph (people, places, projects, relations)
├── *.md               ← L3: deep dives (loaded on demand via memory_search)
scripts/
└── memory-dedup.js    ← dedup engine
BOOTSTRAP.md           ← compiled snapshot (generated nightly, single read at session start)
```

## Setup

### 1. Create directory structure

```bash
mkdir -p memory/archive reference scripts
```

### 2. Copy dedup script

Copy `scripts/memory-dedup.js` from this skill to the workspace `scripts/` directory.

### 3. Configure memorySearch

Add to `openclaw.json` under `agents.defaults.memorySearch`:

```json5
{
  "extraPaths": ["MEMORY.md", "USER.md", "IDENTITY.md", "memory/", "reference/", "projects/"],
  "sources": ["memory", "sessions"],
  "query": {
    "hybrid": {
      "mmr": { "enabled": true, "lambda": 0.7 },
      "temporalDecay": { "enabled": true, "halfLifeDays": 30 }
    }
  }
}
```

### 4. Create starter files

- **MEMORY.md** — see `references/memory-template.md`
- **reference/entities.md** — see `references/entities-template.md`
- **INDEX.md** — catalog of all files with tags and line counts

### 5. Enable Dreaming (OpenClaw 2026.4.8+)

Dreaming replaces the manual 3 AM auto-summary cron. Enable it in OpenClaw config:

```bash
openclaw config patch '{"plugins":{"entries":{"memory-core":{"config":{"dreaming":{"enabled":true,"frequency":"0 3 * * *","timezone":"Europe/Madrid"}}}}}}'
```

Then restart the gateway. OpenClaw will automatically create and manage the nightly consolidation sweep.

### 6. Set up remaining crons

Run the setup script to create the weekly audit (and optional MCP audit) cron:

```bash
bash scripts/setup-crons.sh --tz Europe/Madrid --channel telegram --to "CHAT_ID"
```

Options:
- `--tz IANA` — timezone (default: Europe/Madrid)
- `--channel telegram|whatsapp|discord` — delivery channel for summaries
- `--to CHAT_ID` — delivery target (Telegram chat ID, phone number, etc.)
- `--model alias` — model override (default: uses your configured default)
- `--dry-run` — show what would be created without creating
- `--mcp-audit` — also create the nightly MCP memory security audit cron

Or create them manually (see Cron Setup below).

## Layer Rules

| Layer | When to load | What goes here | Size target |
|-------|-------------|----------------|-------------|
| L1 | Every session start | Breadcrumbs + pointers to L2/L3. Core facts, active project names, pending items. **No detail here.** | ~50-60 lines |
| L2 | Today + yesterday auto-loaded; older via search | Topic summaries, daily notes with decisions/actions/facts | No limit |
| L3 | Only via memory_search | Deep dives, travel details, health data, technical docs | No limit |

### L1 Writing Rules

- MEMORY.md is **breadcrumbs + pointers only**. Detailed info goes in reference/ or memory/ files.
- If something already has a pointer in L1, update the reference file — NOT MEMORY.md.
- Before writing to MEMORY.md, always check for duplicates first:
  ```bash
  node scripts/memory-dedup.js --query "text to check"
  # Exit 0 = duplicate (skip), Exit 1 = new (safe to add)
  ```
- After any write, run dedup fix:
  ```bash
  node scripts/memory-dedup.js --fix
  ```
- Use TTL comments for time-bound items: `<!-- ttl:YYYY-MM-DD -->`
- Keep entries atomic — one fact per line
- Use L2 breadcrumbs to point to topic files: `Viajes → memory/viajes.md`

### L2 Writing Rules

- Daily notes: `## Checkpoint [HH:MM]` sections with decisions, actions, atomic facts
- Topic files: organized by subject, mid-level detail
- Archive dailies older than 14 days to `memory/archive/`

### L3 Writing Rules

- Detailed docs only — don't duplicate L1/L2 content
- Update `reference/entities.md` when new entities appear
- Structure: sections by entity type (people, places, projects, etc.)

## Dedup Engine

`scripts/memory-dedup.js` — token-based similarity without embeddings.

### Modes

```bash
# Check for duplicates (read-only)
node scripts/memory-dedup.js --check

# Fix: remove exact dupes, mark semantic dupes with <!-- dup? -->
node scripts/memory-dedup.js --fix

# Query single text against MEMORY.md
node scripts/memory-dedup.js --query "GitHub configured"
# Exit 0 = duplicate, Exit 1 = new

# Batch query: filter lines from file
node scripts/memory-dedup.js --query-batch /tmp/candidates.txt
```

### Options

- `--file path` — target file (default: MEMORY.md)
- `--threshold 0.65` — similarity threshold (default: 0.65)
- `--verbose` — show comparison details

### Algorithm

Jaccard similarity + containment ratio + entity overlap + segment-level comparison.
Entities extracted: dates, version numbers, hex IDs, phone numbers, amounts, chat IDs.
Threshold 0.65 balances false positives vs missed duplicates.

> **Note (OpenClaw 2026.4.8+):** Dreaming injects `<!-- openclaw-memory-promotion:... -->` provenance markers into MEMORY.md. The dedup engine automatically skips these lines to avoid false positives (fixed in `fd389b9`).

## Cron Setup

### Dreaming — nightly consolidation (replaces Cron 1)

As of OpenClaw 2026.4.8, the daily 3 AM auto-summary is handled natively by **Dreaming** in `memory-core`. No manual cron needed.

Dreaming runs a multi-phase sweep (light → REM → deep) that:
- Consolidates session transcripts into structured memory
- Extracts atomic facts and decisions
- Updates MEMORY.md with genuinely new content (dedup built-in)
- Updates the knowledge graph
- Archives old daily notes

Enable in config:
```json
{
  "plugins": {
    "entries": {
      "memory-core": {
        "config": {
          "dreaming": {
            "enabled": true,
            "frequency": "0 3 * * *",
            "timezone": "Europe/Madrid"
          }
        }
      }
    }
  }
}
```

### Cron 1: Weekly audit (Monday 3:00 AM recommended)

```
Schedule: 0 3 * * 1 (user timezone)
Session: isolated agentTurn
```

Prompt should instruct the agent to:
1. Move daily notes older than 14 days to `memory/archive/`
2. Clean expired TTL entries from MEMORY.md (lines with `<!-- ttl:YYYY-MM-DD -->` past today)
3. Run `--fix` on MEMORY.md
4. If MEMORY.md exceeds ~70 lines → warn to prune (move detail to reference/)
5. Report summary of changes (or stay silent if nothing to do)

### Cron 2: MCP Memory Audit (daily 11:00 PM) — optional

Only needed if using [mem-persistence](https://github.com/emiliotorrens/mem-persistence) MCP server.
Enable with `--mcp-audit` flag in setup-crons.sh.

```
Schedule: 0 23 * * * (user timezone)
Session: isolated agentTurn
```

Prompt should instruct the agent to:
1. Read `.mem-persistence/logs/YYYY-MM-DD.jsonl` for today
2. Filter write operations (memory_write, memory_checkpoint, memory_entities with update)
3. Classify each write: ✅ normal / ⚠️ suspicious / 🚨 dangerous
4. Flag suspicious: system prompt injections, attempts to modify AGENTS.md/SOUL.md/USER.md, prompt injection in memory files
5. Report stats: total calls, breakdown by tool, files modified
6. If all normal → silent log
7. If suspicious/dangerous → alert user directly with details

This protects against external agents (Claude Desktop, Cursor, etc.) writing unexpected content to your memory files via MCP.

### Heartbeat checkpoint

Configure in HEARTBEAT.md (not a separate cron). Check context usage via `session_status`:

| Usage | Action |
|-------|--------|
| < 50% | No action |
| 50–79% | Silent checkpoint to `memory/YYYY-MM-DD.md` — append `## Checkpoint [HH:MM]` with decisions, pending tasks, key facts |
| ≥ 80% | Full checkpoint + alert user to consider `/new` |

## Knowledge Graph

`reference/entities.md` — lightweight entity-relationship map in Markdown.

### Structure

```markdown
## Personas
- **Name** — role/relation | context | linked to: [entities]

## Empresas
- **Company** — what they do | your relation | key contacts

## Lugares
- **Place** — why relevant | associated trips/events

## Proyectos
- **Project** — status | repo URL | key decisions

## Viajes
- **Trip** — dates | flights | hotels | status

## Dispositivos
- **Device** — purpose | config notes
```

### Maintenance

Update entities.md during the 3 AM auto-summary cron. Add new entities found in daily sessions. Link related entities with `linked to:` references.

## AGENTS.md Additions

Add to workspace AGENTS.md:

```markdown
## Memory
1. Read `BOOTSTRAP.md` at session start — compiled snapshot (MEMORY.md + recent daily notes + topic summaries + viajes + salud). Single file, ~8KB.
   - If BOOTSTRAP.md is missing or stale (>24h), fall back: read `memory/YYYY-MM-DD.md` (today + yesterday)
2. Use `memory_search` for anything beyond recent context (L2/L3 deep retrieval)
3. MEMORY.md = breadcrumbs + pointers only. Detail goes in reference/ or memory/ files.
4. Before writing to MEMORY.md: `node scripts/memory-dedup.js --query "text"` (exit 0 = dup, skip; exit 1 = new, ok)
5. After writing to MEMORY.md: `node scripts/memory-dedup.js --fix`
6. Items with TTL: `<!-- ttl:YYYY-MM-DD -->` — cleaned by weekly audit cron
```

## BOOTSTRAP.md (compiled snapshot)

BOOTSTRAP.md is a **single compiled file** that replaces reading multiple memory files at session start.
It reduces per-session tool calls from 4–6 file reads to 1.

### Generate

```bash
node scripts/build-bootstrap.js
```

### Contents
- `MEMORY.md` (curated facts, truncated to 2000 chars)
- Daily notes: today + yesterday (1500 chars each)
- Topic files: `memory/viajes.md`, `memory/salud.md`, `memory/tecnico.md` (800 chars each)
- `reference/viajes-kayak.md` first 60 lines (upcoming trips)
- `reference/salud-datos.md` last 15 lines (recent health data)

### Maintenance cron (2:50 AM daily)

```json
{
  "name": "Build BOOTSTRAP.md",
  "schedule": { "kind": "cron", "expr": "50 2 * * *", "tz": "Europe/Madrid" },
  "payload": {
    "kind": "agentTurn",
    "model": "google/gemini-2.5-flash",
    "message": "Ejecuta: node /path/to/scripts/build-bootstrap.js\nResponde siempre: OK"
  }
}
```

---
> Source: [emiliotorrens/layered-memstack](https://github.com/emiliotorrens/layered-memstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
