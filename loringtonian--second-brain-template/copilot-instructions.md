## second-brain-template

> <!-- __FILL_FROM_USER__:overview

<!-- __FILL_FROM_USER__:overview
     FOR THE AGENT READING THIS — START HERE.
     This repo is a "Second Brain Template": the *architecture* of a personal
     knowledge-management system, shipped as a blank template you fill in yourself.
     Every place your own data goes is marked __FILL_FROM_USER__.
     Find every fill-in site with:   grep -rn "__FILL_FROM_USER__" .
     Your job when a human adopts this repo: interview them and fill each marker so
     this becomes THEIR second brain. The rules and structure below are battle-tested —
     keep them; swap only the owner-specific specifics. After this file, read
     Orientation_Docs/ROUTER.md (the context-loading decision guide). -->

# Second Brain - Claude Code Configuration

> A personal knowledge management system — years of one person's intellectual output.
> These files represent a lot of personal thinking. **Treat them with care.**
> (Owner identity: `__FILL_FROM_USER__:owner_one_liner` — e.g. "who the owner is in one line.")

@Orientation_Docs/ROUTER.md
@Orientation_Docs/INTELLECTUAL_LANDSCAPE_LITE.md
@MEMORY.md

---

## HARD RULES

### Template A IS the Artifact

When presenting ideas/artifacts: **present Template A directly as markdown.** Not in code blocks, not as a summary, not as a table. The user sees EXACTLY what will be saved.

Before creating ANY file: Present full Template A → Ask "Create? Y/N" → Wait for explicit Y.

### Preserve Exact Language

Keywords are sacred for memory retrieval. **Never paraphrase.** Use the owner's actual words. If the original says "vibe coding," write "vibe coding" - not "intuitive AI-assisted development."

### Never Rename Files or Folders

The owner's recall is keyword-based. Renaming breaks recall. Use original names exactly.

### External vs Original

- **Template A** = the owner's original thinking (e.g. dictations, voice notes, personal notes)
- **Template B** = other people's content (e.g. podcast clips, bookmarks) - always attribute

### Never Assume — Always Investigate

Do not make assumptions about file state, content, or what's wrong. Read the files, trace the logic, verify the state. Spending tokens to properly assess a situation is always preferred over guessing. Find the root cause.

### Browser Verification Protocol

When editing UI files (.html/.css/.js/.py) in browser-based projects (registered in `.claude/hooks/verify_registry.json`):

1. **INTENT** - Before changing, state what the change should achieve visually/functionally
2. **CHANGE** - Make the edit
3. **VERIFY** - Read the project's VERIFY.md, then:
   - Check server: `lsof -i :<port> -t`
   - Start if needed (see VERIFY.md for command)
   - `$B goto http://localhost:<port>/<page>`
   - `$B screenshot /tmp/verify_<project>.png` then Read the screenshot
   - Run structural assertions from VERIFY.md
   - `$B console --errors` for JS errors
   - Compare screenshot against your stated intent
4. **RETRY LOOP** - If verification fails:
   - Diagnose what went wrong (read the screenshot, check console, inspect DOM)
   - Make a targeted fix
   - Re-verify (back to step 3)
   - Max 3 retry attempts. If still failing, report to user with all evidence
5. **UPDATE SPEC** - If the change adds/modifies UI elements, append to VERIFY.md changelog
6. **EVIDENCE** - Attach screenshot when claiming "fixed"

`$B` = your headless-browser harness CLI (`__FILL_FROM_USER__:browser_harness_path`).

Never `open http://localhost:*` -- all testing headless via `$B`.
Never claim a UI fix without verification. If skipped, say "UNVERIFIED".
This protocol does NOT apply to markdown/Second Brain work -- only registered browser projects.

---

## SUB-AGENT DELEGATION

When you dispatch a sub-agent (Task/Agent tool), the prompt you write IS the bottleneck on quality. Use the 10-slot skeleton below. Positive framing only — telling the model "don't do X" primes X. Keep a delegation prior-art doc with your own primary-source citations (`__FILL_FROM_USER__:delegation_rationale_doc`).

```
 1. <purpose>     PURPOSE & CONTEXT      — why this matters, what feeds downstream.
 2. <role>        ROLE                   — one sentence, optional. Skip character sheets.
 3. <return>      RETURN FORMAT (top)    — concrete output schema, up front.
 4. <approach>    GENERAL APPROACH       — shape of the work, not a numbered recipe.
                                           Trust Claude's reasoning. Explicit steps only
                                           when the procedure is genuinely multi-stage rigid.
 5. <examples>    WORKED EXAMPLES        — 1–3 POSITIVE examples in <example> tags.
                                           No negative examples.
 6. <constraints> CONSTRAINTS            — hard rules in POSITIVE framing.
                                           If you must prohibit, pair with the reason.
 7. <verify>      VERIFICATION CHECKLIST — "before returning, confirm X, Y, Z are true."
                                           Forward-looking, replaces "don't do X" warnings.
 8. <done>        DEFINITION OF DONE     — what a complete answer looks like.
                                           No budget, no explicit stop-clause.
 9. <return>      OUTPUT CONTRACT (bot.) — re-pin schema at the bottom (format-drift insurance).
10. <context>     LIVE CONTEXT           — state, absolute paths, prior outputs. Absolute paths only.
```

