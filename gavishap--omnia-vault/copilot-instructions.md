## omnia-vault

> This folder is an **Omnia Vault**: an Obsidian **LLM Wiki** (captured sources

# AGENTS.md — Omnia Vault Rules

This folder is an **Omnia Vault**: an Obsidian **LLM Wiki** (captured sources
compiled into short, linked, source-traceable notes) + **Graphify code graphs**
of any tracked repos + a **relay** that lets different coding agents (Codex,
Claude Code, …) work the same project with zero copy-paste. The vault is the
project's memory; your chat history is not.

Every agent working in this folder follows these rules. The narrative guide is
[VAULT-GUIDE.md](VAULT-GUIDE.md); Claude-specific wiring is in
[CLAUDE.md](CLAUDE.md).

## The relay — FIRST and LAST thing in every session

- **Session start:** read `_relay/STATE.md` (the baton), run
  `python scripts/relay_tool.py status` and `git log --oneline -10`, then tell
  the user where things stand. Do not re-read raw sources or sweep code to
  resume. If the status tool warns the baton is stale, trust git + the newest
  `Wiki/Logs/` note, then repair STATE.md.
- **Session end (or when the user switches agents):** rewrite
  `_relay/STATE.md` (sections: Now / Just landed / Next / Open questions /
  Watch out — present-tense state, written for a stranger, brutally honest
  about what's unfinished), stamp it with
  `python scripts/relay_tool.py stamp --agent codex --summary "<paragraph>"`
  (use your own agent name), run the maintenance gate, and **commit**.
- Full protocol: `_relay/PROTOCOL.md`.

## Layers

| Layer | Path | Purpose |
|-------|------|---------|
| Raw sources | `Raw/Sources/` | Captured source material (transcripts, articles, notes). Never treat as compiled knowledge. |
| Raw files | `Raw/Files/` | Binary/attachment sources. Gitignored except `.gitkeep`. |
| Compiled Wiki | `Wiki/` | Short, linked, reusable notes compiled from Raw sources. The product. |
| Code graphs | `graphify/<repo>/` | Committed Graphify snapshots of tracked repos (generated — never hand-edit). |
| Relay | `_relay/` | The agent-to-agent baton (STATE.md), history, protocol. |
| Schema | `Schema/` | Frontmatter contracts, naming, lint rules, command reference. |
| Templates | `_templates/` | Note templates for every note type. |
| Tooling | `scripts/` | Deterministic build/lint/audit/relay tools (python stdlib only). |
| Chat archive | `chats/` | Both agents' transcripts as **local memory** (gitignored; secrets redacted on import). |

## Core rules

1. **Write reusable knowledge only under `Wiki/`** — topics in `Wiki/Topics/`,
   concepts in `Wiki/Concepts/`, entities in `Wiki/Entities/`, projects in
   `Wiki/Projects/`, logs in `Wiki/Logs/`.
2. **Never edit Raw sources to "improve" them**; compile them into `Wiki/`
   notes instead.
3. **Every compiled note links to its Raw sources**: `sources` lists real
   files under `Raw/Sources/`, and `source_count == len(sources)`. Never
   invent citations — no source, no claim (capture it as an Open Question).
4. **Search before reading broadly:**
   `python scripts/wiki_tool.py search-catalog --query "text"` first; open Raw
   sources only when compiled notes are insufficient. For code questions, the
   graph comes even earlier — see the 3-Layer Query Rule below.
5. **Maintenance gate before every commit** (use `python3` on macOS/Linux):

   ```bash
   python scripts/wiki_tool.py doctor
   python scripts/wiki_tool.py build
   python scripts/wiki_tool.py lint
   python scripts/wiki_tool.py source-lint
   python scripts/audit_public.py
   ```

   After source ingestion, also
   `python scripts/wiki_tool.py source-scan --update --accept-covered` then
   `source-lint`. All must pass.
6. **Never commit:** secrets, machine-local absolute paths, tracked work-repo
   code (gitignore repo folders; only `graphify/<repo>/` snapshots are
   committed), Obsidian plugin/cache state, or hand edits to generated files
   (`Wiki/catalog.jsonl`, `*/index.md`, `Schema/source-manifest.jsonl`,
   `graphify/`).

## 3-Layer Query Rule (questions about the project/code)

1. **Graph:** `graphify query "<question>" --graph graphify/<repo>/graph.json`
   (also `explain` / `path` / `affected`; overview in
   `graphify/<repo>/_GRAPH_REPORT.md` and `_COMMUNITY_*.md`).
2. **Wiki:** `python scripts/wiki_tool.py search-catalog --query "<topic>"` →
   top 1–3 notes.
3. **Raw:** the cited source file / the specific code files only.

## Workflows (readable by any agent — follow the file, not the folder name)

- Ingest a source: `.claude/skills/llm-wiki-ingest/SKILL.md`
- Ingest a recording or video URL: `.claude/skills/video-ingest/SKILL.md`
- Adopt an existing project: `.claude/skills/import-project/SKILL.md`
- The living plan + plan-aware intel triage ("is this video worth using?"):
  `.claude/skills/war-room/SKILL.md` — plan digest
  (`python scripts/plan_tool.py context`) BEFORE watching; the user gates
  every incorporation and every tool install.
- Answer questions: `.claude/skills/project-context-query/SKILL.md`
- Validate/fix notes: `.claude/skills/llm-wiki-lint/SKILL.md`
- Maintain/commit: `.claude/skills/llm-wiki-maintain/SKILL.md`
- Agent handoff (async baton): `.claude/skills/relay/SKILL.md`
- Cross-model plan/diff review (sync — before high-stakes builds):
  `.claude/skills/sparring/SKILL.md`. Works with Codex as the driver too:
  start with `--driver codex --reviewer claude` and use headless Claude
  (`claude -p ... --permission-mode plan --output-format json`) as the
  read-only critic; same `scripts/spar_tool.py` state, same rules.
- Cross-model decision panel (breadth, before locking a direction):
  `.claude/skills/council/SKILL.md` — five blind seats split across both
  models, anonymized cross-bench review, ruled verdict recorded in
  `_relay/council/`. The driving agent chairs; seats on the other model run
  headlessly (fresh thread/session per seat, never reused).
- GitHub PRs without gh CLI: `.claude/skills/github-pr-api/SKILL.md`
- Obsidian editing: `.claude/skills/obsidian-markdown/SKILL.md` (+ bases,
  canvas, cli, defuddle)
- Outward updates/reports: `.claude/skills/stakeholder-update-writing/SKILL.md`

## Reference

- Frontmatter contracts: `Schema/frontmatter-schema.md`
- Naming rules: `Schema/naming-conventions.md`
- Lint rules: `Schema/lint-checklist.md`
- Every command: `Schema/command-reference.md`
- Worked examples: `Schema/workflow-examples.md`

## Environment

- `python` on Windows, `python3` on macOS/Linux; scripts are stdlib-only.
- Optional tooling check: `python scripts/setup_vault.py --check`
  (git, graphify, crv, ffmpeg, node — all optional, each unlocks a feature).
- On Windows PowerShell set `$env:PYTHONUTF8=1` before `graphify`/`crv`.

---
> Source: [gavishap/omnia-vault](https://github.com/gavishap/omnia-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
