## claude-skill-dump

> Persist session knowledge (conventions, gotchas, decisions, learnings) into durable documentation: the project's authoritative agent file (CLAUDE.md or AGENTS.md — whichever the project uses, never both), README.md, docs/standards/, ~/source/standards/ (prescriptive cross-project rules), ~/source/knowledgebase/ (descriptive cross-project facts), and other docs/ as needed. **Trigger only on the explicit `/dump` slash command.** Do not trigger on natural-language phrases — the user wants deterministic invocation, never inferred intent. Dispatches parallel subagents per documentation target, writes directly, reports the list of changed paths after.


# dump

## Pre-flight inventory

The following listings are injected once at skill load. Refer back to them from later steps instead of re-running these commands. If a path is missing the output will say so — treat that as "not present" and act accordingly.

### Root agent/human docs (auto-injected)

ls -1 README.md AGENTS.md CLAUDE.md:

!`ls -1 README.md AGENTS.md CLAUDE.md 2>&1 || true`

Referenced by: Step 2 routing (which root files exist), Step 3 dispatch (target picker for CLAUDE.md vs AGENTS.md), edge cases (no CLAUDE.md/AGENTS.md/README.md).

### Project docs tree  (auto-injected)

ls -la docs/:

!`ls -la docs/ 2>&1 || true`

ls -la docs/standards/:

!`ls -la docs/standards/ 2>&1 || true`

Referenced by: Step 2 routing (existing `docs/` structure determines whether a nugget gets a new file or joins an existing one), Step 3 dispatch (subagent scopes for `docs/standards/` and other `docs/*`), routing heuristics (architectural decision logs, troubleshooting docs).

### Cross-project standards  (auto-injected)

!`${CLAUDE_SKILL_DIR}/scripts/resolve-listing.sh standards`

**Shallow** listing — top level only. For file-level routing of a nugget, read `CLAUDE.md` in the listed path; it is the index mapping topics to files.