Hard framing rules:
- **Positive over negative.** Anthropic: *"Tell Claude what to do instead of what not to do."* When a prohibition is unavoidable, pair it with the reason — context defuses priming.
- **General over prescriptive.** Anthropic: *"Prefer general instructions over prescriptive steps. Claude's reasoning frequently exceeds what a human would prescribe."*
- **Forward-looking over backward-looking.** Verification checklists > pre-warned failure modes.
- **Scope over stop-clauses.** Definition of done terminates naturally — no "you have N steps" budgets.
- **XML tag each slot.** Anthropic endorses tag-wrapped sections to reduce misinterpretation.
- **Asymmetric models.** Opus orchestrator writes prompts → Sonnet/Haiku sub-agents execute.
- **Sub-agents do NOT inherit the output style.** A spawned sub-agent gets its own system prompt, not the parent's house style. If you want the sub-agent's *reply to you* to be terse and dense, say so in its prompt (the `<constraints>` slot is the natural home: "report tersely — answer-first, no preamble/postamble, structure over prose").

Before dispatching: invoke `/delegate-check` to lint the draft against this skeleton.

### Generalizable delegation heuristics

These patterns apply broadly across sub-agent prompts (validate them against your own A/B test harness):

- **CONTEXT-paraphrase escape hatch.** When a schema demands verbatim quotes/excerpts but the conceptual connection is real yet un-quotable, give the sub-agent a sanctioned paraphrase form (e.g., `CONTEXT:` prefix on the quote field) so it doesn't fabricate text to satisfy the slot. Applies to any skill with evidence/quote fields.
- **Divergence-check before consolidation.** When you find two places that compute the "same" thing, diff them for algorithmic or constant divergence before proposing consolidation. Different algorithms, ratios, thresholds, or error-recovery paths mean "deduplicating" silently changes behavior at one call site. Any such divergence is UNCERTAIN by default — surface it. Applies to code-cleanup, refactor skills, type-consolidation, legacy-removal.
- **Aggregator-contract clause.** When a sub-agent's output is parsed by a lead agent, name the strict-section contract explicitly: "The N structured sections are the aggregator contract. Any prose outside those headings is optional padding — keep it short and after the last structured section so it does not interleave with the parsed content." Applies to fan-out skills where a lead aggregates reports.

---

## BOUNDARIES

### ALWAYS DO
- Work only in the brain's content roots (e.g. the repo root, `Ingestion_Archive/`)
- Connect ideas to the owner's real projects (see `Orientation_Docs/INTELLECTUAL_LANDSCAPE.md`)
- Use COMPLETE templates - never truncate
- Route raw input through the right ingestion skill rather than hand-creating files

### NEVER DO
- Modify original source files (READ-ONLY)
- Paraphrase the owner's language
- Delete files without explicit approval
- Truncate content when copying/converting
- Destroy data before verifying capture

### ASK FIRST
- Before deleting any file
- Before merging ideas into existing files
- Before creating new folder structures

---

## GIT ETIQUETTE (CRITICAL)

Multiple Claude instances may run concurrently. The working tree is SHARED.

### NEVER
- `git add -A` or `git add .` - Stages other agents' work
- `git reset --hard` - Destroys ALL uncommitted work
- `git clean -fd` - Destroys untracked files

### ALWAYS
- `git status` before committing
- Stage files individually: `git add <specific-file>`
- Ask before destructive operations

---

## DATA PRESERVATION

Before deleting ANY file:
1. Identify what unique info it contains
2. Confirm that info exists in destination
3. **VERIFY** by reading destination - not "feeling confident"
4. Only then delete

If source has URL `x.com/user/status/123456`, destination MUST have that exact URL.

---

## SEARCHING

```bash
grep -ri "keyword" . --include="*.md"
```

Case-insensitive. Truncate search terms for variations. Don't standardize capitalization.

---

## CLI JSON MODES

Installed CLIs with JSON output. Prefer JSON mode when parsing, filtering, or chaining results.

