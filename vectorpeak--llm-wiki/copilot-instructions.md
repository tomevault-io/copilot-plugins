## llm-wiki

> > Local vault overrides: also follow `_CLAUDE.md` for folder conventions and

# Obsidian Second Brain - Codex CLI Operating Manual

> Local vault overrides: also follow `_CLAUDE.md` for folder conventions and
> Markdown authoring rules. In particular, `._trash&cache/` is reserved for
> temporary stitching/cache files and soft trash, and Mermaid diagrams in new
> Markdown should default to the `forest` theme with compact, whole-graph
> layouts.

This vault runs the **obsidian-second-brain** skill. The skill ships a set of
*commands*: each one is a multi-step instruction file that you (the Codex
agent) should follow when the user's request matches its trigger phrase.

## How to operate

1. Read `_CLAUDE.md` in the vault root, if it exists, to learn the user's
   vault conventions (folder map, daily note format, naming).
2. When the user's request matches a trigger in the tables below, read the
   matching file under `.codex/commands/<name>.md` and follow its
   instructions step by step.
3. Treat the AI-first vault rule (`.codex/references/ai-first-rules.md`) as
   non-negotiable for every note you write: `## For future Claude` preamble,
   rich frontmatter (`type`, `date`, `tags`, `ai-first: true`), `[[wikilinks]]`
   for every person/project/concept, recency markers per external claim,
   sources verbatim, confidence levels where applicable.
4. Do not invent commands. If no command matches, ask the user what they
   want or fall back to plain natural-language help.

## Command routing tables (grouped by category)

### Wiki layer - simplified knowledge operations

| Command | What it does | Read this file |
|---|---|---|
| `/ingest` | LLM_wiki intake protocol - preserve raw sources, extract learning units, and update durable wiki knowledge | `02.wiki/projects/LLM_wiki Wiki Layer Protocol.md`, then `.codex/commands/ingest.md` |
| `/query` | LLM_wiki retrieval protocol - search, explain, synthesize, and answer from vault evidence | `02.wiki/projects/LLM_wiki Wiki Layer Protocol.md`, then `.codex/commands/query.md` |
| `/lint` | LLM_wiki maintenance protocol - audit links, duplicates, stale claims, raw backlog, DailyNotes drift, Mermaid defaults, and Mentor consistency | `02.wiki/projects/LLM_wiki Wiki Layer Protocol.md`, then `.codex/commands/lint.md` |

### Vault - daily writing, capture, find

| Command | What it does | Read this file |
|---|---|---|
| `/obsidian-board` | Show or update a kanban board - flags overdue items, updates from conversation | `.codex/commands/obsidian-board.md` |
| `/obsidian-capture` | Quick idea capture - zero friction, saves to the maintained capture area and mentions in daily note | `.codex/commands/obsidian-capture.md` |
| `/obsidian-daily` | Create or update today's daily note - pulls calendar events, overdue tasks, and conversation context | `.codex/commands/obsidian-daily.md` |
| `/obsidian-find` | Smart vault search - returns results with context, not just filenames | `.codex/commands/obsidian-find.md` |
| `/obsidian-log` | Log this work or dev session to the vault - infers project from context | `.codex/commands/obsidian-log.md` |
| `/obsidian-person` | Create or update a person note from conversation context | `.codex/commands/obsidian-person.md` |
| `/obsidian-project` | Create or update a project note - adds to board and daily note automatically | `.codex/commands/obsidian-project.md` |
| `/obsidian-projects` | Live project status from git + local docs - infers all context from vault notes, no config required | `.codex/commands/obsidian-projects.md` |
| `/obsidian-recap` | Summarize a time period from the vault - today, week, or month | `.codex/commands/obsidian-recap.md` |
| `/obsidian-recurring` | Track a recurring obligation (payment, filing, ops) with a cadence and a computed next-due date | `.codex/commands/obsidian-recurring.md` |
| `/obsidian-save` | Save everything worth keeping from this conversation to the vault | `.codex/commands/obsidian-save.md` |
| `/obsidian-task` | Add a task to the right kanban board with inferred priority and due date | `.codex/commands/obsidian-task.md` |
| `/obsidian-world` | Load your identity, values, priorities, and current state in one shot - with progressive context levels to avoid burning tokens | `.codex/commands/obsidian-world.md` |

### Thinking - synthesis, decisions, learning, reviews

