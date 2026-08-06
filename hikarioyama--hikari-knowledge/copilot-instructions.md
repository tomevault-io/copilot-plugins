## hikari-knowledge

> You were given this repository (URL or local path). Treat it as a **curated English knowledge graph of measured results, conditioned reports, and tombstones for local-LLM systems**, not a chat log and not a private vault dump.

# AGENTS.md — how to use Hikari-knowledge

You were given this repository (URL or local path). Treat it as a **curated English knowledge graph of measured results, conditioned reports, and tombstones for local-LLM systems**, not a chat log and not a private vault dump.

Canonical remote (may be private or public):

```text
https://github.com/hikarioyama/Hikari-knowledge
```

If clone fails with 404/auth: the repo is private — ask the user for access (collaborator invite or `gh auth`), then retry. Do not invent content.

Author: Hikari · GitHub `hikarioyama` · X [@Hikari_07_jp](https://x.com/Hikari_07_jp)


## If the user only said “install / 導入して” with this URL

That is enough intent. Do **not** wait for a longer prompt.

```text
1. Resolve vault/workspace path (cwd, open workspace, or ask once)
2. git clone this repo as <vault>/Hikari-knowledge
   (or git pull --ff-only if present)
3. Read skills/hikari-knowledge/SKILL.md → Mode A
4. Create one user-side MOC linking INDEX + a few hubs
5. Bridge a few of the user's related notes → [[upstream-ids]]
6. Report path, HEAD, pull command, collisions
```

Default is subfolder install. Full detail: `docs/OBSIDIAN_MERGE.md`.  
Expanded checklist prompt (optional): `prompts/obsidian-merge.md`.
---

## 30-second orientation

1. Read `INDEX.md` first.
2. Open nodes under `nodes/**/*.md`.
3. Follow frontmatter `links:` and body `[[wikilinks]]`.
4. Prefer `status: active|resolved` over `tombstone` unless you need a dead-end warning.
5. Numbers without conditions are weak. Prefer claims that keep `(n=?, same-condition?, hardware/config)`.

Optional tools:

```bash
# graph integrity
python3 tools/build_graph.py --check

# rebuild related: mirrors + graph.json
tools/sync_vault.sh

# optional MCP (local only; author does not host)
KG_ROOT=$PWD KG_VECTOR=0 python mcp/server.py
```

MCP tools if configured: `kg_index`, `kg_search`, `kg_get`, `kg_neighbors`.

---

## Install into an existing Obsidian / markdown graph

**Ideal:** subfolder clone + bridges from the user's notes.  
Details: `docs/OBSIDIAN_MERGE.md`  
Copy-paste prompt: `prompts/obsidian-merge.md`

**Do not merge node files into a flat root by default.**  
Keep this corpus in its own subfolder so updates stay a `git pull`.

```bash
# From the user's existing vault / knowledge root:
cd /path/to/existing-vault
git clone https://github.com/hikarioyama/Hikari-knowledge.git Hikari-knowledge
# or, if already cloned elsewhere:
# git -C Hikari-knowledge pull --ff-only
```

Result:

```text
existing-vault/
  ...user notes...
  Hikari-knowledge/
    INDEX.md
    nodes/...
    AGENTS.md
    skills/...
```

### Linking from the user's notes
- User notes may link here with short ids: `[[serve-bench-discipline]]`
- Obsidian resolves by filename stem; ids are globally unique inside this repo
- Do **not** rewrite upstream nodes to point at the user's private notes

### If the user already has a knowledge graph with clusters
Preferred: subfolder clone (above).

Alternative (only if user explicitly wants a single tree):

1. Copy or sparse-checkout `nodes/*` into their tree **under a namespace prefix folder**, e.g. `nodes/hikari/...`  
2. **Do not rename ids** unless collision forces it  
3. On collision with an existing id, keep both folders and disambiguate links; never silently overwrite  
4. Record the install path in their vault README

### Update later
```bash
cd Hikari-knowledge && git pull --ff-only
python3 tools/obsidianize.py   # optional
python3 tools/build_graph.py --check
```

If MCP points at this checkout, files are truth — no special migrate step after pull.

---

## When the user asks you to maintain / extend this corpus

Load:

- Install/pull/read: `skills/hikari-knowledge/SKILL.md`
- **Curate / add / update / mid-session subagents:** `skills/hikari-knowledge-curate/SKILL.md`
- Schema: `docs/CONVENTIONS.md` + this file

Hard rules (non-negotiable):

- English only
- Public sources only in `sources:` (GitHub / HF / arXiv / docs)
- No `/home/...`, emails, LAN/Tailscale/IPs, GPU UUIDs, customer/friend data, internal hostnames
- Merge-before-create
- Tombstones for dead ends — do not delete overturned knowledge
- `related:` is generated (`tools/obsidianize.py`); do not hand-edit
- Do not commit/push unless the user explicitly asks

After any node edit:

```bash
python3 tools/obsidianize.py
python3 tools/build_graph.py --check   # dangling should be 0 or intentional
python3 tools/build_graph.py
# update INDEX.md blurb for new/changed nodes
```

---

## Retrieval recipe for agents

| Goal | Do this |
|---|---|
| Orient | `INDEX.md` or `kg_index` |
| Find mechanism | `kg_search` / grep `nodes/` for keywords |
| Read one fact | open node / `kg_get` |
| Expand | follow `links:` / `kg_neighbors` |
| Avoid bad paths | read matching `tombstone` nodes |

High-signal hubs:

- methodology: `mechanism-decomposition-anti-inflation`, `serve-bench-discipline`, `quant-native-model-surgery`
- serving: `dsv4-mtp-sm120`, `nvfp4-kv-cache-sm120-vllm`, `sm120-serving-gotchas`
- specdec: `dspark-aeon27b-beats-dflash`, `dspark-confidence-dynamic-k-dead`, `spec-bench-token-counting`
- training: `alpha-dial-k-expansion`, `esft-delta-adafactor-oom`
- steering: `repe-refusal-single-direction`, `repe-verification-methodology`

---

## What this is not

- Not Hikari's private Japanese vault
- Not auto-synced from local experiments
- Not a place for diary entries, single-shot numbers, or personal desktop ops
- Not a hosted MCP service

Updates land **about weekly** (approximate; skip weeks with no measured additions). Announcements: https://x.com/Hikari_07_jp — prefer `git pull --ff-only` after a post, or poll weekly.

---
> Source: [hikarioyama/Hikari-knowledge](https://github.com/hikarioyama/Hikari-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