| Tool     | JSON Flag                                      | Use Case                          |
|----------|-------------------------------------------------|-----------------------------------|
| ffprobe  | `-print_format json -show_format -show_streams` | Video duration, codec, resolution |
| yt-dlp   | `-j`                                            | YouTube metadata (60+ fields)     |
| rg       | `--json`                                        | Structured grep: file, line, match |
| sqlite3  | `.mode json`                                    | Query SQLite DBs as JSON          |
| jq       | _(processor)_                                   | Filter/transform any JSON         |
| gh       | `--json field1,field2`                          | GitHub PRs, issues, repos         |
| ollama   | `curl localhost:11434/api/tags`                 | Local model list                  |

---

## SKILLS

Skills live in `.claude/skills/`. Before doing manual work, check if a skill exists. Full descriptions live in each `SKILL.md`.

> **First run? Say "set up my second brain"** → the **`/setup`** skill interviews you (purpose, gaps, who you are, voice, tools, privacy levels) and fills this template in.

This Second Brain Template ships the **architecture-generic** skills below; only the original owner's deeply personal / device-coupled skills (a specific media archive, drive utilities, a private podcast pipeline) were removed. Image generation (`/nano-banana-flash`, `/nano-banana-pro`) **is** included — it works once you set `GEMINI_API_KEY` in `.env`. **Add your own domain skills here** as you build them: `__FILL_FROM_USER__:domain_skills`.

**Ingestion & processing**

| User says... | Skill |
|------------------------------------------|-------------------------|
| "brain dump", "process this", "ingest this" | `/ingest-brain-dump` |
| "make files from this"                   | `/process-content`      |
| "mine my [source]", "calibrate the mining", "idea mining" | `/mine` (triage a large export, calibrate judgment on small batches, then bulk-mine) |

**Search & exploration**

| User says... | Skill |
|------------------------------------------|-------------------------|
| "explore the brain"                      | `/explore-second-brain` |
| "semantic search"                        | `/semantic-search`      |
| "check for duplicates", "do I have this" | `/verify-idea`          |
| "what connects to", "related ideas"      | `/connection-finder`    |
| "group related ideas"                    | `/group-related-ideas`  |

> *The semantic skills above (`/semantic-search`, `/verify-idea`, `/explore-second-brain`, `/connection-finder`) work from day one via keyword `grep`, and get **semantic** once you implement the `scripts/sb_embed.py` embedding stub on your own content — see `SETUP.md`.*

**Generation & ideation**

| User says... | Skill |
|------------------------------------------|-------------------------|
| "generate image" (fast / pro)            | `/nano-banana-flash` · `/nano-banana-pro` |
| "launch this idea"                       | `/launch-idea`          |
| "surprise me"                            | `/surprise-me`          |

**Projects, docs & session**

| User says... | Skill |
|------------------------------------------|-------------------------|
| "set up", "personalize", "make this mine"| `/setup` (first-run interview) |
| "new project", "scaffold project"        | `/new-project`          |
| "project tracker", "project status"      | `/project-tracker`      |
| "load brain", "full context" *(needs setup)* | `/load-brain`           |
| "extract todos"                          | `/extract-todo`         |
| "sync orientation docs"                  | `/sync-orientation-docs`|
| "weekly maintenance", "maintenance"      | `/weekly-maintenance`   |
| "wind down", "checkpoint", "done"        | `/wind-down`            |
| "breadcrumb", "register this"            | `/breadcrumb`           |

**Self-improvement & meta**

| User says... | Skill |
|------------------------------------------|-------------------------|
| "lint this sub-agent prompt"             | `/delegate-check`       |

**Dev, media & disk**

| User says... | Skill |
|------------------------------------------|-------------------------|
| "code cleanup" (8-axis sweep)            | `/code-cleanup`         |
| "tweak this HTML"                        | `/html-tweaker`         |
| "visualize", "viz"                       | `/viz`                  |
| "free up memory", "what's eating my ram" | `/free-memory`          |
| "clean up my disk", "free up space"      | `/disk-cleanup`         |
| "remove watermark", "dewatermark"        | `/remove-watermark`     |

> **Exemplar skill (included):** `media-pipeline-example` shows how a richer skill family is built. (The ingest + connection-finder examples were replaced by the full `/ingest-brain-dump` and `/connection-finder` skills.) A few skills call a helper you install — see `SETUP.md`.

See individual SKILL.md files for full descriptions and trigger phrases.

---

## SUBAGENT MODELS

| Task | Model |
|------|-------|
| Search/exploration | haiku |
| Doc sync | sonnet |
| Complex reasoning | opus |

---

## ENVIRONMENT / MACHINE