| Command | What it does | Read this file |
|---|---|---|
| `/idea-discovery` | Surface 3-5 next-direction candidates by reading ungraduated ideas, open project questions, and orphan research notes - what is worth working on next | `.codex/commands/idea-discovery.md` |
| `/obsidian-adr` | Generate a decision record when the vault structure changes - the vault knows why it knows what it does | `.codex/commands/obsidian-adr.md` |
| `/obsidian-challenge` | Red-team your current idea against your own vault history - finds contradictions, past failures, and flawed assumptions | `.codex/commands/obsidian-challenge.md` |
| `/obsidian-connect` | Bridge two unrelated domains using your vault's link graph - forces creative friction to spark new ideas | `.codex/commands/obsidian-connect.md` |
| `/obsidian-decide` | Extract decisions from this conversation and log them to the right project notes | `.codex/commands/obsidian-decide.md` |
| `/obsidian-emerge` | Surface unnamed patterns from your recent notes - recurring themes, hidden connections, and conclusions you haven't explicitly stated | `.codex/commands/obsidian-emerge.md` |
| `/obsidian-graduate` | Promote an idea fragment into a full project spec with tasks, board entries, and structure | `.codex/commands/obsidian-graduate.md` |
| `/obsidian-learn` | Review vault learnings, prune stale ones, surface active patterns - the vault's lessons compound or expire | `.codex/commands/obsidian-learn.md` |
| `/obsidian-panel` | Convene a panel of distinct perspectives on a decision - one independent verdict per lens, then a synthesis. A multi-persona complement to /obsidian-challenge | `.codex/commands/obsidian-panel.md` |
| `/obsidian-reconcile` | Find and resolve contradictions in the vault - the vault maintains its own truth | `.codex/commands/obsidian-reconcile.md` |
| `/obsidian-review` | Generate a structured weekly or monthly review note from vault history | `.codex/commands/obsidian-review.md` |
| `/obsidian-synthesize` | Automatic synthesis - scans the vault for unnamed patterns and writes synthesis pages without being asked | `.codex/commands/obsidian-synthesize.md` |
| `/vault-deep-synthesis` | Deep cross-reference of everything the vault knows about one topic - agreements, contradictions, stale claims, and coverage gaps. Pure vault, no network | `.codex/commands/vault-deep-synthesis.md` |

### Research - bring external sources into the vault

| Command | What it does | Read this file |
|---|---|---|
| `/notebooklm` | Vault-first source-grounded research via Gemini File Search. One command, no browser. The grounded parallel to /research-deep (which is open-web via Perplexity). | `.codex/commands/notebooklm.md` |
| `/obsidian-ingest` | Ingest a source into the vault - the vault rewrites itself around new knowledge. Every ingest updates entities, rewrites stale claims, synthesizes new concepts, and resolves contradictions. | `.codex/commands/obsidian-ingest.md` |
| `/podcast` | Extract metadata, transcript, and summary from a podcast episode, saved as an AI-first note in the vault | `.codex/commands/podcast.md` |
| `/research` | Web research with citations - Perplexity Sonar when an API key is set, free key-less sources (Wikipedia, HackerNews, arXiv, Reddit, and more) otherwise. Deep dossier with summary, facts, timeline, players, contrarian views, open questions | `.codex/commands/research.md` |
| `/research-deep` | Vault-first deep research - scans the vault, fills gaps (Perplexity + Grok when keyed, free key-less sources otherwise), synthesizes a delta, then propagates updates across people/projects/ideas via /obsidian-save | `.codex/commands/research-deep.md` |
| `/x-pulse` | Scan X for what's trending in a topic - themes, voices, hooks, and post ideas powered by Grok + Live Search | `.codex/commands/x-pulse.md` |
| `/x-read` | Deep-read an X (Twitter) post via Grok + Live Search - verbatim post, thread, TL;DR, claims, reply sentiment, voices to watch | `.codex/commands/x-read.md` |
| `/youtube` | Extract transcript, metadata, and top comments from a YouTube video - summarized via Grok and saved to vault | `.codex/commands/youtube.md` |

### Meta - vault setup, health, structure

| Command | What it does | Read this file |
|---|---|---|
| `/create-command` | Create a new obsidian-second-brain command via interview - zero markdown editing required | `.codex/commands/create-command.md` |
| `/obsidian-architect` | Scan a codebase and write a maintained set of architecture notes into the vault - overview, per-module notes, key decisions. Re-run to refresh without clobbering your edits | `.codex/commands/obsidian-architect.md` |
| `/obsidian-export` | Export a clean structured snapshot of the vault that any agent or tool can consume - flat JSON or markdown index | `.codex/commands/obsidian-export.md` |
| `/obsidian-health` | Run a vault health check - grouped by severity, detects contradictions, concept gaps, stale claims, and structural issues | `.codex/commands/obsidian-health.md` |
| `/obsidian-init` | Scan your vault and generate a _CLAUDE.md operating manual, index.md catalog, and `.codex/references/log.md` pointer | `.codex/commands/obsidian-init.md` |
| `/obsidian-visualize` | Generate a visual canvas map of your vault - see the shape of your second brain and how knowledge connects | `.codex/commands/obsidian-visualize.md` |

## Trigger phrases

When the user says any of the phrases below (or a close paraphrase), follow the matching command file.

### English (`en`)

**Wiki layer - simplified knowledge operations**