Referenced by: Step 3 dispatch (subagent must read the standards repo's `CLAUDE.md` for file-level routing), routing heuristics (where cross-repo patterns live).

### Cross-project knowledgebase  (auto-injected)

!`${CLAUDE_SKILL_DIR}/scripts/resolve-listing.sh knowledgebase`

**Shallow** listing — top level only. For file-level routing of a nugget, read `CLAUDE.md` in the listed path; it is the index mapping topics to files.

Knowledgebase is the **descriptive** sibling of standards: vendor quirks, tool internals, API taxonomies, discovered failure modes — "this is how X works", not "do it this way". A nugget routes here when it describes how an outside vendor/tool/API behaves; it routes to standards when it prescribes how to write code. Hybrid topics live in both, cross-linked.

Referenced by: Step 2 routing (descriptive vs prescriptive split), Step 3 dispatch (subagent must read the knowledgebase repo's `CLAUDE.md` for file-level routing), routing heuristics.

### Repo mode and ownership (auto-injected)

${CLAUDE_SKILL_DIR}/scripts/check-user-owned.sh:

!`${CLAUDE_SKILL_DIR}/scripts/check-user-owned.sh`

Output format depends on mode:

- `MODE: single` + `OWNERSHIP: APPLICABLE|NOT_APPLICABLE: <reason>` — CWD is a git repo.
- `MODE: multi` + one `REPO: <name> APPLICABLE|NOT_APPLICABLE: <reason>` per child repo — CWD is a parent directory containing child repos.
- `MODE: none` — no git repo at CWD and no child repos found.

All modes except `none` also emit:

- `STANDARDS: <absolute-path>|NOT_FOUND` — resolved location of standards repo (checks `~/source/standards/`, `~/standards/`, `./standards/` in order; first git repo wins).
- `KNOWLEDGEBASE: <absolute-path>|NOT_FOUND` — resolved location of knowledgebase repo (same resolution order).

If `STANDARDS` or `KNOWLEDGEBASE` is `NOT_FOUND`, skip that subagent entirely — do not attempt to discover or create it later.

Referenced by: Step 0 multi-repo detection (determines mode, lists child repos, per-repo ownership, and cross-project repo availability), Step 1 environment detect (gates `~/source/standards/` and `~/source/knowledgebase/` subagent dispatch in single-repo mode), Step 4 report (`Skipped` section cites the reason when `NOT_APPLICABLE` or `NOT_FOUND`).

### Remote URLs (auto-injected)

${CLAUDE_SKILL_DIR}/scripts/get-remotes.sh:

!`${CLAUDE_SKILL_DIR}/scripts/get-remotes.sh`

Output: six lines. Both the raw form (whatever `git remote get-url origin` returned — SSH or HTTPS) and the converted public HTTPS form are pre-computed so subagents never have to convert. Standards and knowledgebase paths are resolved using the same logic as `check-user-owned.sh` (`~/source/<name>`, `~/<name>`, `./<name>` — first git repo wins).

- `LOCAL_REMOTE_RAW: <url|none>` — origin of the local repo (cwd or `$1`), as stored in `.git/config`.
- `LOCAL_REMOTE_HTTPS: <url|none>` — same, converted to `https://` browser form.
- `STANDARDS_REMOTE_RAW: <url|none>` — origin of the resolved standards repo.
- `STANDARDS_REMOTE_HTTPS: <url|none>` — same, converted to `https://` browser form.
- `KNOWLEDGEBASE_REMOTE_RAW: <url|none>` — origin of the resolved knowledgebase repo.
- `KNOWLEDGEBASE_REMOTE_HTTPS: <url|none>` — same, converted to `https://` browser form.

The model must use these injected values directly, **never reconstruct them from filesystem paths, do its own SSH→HTTPS conversion, or guess from context**. Anti-pattern reminder: cross-repo references in any written doc must use the `_HTTPS` value verbatim.

Referenced by: Step 3 dispatch (subagent prompts include the `_HTTPS` URL for provenance and cross-repo references), anti-patterns (cross-repo reference rule).

## Purpose

Sessions accumulate context that dies on `/compact`: conventions discovered, gotchas hit, decisions made, patterns established, command incantations, error/fix pairs, and design rationale. This skill harvests that context and persists it into the right durable documents so future sessions start informed.

Note: `/clear` cannot be intercepted — it wipes context before any model invocation. To dump before clearing, the user must invoke `/dump` first.

## When to trigger

**Only on the explicit `/dump` slash command.** Determinism is the goal — the user wants to know exactly when this runs.

Do **not** trigger on natural-language phrases ("dump context", "save learnings", "wrap up session", "before I compact", etc.), even when the surface intent looks identical. If the user expresses dump intent without typing `/dump`, prompt them: "Run `/dump`?" via `AskUserQuestion` rather than firing the skill silently.

Do **not** auto-trigger on session end, commits, `/compact`, `/clear`, or PR creation.

## Workflow

### Step 0: Multi-repo detection

Check the **Repo mode and ownership** pre-flight output:

- `MODE: single` → CWD is a git repo. **Skip to Step 1** (single-repo mode). The `OWNERSHIP:` line carries forward to Step 1.
- `MODE: multi` → CWD is a parent directory with child repos. Enter **multi-repo mode** (below). The `REPO:` lines list each child repo and its ownership status.
- `MODE: none` → report "no git repo at CWD and no child repos found" and exit.

#### Multi-repo mode

When CWD is a parent directory containing child repos (e.g., a sandbox or monorepo root), the skill processes all session-relevant child repos in a single invocation. After this section completes, **skip Steps 1–4** — they are single-repo only.

**0a. Identify session-relevant repos.** Scan the conversation for which child repos were touched — files read/edited, Bash commands run in their directories, code discussed. Build a set of child repo paths from the pre-flight listing. Repos with no session activity are skipped entirely.

**0b. Run per-repo pre-flight.** Ownership is already known from the `REPO:` lines. For each session-relevant child repo, run via the Bash tool (all repos in parallel):

- `${CLAUDE_SKILL_DIR}/scripts/get-remotes.sh <repo-path>` — remote URLs
- `ls -1 <repo-path>/README.md <repo-path>/AGENTS.md <repo-path>/CLAUDE.md 2>&1 || true` — root doc files
- `ls -la <repo-path>/docs/ 2>&1 || true` — docs tree

**0c. Per-repo environment detection.** Apply Step 1 logic (agent-file authority, ownership gate) independently per child repo using that repo's pre-flight results.

**0d. Inventory and route.** Build the session knowledge inventory (Step 2 logic) once across the whole session, but route each nugget to the specific child repo it pertains to — the repo whose code, gotcha, or workflow the nugget describes. Cross-project targets (`~/source/standards/`, `~/source/knowledgebase/`) are shared: the ownership gate passes if **any** session-relevant child repo is user-owned. Duplicate-check per the usual Step 2 rules: grep each target file for the concept before adding it to the routing map.

The routing map in multi-repo mode groups by repo:

```
# Routing map (multi-repo mode, N repos, M subagents)

## <child-repo-1>/
### <child-repo-1>/<target>
- <nugget summary>

## <child-repo-2>/
### <child-repo-2>/<target>
- <nugget summary>

## ~/source/standards/ (shared)
- <nugget summary> (from <repo>)

## Skipped (already documented)
- <nugget summary> — found in <path>

## Skipped (ephemeral)
- <nugget summary>
```

Print the routing map, then proceed immediately to dispatch — same contract as single-repo.

**0e. Dispatch.** Apply Step 3 dispatch logic — one subagent per target file per repo. Cross-project targets get one subagent each regardless of how many repos contributed nuggets. All subagents dispatch in a **single message** for parallelism. Each subagent prompt must use the **per-repo** remote URLs from step 0b (not the parent-level pre-flight, which has `none`). File paths in subagent prompts are absolute (e.g., `/sandbox/source/claude-dashboard/CLAUDE.md`).

**0f. Report.** Apply Step 4 report logic with `Changed` grouped by repo:

```
# Dump report

## Changed
### <child-repo-1>/
- <path>
### <child-repo-2>/
- <path>

## Skipped
- <repo>: <reason>

## Standards / Knowledgebase TL;DR (cross-project repos only)
- <repo>/<file>: <one-line what and why>

## Verify (subagent bullets that look off — re-read these)
- <path>: <which bullet, what looks wrong>

## Suggested follow-ups
- <anything that didn't fit>
```

Same rules as Step 4: no commentary, no per-file breakdown, omit empty sections.

### Step 1: Detect environment

Everything is already injected at the top of this skill under **Pre-flight inventory** — doc listings, local-repo ownership, and both remote URLs. Do not re-run any of those commands.

Two gates to read from the injections:

1. **Cross-project repo gates** — two independent checks from the pre-flight output:
   - **Ownership**: if `OWNERSHIP: NOT_APPLICABLE`, skip standards and knowledgebase subagents and carry the reason into the report's `Skipped` section.
   - **Availability**: if `STANDARDS: NOT_FOUND`, skip the standards subagent. If `KNOWLEDGEBASE: NOT_FOUND`, skip the knowledgebase subagent. Carry `NOT_FOUND` into the report's `Skipped` section. Use the resolved absolute path from the `STANDARDS:`/`KNOWLEDGEBASE:` line (not a hardcoded `~/source/` path) for all downstream references.
2. **Agent-file authority** — determine which of `CLAUDE.md` or `AGENTS.md` is the authoritative one for this repo. Never both. Rules, in order:
   - If only one of the two files exists in the root listing, that file is authoritative.
   - If both exist, read each one's opening (first ~50 lines). If one explicitly references the other as the source of truth (e.g., `AGENTS.md` says "see CLAUDE.md" or vice versa), the *referenced* file is authoritative.
   - If both exist with no cross-reference, treat `CLAUDE.md` as authoritative by default and record an entry in the report's `Suggested follow-ups` flagging the duplication so the user can collapse it.
   - If neither exists, create `CLAUDE.md` unless the project's ecosystem clearly uses `AGENTS.md` (e.g., the user has previously expressed a preference for `AGENTS.md` in this repo).

   The authoritative file is the **only** agent-doc target the dispatch step writes to. Do not write to the other one. They are never kept in verbatim sync — that's the user's explicit rule.

### Step 2: Inventory session knowledge

Before dispatching subagents, the main thread must inventory what the session contains that is worth persisting. Scan the conversation for:

- **Conventions and patterns** the user corrected you on or established (→ authoritative agent file)
- **Gotchas, footguns, environment quirks** discovered (→ authoritative agent file)
- **Workflow commands and recipes** (build, test, lint, deploy) that worked (→ authoritative agent file, and README.md if humans also run them)
- **User-facing features added or changed** (→ README.md)
- **Install, setup, or usage changes** (→ README.md)
- **Coding standards** that emerged or were enforced (→ `docs/standards/` if project-local, `~/source/standards/` if cross-project and local repo is user-owned)
- **Vendor quirks, tool internals, API behaviors, discovered failure modes** — descriptive facts about how an outside thing works (→ `~/source/knowledgebase/` if cross-project and local repo is user-owned; else the authoritative agent file). This is the descriptive counterpart to standards: route here when the nugget is "X behaves this way", not "write code this way".
- **Bugs, fixes, error/cause pairs** worth recording (→ `docs/<topic>.md` if recurrence-prone, e.g., `docs/troubleshooting.md`)
- **Open questions, deferred work, known issues** (→ `docs/<topic>.md` or README.md)

Architectural decisions are **not** auto-routed to a decisions log. Decisions docs go stale fast and the user does not maintain them — see routing heuristics for the exception.

Build a routing map: each knowledge nugget → one or more target docs. Discard nuggets that are ephemeral (single-task state, in-flight TODOs already tracked elsewhere, exploratory dead-ends).

If a piece of knowledge has no obvious home and seems durable, propose a **new doc** — but it must be **topic-scoped** (e.g., `docs/troubleshooting.md`, `docs/observability.md`), not a catch-all like `LEARNINGS.md` or `NOTES.md`. New docs are created by the relevant subagent, not the main thread.

**Duplicate-check before adding to a subagent prompt.** For each nugget, grep the target file (and any related sibling docs) for the concept's key terms. If the concept is already documented, drop the nugget from the routing map and note it under "Skipped (already documented)" in the report. Re-documenting bloats files and creates conflicting wording. Earlier-in-this-same-session edits count — if you Edit'd the agent file ten turns ago to add a convention, do not pass that convention to the subagent again.

**Output the routing map before dispatch.** Once the map is built, print it to the user in this exact form, then proceed to Step 3:

```
# Routing map (dispatching N subagents)

## <target path 1>
- <nugget summary, one line>
- <nugget summary, one line>

## <target path 2>
- <nugget summary, one line>

## Skipped (already documented)
- <nugget summary, one line> — found in <path>

## Skipped (ephemeral)
- <nugget summary, one line>
```

This gives the operator a sanity-check moment. The dispatch proceeds immediately after printing; the user may interrupt if a nugget is misrouted. Do not wait for explicit approval — the user invoked `/dump` deliberately; pausing would defeat the batch contract.

### Step 3: Dispatch parallel subagents

Spawn one subagent per documentation target that has content to write. Send all in a single message for parallelism. Use the `general-purpose` agent type unless a more specific one fits. Set `model: "sonnet"` on each Agent call.

Targets and their subagent scopes:

| Target | Scope |
|---|---|
| Authoritative agent file (`CLAUDE.md` **or** `AGENTS.md`, never both) | Agent-facing conventions, gotchas, workflows, commands. Step 1 already selected which of the two is authoritative — write only to that one. |
| `README.md` | Human-facing: what the project does, install, usage, examples, recent user-visible changes. |
| `docs/standards/` | Project-local coding standards that emerged. Topic-scoped filenames only. |
| `~/source/standards/` | Cross-project **prescriptive** patterns ("do it this way"). **Only if local repo is user-owned** (per `${CLAUDE_SKILL_DIR}/scripts/check-user-owned.sh`). The subagent must read `~/source/standards/CLAUDE.md` first — that file is the index of which topic lives in which file. Route each nugget to the correct existing file rather than guessing from the shallow pre-flight listing. Report change with TL;DR per user's global rules. **Do not run any quality checks** in `~/source/standards/` — see the out-of-repo rule below. |
| `~/source/knowledgebase/` | Cross-project **descriptive** facts ("this is how X works") — vendor quirks, tool internals, API taxonomies, discovered failure modes. **Only if local repo is user-owned** (same gate as standards). The subagent must read `~/source/knowledgebase/CLAUDE.md` first — it is the topic→file index. Route each nugget to the correct existing file rather than guessing from the shallow listing. If the nugget is hybrid (has both a rule and the mechanics behind it), the prescriptive half goes to standards and the descriptive half here, each cross-linking the other via the provided HTTPS URLs. Report change with TL;DR per user's global rules. **Do not run any quality checks** in `~/source/knowledgebase/` — see the out-of-repo rule below. |
| `docs/<topic>.md` | Troubleshooting, runbooks, design notes. Topic-scoped filenames only — never `LEARNINGS.md`, `NOTES.md`, or other catch-alls. Do not auto-create a decisions log; see routing heuristics. |

Each subagent prompt must include:

1. **Goal**: which file to update or create, and the durable purpose of that file
2. **Knowledge bundle**: the relevant nuggets from the session inventory, verbatim where possible
3. **Provenance**: paste the `LOCAL_REMOTE` URL from the **Remote URLs** pre-flight injection so the subagent can cite the source repo via its public `https://` form when the doc requires it (e.g., standards entries). For the `~/source/standards/` subagent, paste `LOCAL_REMOTE` and `STANDARDS_REMOTE`. For the `~/source/knowledgebase/` subagent, paste `LOCAL_REMOTE`, `KNOWLEDGEBASE_REMOTE`, and `STANDARDS_REMOTE` (the last so it can cross-link a hybrid nugget's prescriptive half). The subagent must never derive these URLs itself — use the injected values verbatim.
4. **Style rules**:
   - Preserve existing structure and heading conventions in the file
   - Insert new content under the right section; do not reorder unrelated content
   - No editorializing, no marketing language
   - Code blocks for commands; absolute paths where the file's convention is absolute paths
   - For the authoritative agent file (CLAUDE.md or AGENTS.md): imperative, concise, "do this / don't do that" framing
   - For README.md: user-facing, no agent-only details
   - For standards: cite rationale; pattern → why → when to apply
5. **Constraint**: write the file directly, do not stage to scratch. Return the path written **plus a 2–5 bullet summary of the content inserted** — one bullet per concept added, enough detail that the main thread can spot a misroute or a poorly-worded insertion without re-reading the file. Bullets describe what was added, not what the file is. No diff, no full quotes.
6. **Out-of-scope**: do not touch other docs; do not run commits; do not run formatters or linters unrelated to the doc itself.
7. **Out-of-repo no-checks rule**: when the target file is outside the local repo (cwd) — most notably anything under `~/source/standards/` or `~/source/knowledgebase/` — the subagent must **not** run any quality checks against the target repo. That means no `make check`, no `make lint`, no `make test`, no reachability scripts, no formatters, no link checkers, no pre-commit hooks. These commands prompt the user and break the dump's "write directly, report after" contract. The user reviews and runs checks on the standards repo in a separate session.

Sample subagent prompt skeleton:

```
You are persisting session knowledge into <ABSOLUTE_PATH>.

Purpose of this file: <one-line purpose, e.g., "agent-facing project conventions and gotchas">

Provenance (use verbatim, do not derive or convert):
- Local repo URL (HTTPS): <LOCAL_REMOTE_HTTPS from pre-flight>
- Standards repo URL (HTTPS): <STANDARDS_REMOTE_HTTPS from pre-flight, if applicable>

Knowledge to integrate (from session):
- <nugget 1>
- <nugget 2>
- ...

Rules:
- Read the file first if it exists; preserve its structure and tone.
- Before inserting a nugget, grep for the concept inside the file — if already documented, skip and note "already documented" in your return.
- Insert each new nugget under the appropriate existing section, or create a new section if none fits.
- Imperative voice for the agent file. User-facing voice for README.md.
- No marketing language, no editorializing, no praise.
- Cross-repo references must use the provided HTTPS URL verbatim — never local filesystem paths and never convert from SSH yourself.
- Do not modify content unrelated to these nuggets.
- Write the file directly.
- If the target file is outside the local repo (e.g. anywhere under `~/source/standards/` or `~/source/knowledgebase/`), do NOT run quality checks of any kind: no `make check`, no lint, no tests, no reachability scripts, no formatters. The user reviews and runs those in a separate session.

Return format (no diff, no full quotes):

```
Path: <absolute path written>
Inserted:
- <one-line description of concept added, with section name>
- <one-line description of concept added, with section name>
Skipped:
- <nugget summary> — already documented under <section>
```

Inserted bullets: 2–5 total across the file. If only one concept was added, one bullet is fine.
```

### Step 4: Aggregate and report

Before printing the report, **sanity-check each subagent's return**. Read its `Inserted:` bullets against the routing map you produced in Step 2. If a bullet describes a concept that does not match the nugget you routed to that target (misroute) or the wording looks wrong, add an entry to the report under `Verify` — do not silently accept. The user can then re-read just those files instead of all of them.

The report itself is one thing: **what changed where**. Paths only. The user reads `git diff` for content. The only narrative element is the standards TL;DR, which the user's global rules require for any `~/source/standards/` edit.

```
# Dump report

## Changed
- <path>
- <path>

## Skipped
- ~/source/standards/: <reason>
- ~/source/knowledgebase/: <reason>
- <other>: <reason>

## Standards / Knowledgebase TL;DR (cross-project repos only)
- <repo>/<file>: <one-line what and why>

## Verify (subagent bullets that look off — re-read these)
- <path>: <which bullet, what looks wrong>

## Suggested follow-ups
- <topic-scoped new doc proposed but not created>
- <nugget that didn't fit anywhere>
- <duplicate-agent-file collapse flagged in Step 1, if applicable>

Next step: run `/commit` to capture these changes before the context resets. Changes in ~/source/standards/ and ~/source/knowledgebase/ commit separately (different repos, one commit each).
```

No commentary on significance, no per-file section breakdown, no diff content. If a target had no changes, omit it.

## Routing heuristics

When a nugget plausibly belongs to multiple targets, use these rules. Wherever this section says "agent file", it means the **authoritative** file chosen in Step 1 — CLAUDE.md or AGENTS.md, exactly one.

- **Convention enforced by code review or linter** → `docs/standards/` (project) or `~/source/standards/` (cross-project, local repo user-owned only). Mention in the agent file only if agents need a reminder.
- **Workflow command discovered** → agent file (so agents reuse), and README.md (if humans also run it).
- **Gotcha specific to this repo's environment** → agent file only.
- **Gotcha that recurs across user's repos** → if local repo (cwd) is user-owned, route by shape: a *rule* ("always do X") goes under `~/source/standards/`; a *behavior fact* ("vendor/tool Y does Z") goes under `~/source/knowledgebase/`. Else agent file. To find the right file, **read the target repo's `CLAUDE.md`** — each is the index that maps topics to files. Do not guess from the shallow root listing alone.
- **Vendor quirk / tool internal / API behavior** (descriptive, recurs across repos) → `~/source/knowledgebase/` if local repo is user-owned, else agent file. Never standards — standards is prescriptive only.
- **User preference** (e.g., "always use X over Y") → agent file if repo-specific. `~/.claude/CLAUDE.md` is **out of scope** for this skill (user manages globally).
- **Architectural decision** → agent file under a "Design notes" section, **only if the user explicitly asks during the dump**. Do not auto-create a decisions log directory or scaffold. Decisions documents go stale fast and the user does not maintain them.

## Guardrails

- **Do not write to `~/.claude/`** — that is global user config, managed elsewhere.
- **Do not modify `~/source/standards/` or `~/source/knowledgebase/` when no user-owned repo is in scope.** In single-repo mode, "in scope" means the local repo (cwd). In multi-repo mode, it means any session-relevant child repo. Both capture material from the user's own work; updates from external repos would mix in unvetted content.
- **Do not commit.** This skill only writes files; commits are a separate, deliberate action by the user.
- **Do not run quality checks against out-of-repo targets.** For any file outside the local repo (cwd) — `~/source/standards/` in particular — no `make check`, lint, tests, reachability scripts, formatters, or pre-commit hooks. These prompt the user and derail the dump's batch behavior. The user runs those checks in a separate session when they review the standards changes.
- **Do not fabricate rationale.** If a nugget's "why" is unclear from the session, omit the rationale or list the gap under `Suggested follow-ups` — invented reasons rot the doc.
- **Do not duplicate** content the doc already covers. If a nugget is already documented, skip it.
- Preserve license headers and frontmatter — subagents must not strip them.
- Gender-neutral language in any prose written.
- Redact strong language per user's global rules.

## Edge cases

- **Multi-repo parent directory**: Step 0 handles this. The skill detects child repos, runs pre-flight per repo, and processes all session-relevant repos in one invocation.
- **Multi-repo with mixed ownership**: Some child repos user-owned, others not. Cross-project targets (standards, knowledgebase) are gated on whether **any** session-relevant child repo is user-owned. Per-repo targets are written regardless of ownership.
- **Multi-repo with no session activity in any child repo**: Report "no durable knowledge identified" and exit, same as empty inventory.
- **Agent-file authority cases** are handled in Step 1 (single-repo) or Step 0c (multi-repo). The bare "neither exists" case is also handled there (create `CLAUDE.md` by default).
- **No README.md exists**: create one only if there is user-facing content worth persisting. Skip creation for purely internal nuggets.
- **No `docs/` directory**: create it only if a nugget genuinely belongs in a new topic-scoped doc. Do not scaffold an empty `docs/`.
- **No `~/source/standards/CLAUDE.md`** (the standards index): the `~/source/standards/` subagent has no map to route by — skip the standards subagent and add a follow-up entry to the report flagging that the index is missing.
- **No `~/source/knowledgebase/CLAUDE.md`** (the knowledgebase index): same handling — skip the knowledgebase subagent and add a follow-up entry flagging the missing index.
- **Empty session inventory**: report "no durable knowledge identified" and exit. Do not write empty updates.
- **Conflicting nuggets** (e.g., two corrections that contradict): surface to the user via `AskUserQuestion`. Batch all conflicts into a single `AskUserQuestion` call with one question per conflict (up to the tool's max of 4 questions) — never ask one at a time. If more than 4 conflicts exist, batch the first 4 and queue the rest for a follow-up `AskUserQuestion` only if the first batch's answers don't resolve them transitively.

## Parallelism

Dispatch all subagent tasks in a single message with multiple Agent tool uses. Each subagent operates on a distinct target file or directory, so they cannot conflict. The main thread waits for all to complete before producing the report.

If only one target has content, dispatch a single subagent or do the write inline — overhead of subagent dispatch is not worth it for a single file.

## Anti-patterns

- Do not dump the entire conversation transcript into a doc. Extract durable nuggets only.
- Do not write "as discussed in our session" or any reference to the session itself. Docs are timeless artifacts.
- **Do not reference other repos by local filesystem path** (e.g., `~/source/foo/bar.sh`, `/home/user/source/foo`). Local paths are not portable and rot when directory layouts change. Any cross-repo reference must use the **public git URL** derived from the `LOCAL_REMOTE` / `STANDARDS_REMOTE` values in the **Remote URLs** pre-flight injection, converted to its `https://` browser form if the remote is SSH. Example: `git@github.com:jewzaam/jewzaam-reviews.git` → `https://github.com/jewzaam/jewzaam-reviews`. The local repo (where `/dump` was invoked) is the only repo a doc may reference by relative path.
- **Do not derive remote URLs from filesystem paths or guess them.** Use only the injected `LOCAL_REMOTE` / `STANDARDS_REMOTE` values. Models hallucinate URLs; the script is deterministic.
- Do not add timestamps, "last updated" lines, or changelog entries unless the file already has them.
- Do not reformat or restructure existing content under the guise of "while I'm here".
- **Do not create catch-all dumping grounds** (`LEARNINGS.md`, `NOTES.md`, `THOUGHTS.md`, `MISC.md`). New docs must be topic-scoped — the file name should describe its single subject (e.g., `docs/troubleshooting.md`, `docs/observability.md`, `docs/auth-flow.md`).
- Do not write to both `CLAUDE.md` and `AGENTS.md`. One is authoritative; the other is either absent or a stub. Verbatim sync is explicitly forbidden by the user.

---
> Source: [jewzaam/claude-skill-dump](https://github.com/jewzaam/claude-skill-dump) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