> Your machine, OS, and tooling — so the agent makes resource-aware decisions (background jobs, heavy ops, what's safe to kill or clean) and knows your capture surfaces. Calibrated by `/setup` (the machine-&-environment phase); consumed by `/free-memory` and `/disk-cleanup`.

`__FILL_FROM_USER__:machine_profile` — e.g.:
- **Machine:** `<make / chip>` · **RAM:** `<N GB>` · **OS:** `<macOS 15 / Ubuntu 24.04 / …>`
- **Disposable servers** (safe for `/free-memory` to kill — they relaunch): `<local model server, dev server, build watcher>`
- **Interactive apps** (never auto-kill): `<your editor, agent CLI, browser>`
- **Disk:** internal size + any external drive used for archival (`/disk-cleanup` Phase 2)
- **Phone:** `<iOS / Android>` — for capture / inbox tooling
- **Other tooling:** `<CLIs, GPUs, anything the agent should know about>`

---

## DOCUMENT HIERARCHY

Context loads in tiers. `Orientation_Docs/ROUTER.md` is the decision guide and is authoritative — read it first if you need to explain why something was or wasn't loaded.

### Tier 0 — Always loaded
- **CLAUDE.md** — this file (hard rules, skills table, active reminders)
- **Orientation_Docs/ROUTER.md** — tier decision rules
- **Orientation_Docs/INTELLECTUAL_LANDSCAPE_LITE.md** — distilled "who the owner is"
- **MEMORY.md** — your portable cross-session memory (`@`-imported above). Copy it from `MEMORY.example.md` on first run (`cp MEMORY.example.md MEMORY.md`); it's gitignored because it accrues personal facts. Distinct from Claude Code's native `~/.claude/projects/<project>/memory/` store — this one travels with the repo.

### Tier 1 — Brain nav / status / what's-next (ROUTER Rule 2)
Loaded when user asks "what's next", project status, folder routing, near-term direction:
- ORIENTATION.md, SECOND_BRAIN_MASTER_INDEX.md, STATE_OF_SECOND_BRAIN.md, PHASE_2_VISION.md, TODO_MASTER.md

### Tier 2 — Deep intellectual / strategic / personal-data analysis (ROUTER Rule 3)
Loaded for thinkers, influences, ideas, priorities, cross-project synthesis, OR analytical tasks on the owner's own data that ask "what's most interesting/striking/best":
- INTELLECTUAL_LANDSCAPE.md (full), COGNITIVE_PROFILE.md (Template C draft), KEYWORD_GUIDE.md, CONTENT_TAXONOMY.md, PRIVACY_DEPTH.md (privacy-depth 1–5 rubric + publish/load gating)

### Tier 2-Voice — First-person writing (ROUTER Rule 4)
- VOICE_GUIDE.md

### Tier 4 — Full brain snapshot (ROUTER Rule 6)
Only via `/load-brain` — a generated full-brain snapshot (large; not shipped — built on demand from your own content).

### Skill-driven
Each skill declares `required_context_files` in its frontmatter and silently loads them via Step 0. Skill loads supersede tier rules when the skill is invoked.

### Never auto-load
- `Ingestion_Archive/` (on-demand only)
- Oversized reference archives
- The full-brain snapshot (only via `/load-brain`)

---

## VERIFICATION

After creating files, invoke the verify-second-brain agent (`.claude/agents/verify-second-brain.md`) to check:
- Template compliance
- Correct folder placement
- Keyword quality
- Language preservation

**Verification feedback loops 2-3x the quality of output.**

---

## READ-ONLY PROTECTED

Some folders hold irreplaceable source material that must never be modified. List the owner's here:

| Folder | Never modify |
|--------|--------------|
| `__FILL_FROM_USER__:protected_folders` | e.g. an imported biographical timeline / original source archive |

---

## COMMON MISTAKES

1. Starting without reading orientation docs
2. "Improving" the owner's language
3. Deleting apparent duplicates without asking
4. Truncating Template A
5. Doing manual work when a skill exists

---

## CURRENT STATUS

A second brain typically moves through phases — Ingestion → Tooling & Retrieval → Execution. Record where this one is:

`__FILL_FROM_USER__:current_status` — e.g. "Phase 1 (Ingestion): COMPLETE; Phase 2 (Execution): ACTIVE." See STATE_OF_SECOND_BRAIN.md for details.

---

## ACTIVE REMINDERS

> **Mechanism (keep this):** persistent tasks that every new agent should surface at session start / "what's on deck" until done. This is how long-running intentions survive across sessions. The owner's personal reminders were removed — add yours below.

`__FILL_FROM_USER__:active_reminders` — e.g. an open deliverable, an in-progress migration, a security-hardening queue. One checkbox bullet each, with an "Added: DATE" stamp and a one-line "surface until done" note.

---

*For templates: ORIENTATION.md | For projects: INTELLECTUAL_LANDSCAPE.md | For status: STATE_OF_SECOND_BRAIN.md*

---
> Source: [Loringtonian/second-brain-template](https://github.com/Loringtonian/second-brain-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