- `/ingest` - "ingest", "import to wiki", "add to wiki", "absorb into wiki", "导入", "吸收", "入库", "收进知识库", "剪藏入库"
- `/query` - "query wiki", "ask wiki", "search wiki", "explain from wiki", "what does my vault say", "查询", "问 wiki", "查一下知识库", "从我的笔记里回答", "解释这个概念"
- `/lint` - "lint wiki", "check wiki", "audit wiki", "vault lint", "knowledge base health", "体检", "检查 wiki", "检查知识库", "清理知识库"


**Vault - daily writing, capture, find**

- `/obsidian-board` - "show board", "kanban", "what is on my board", "update board"
- `/obsidian-capture` - "capture this idea", "save this idea", "quick note", "drop a thought"
- `/obsidian-daily` - "todays note", "create todays daily", "open daily", "today daily note"
- `/obsidian-find` - "find in vault", "search my notes", "where is", "what did I write about"
- `/obsidian-log` - "log this work", "log this session", "log this dev session", "obsidian log"
- `/obsidian-person` - "save this person", "add person", "new contact note", "create person note"
- `/obsidian-project` - "new project", "create project note", "project setup", "start a project"
- `/obsidian-projects` - "projects overview", "project status", "what am I working on", "show projects"
- `/obsidian-recap` - "recap today", "recap the week", "summarize the week", "month recap"
- `/obsidian-recurring` - "recurring task", "monthly obligation", "remind me every month", "recurring payment", "track a recurring"
- `/obsidian-save` - "save this", "save the conversation", "save to vault", "obsidian save"
- `/obsidian-task` - "add task", "new todo", "track this", "remind me"
- `/obsidian-world` - "load context", "what is going on", "where am I", "load my world"

**Thinking - synthesis, decisions, learning, reviews**

- `/idea-discovery` - "what should I work on next", "idea discovery", "surface next directions", "what's worth pursuing"
- `/obsidian-adr` - "log this decision", "ADR", "record decision", "decision record"
- `/obsidian-challenge` - "challenge this", "grill me on this", "red team my idea", "stress test this"
- `/obsidian-connect` - "connect domains", "cross-pollinate", "bridge ideas", "find an unexpected link"
- `/obsidian-decide` - "extract decisions", "log decisions", "what did we decide"
- `/obsidian-emerge` - "find patterns", "what is emerging", "surface themes", "unnamed patterns"
- `/obsidian-graduate` - "promote idea", "graduate this to project", "make a project from this", "elevate idea"
- `/obsidian-learn` - "review learnings", "what have I learned", "show lessons", "prune learnings"
- `/obsidian-panel` - "convene a panel", "advisor panel", "get multiple perspectives on", "panel review", "what would the experts say about"
- `/obsidian-reconcile` - "find contradictions", "reconcile vault", "fix conflicts", "vault contradictions"
- `/obsidian-review` - "weekly review", "monthly review", "review my week", "review my month"
- `/obsidian-synthesize` - "synthesize", "auto-synthesis", "make synthesis notes", "find unnamed patterns"
- `/vault-deep-synthesis` - "synthesize what I know about", "deep synthesis on", "cross-reference my notes on", "what does my vault say about"

**Research - bring external sources into the vault**

- `/notebooklm` - "notebooklm", "research grounded", "ground research in vault", "ask my notebook", "source-grounded research"
- `/obsidian-ingest` - "ingest this source", "add this article", "import this", "absorb this"
- `/podcast` - "summarize this podcast", "podcast episode summary", "extract podcast", "what's in this episode"
- `/research` - "research this", "look up", "find information about", "perplexity research"
- `/research-deep` - "deep research", "thorough research", "vault-first research", "research gaps"
- `/x-pulse` - "x pulse", "what is trending on twitter", "scan x for", "twitter pulse"
- `/x-read` - "read this x post", "deep read this tweet", "analyze this tweet", "read this thread"
- `/youtube` - "summarize youtube", "youtube transcript", "extract video", "youtube to vault"

**Meta - vault setup, health, structure**

- `/create-command` - "create command", "new command", "add a command", "scaffold a command"
- `/obsidian-architect` - "document this codebase", "architect this project", "map this code into my vault", "generate architecture notes", "refresh architecture docs"
- `/obsidian-export` - "export vault", "snapshot vault", "dump vault", "vault export"
- `/obsidian-health` - "vault health", "check vault", "audit vault", "vault diagnostics"
- `/obsidian-init` - "init vault", "bootstrap vault", "setup vault", "scan vault"
- `/obsidian-visualize` - "visualize vault", "vault map", "canvas of vault", "show me the vault shape"

## Trigger conventions

Each command file has a leading paragraph that documents its trigger phrases
in plain English. Treat those triggers as primary; the slash-form `/name` is
also accepted if the user types it.

## Scripts

Python helpers live under `.codex/scripts/`. They run via `uv run -m
scripts.research.<name>` from the vault root (or wherever the skill is
installed). The commands that need them reference the exact invocation
inside the command body.

---

*Generated by adapters/codex-cli/adapter.sh - do not edit manually.*

---
> Source: [VectorPeak/LLM-Wiki](https://github.com/VectorPeak/LLM-Wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
