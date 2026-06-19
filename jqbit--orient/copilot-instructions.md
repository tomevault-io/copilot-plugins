## orient

> Use when creating or updating a pure-Markdown orientation layer: lean ORIENT.md maps, thin README/AGENTS.md/CLAUDE.md pointers, Obsidian MOCs, and parallel read-only subtree exploration reports. No scripts or generated code required.


# Orient

## Overview

Create a lean, text-only orientation layer for a repository, documentation tree, or Obsidian-style vault so future agents do less blind discovery before they act.

The canonical file is `ORIENT.md`: a compact Markdown map of where things live, what to read first, what commands matter, and what areas are risky or off-limits. Agent-facing files such as `README.md`, `AGENTS.md`, and `CLAUDE.md` should stay thin and point readers to `ORIENT.md`.

Core principle: **map first, details on demand**. Do not dump the repo into instructions. Give routing hints, entrypoints, ownership boundaries, and search/read recipes.

Hard constraint: **do not create scripts, Python files, generated CLIs, or non-Markdown helper code for this workflow unless the user explicitly asks.** This skill is instructions-only and uses normal agent tools: search, read, write, patch, and delegation.

## When to Use

Use this skill when the user asks to:
- Set up `ORIENT.md`, `README.md` map pointers, `AGENTS.md`, `CLAUDE.md`, MOCs, repo maps, or agent-facing context files.
- Reduce agent time spent exploring a large codebase or vault.
- Create cross-agent orientation files for Claude Code, Codex, Cursor, Gemini, Hermes, OpenCode, Copilot, Pi, Factory/Droid, or generic agents (AGENTS-compatible non-Claude tools are routed through `AGENTS.md`).
- Build a "where to look for what" map for a monorepo, docs site, knowledge base, or Obsidian vault.
- Parallelize repository discovery using read-only subagents and synthesize their reports.
- Rebrand older `orient-map` or `yourient` conventions to `orient` + `ORIENT.md`.

Do not use for full implementation plans, detailed architecture docs, generated API references, or exhaustive code packing. Use external repo-map tools only if the user asks.

## Branding Rules

### Slug Source

The callable slug comes from `SKILL.md` frontmatter `name:`, not the directory name. If you rename the folder, also rename the frontmatter — and vice versa. Two skill folders with the same frontmatter `name:` make agent skill lookup ambiguous; keep one canonical copy per ecosystem.

### Names

Use this branding consistently:

- Skill / slash-command name: `orient`
- Canonical file name: `ORIENT.md`
- Nested map files: `<subtree>/ORIENT.md`
- Adapter block markers: `<!-- ORIENT:BEGIN v=1 -->` / `<!-- ORIENT:END v=1 -->`
- README block markers: `<!-- ORIENT-README:BEGIN v=1 -->` / `<!-- ORIENT-README:END v=1 -->`

Do **not** emit `yourient`, `YOURIENT`, or `orient-map` as active branding.

Reference: `references/rebrand-and-pointer-blocks.md` contains the exact full-rebrand checklist plus the managed README and adapter block shapes.

## Naming Standard

Default to this hierarchy:

```text
ORIENT.md                  # canonical portable orientation map
README.md                   # short human pointer to ORIENT.md
AGENTS.md                   # generic adapter — Codex, Cursor, Gemini, Copilot, OpenCode, Hermes, ...
CLAUDE.md                   # Claude Code adapter
<subtree>/ORIENT.md        # optional local orientation for large subsystems
<subtree>/AGENTS.md         # optional local adapter when nearest-file precedence helps
```

Use uppercase `ORIENT.md` consistently. If a project already has `orient.md`, `orientation.md`, or pre-1.5 map files, follow the migration procedure in `references/rebrand-and-pointer-blocks.md`.

## Agent Adapter Targets

The cross-tool baseline is [`AGENTS.md`](https://agents.md) — an open convention supported by a growing set of coding agents and tools, including OpenAI Codex, Cursor, GitHub Copilot coding agent, Gemini CLI, Aider, OpenCode, Windsurf, Zed, Factory, and others. Orient layers atop it: `ORIENT.md` holds the canonical map; `AGENTS.md` and the Claude-specific `CLAUDE.md` point to that map. The skill does not replace `AGENTS.md`; it gives it something concrete to link to. By default orient emits only these two adapters — `AGENTS.md` for AGENTS-compatible tools and `CLAUDE.md` for Claude Code — and does not create `GEMINI.md`, `.cursor/rules/*.mdc`, or other tool-specific files unless the user explicitly asks.

Ask the user which agents to target if they did not specify. If they do not answer, default to portable files only: `ORIENT.md` + `README.md` + `AGENTS.md`.

| Target | Project file(s) | Notes |
|---|---|---|
| Humans / contributors | `README.md` | Add a short "read `ORIENT.md` first" pointer, not the full map. |
| Generic / AGENTS-compatible | `AGENTS.md` | Widest portable adapter. Keep short. |
| OpenAI Codex | `AGENTS.md` | Supports nested `AGENTS.md`; nearest file generally wins. |
| Hermes | `AGENTS.md` | Use as project context when supported; keep canonical guidance in `ORIENT.md`. |
| Claude Code | `CLAUDE.md` | Claude Code project memory. Add a short orientation block pointing to `ORIENT.md`. |
| Gemini CLI / Antigravity | `AGENTS.md` | Gemini CLI can be configured to use `AGENTS.md`; do not emit `GEMINI.md` unless the user explicitly asks. |
| Cursor | `AGENTS.md` | Supported by the AGENTS.md convention. Keep the adapter short. |
| GitHub Copilot coding agent | `AGENTS.md` | Keep repo-scoped operational instructions here. |
| OpenCode | `AGENTS.md` | Use the generic adapter unless user has a tool-specific convention. |
| Pi / other agents | `AGENTS.md` | Use only when that agent supports project context files; global setup requires separate confirmation. |
| Factory/Droid | `AGENTS.md` | Factory supports `AGENTS.md`; do not invent `.droid/*`. |
| Obsidian/vault | `ORIENT.md`, `INDEX.md`, MOC notes | Prefer existing MOC folder if present. |

### Notes on the above

- **`CLAUDE.md` is Claude Code project memory, not a skill schema.** Treat it as a thin adapter that points to `ORIENT.md`, not as a place for canonical guidance.
- **AGENTS-compatible tools go through `AGENTS.md`.** Codex, Cursor, Gemini CLI when configured, Copilot coding agent, OpenCode, Factory, and other compatible tools can share the portable adapter, so orient does not maintain per-tool files like `GEMINI.md`. Only add a tool-specific adapter when the user explicitly requests it or the tool's docs require it.

## ORIENT.md Shape

A good `ORIENT.md` is short enough to read at session start and structured enough to route work.

````md
# ORIENT.md

## Purpose
1-3 sentences: what this repo/vault is.

## Start Here
- For task X, read Y first.
- For task A, search B, then inspect C.

## Tree Map
```text
repo/
|___src/                 # app/runtime code
|   |___api/             # HTTP routes/controllers
|   |___core/            # core domain logic
|___tests/               # tests mirroring src
|___docs/                # human docs
```

## Routing Map
| If you need to... | Start at | Then check | Avoid |
|---|---|---|---|
| Change auth | `src/auth/` | `tests/auth/`, migrations | secrets, prod configs |

## Commands
- Test: `...`
- Typecheck/lint/build: `...`

## Conventions
- Package manager, language/runtime, naming, test placement.

## No-Go / High-Risk Areas
- Generated files, vendored code, migrations, secrets, deployment config.

## Subsystem Notes
- Link to nested `ORIENT.md` files or MOCs.

## Maintenance
- Update this file when directories, entrypoints, or commands change.
- `Last reviewed: YYYY-MM-DD`
````

For a complete worked example, see `references/example-ORIENT.md`.

## Thin Adapter Shape

Use managed blocks so existing user content survives.

```md
<!-- ORIENT:BEGIN v=1 -->
# Orient Map

This project uses `ORIENT.md` as the canonical repo/vault map.

Before broad file search or structural edits:
1. Read `ORIENT.md`.
2. Check for a nearer nested `ORIENT.md` or `AGENTS.md` in the target subtree.
3. Use the routing map there to choose files and commands.
4. Keep this adapter thin; update `ORIENT.md` when structure changes.
<!-- ORIENT:END v=1 -->
```

Use this block shape in `AGENTS.md`, `CLAUDE.md`, and similar adapter files.

### Placement Rule

These are the high-level rules; the exact byte-level steps live in `## Managed Block Algorithm`.

- Empty file or file containing only whitespace: write the block at the top.
- File whose first non-blank line is an H1 (`# `): insert the block immediately after that H1, separated by one blank line above and below.
- Any other existing file with hand-written content: append the block at the end of the file, with one blank line above and a trailing newline at file end.
- Never split or interleave existing user content with the block.
- If a managed block already exists, replace it in place — do not move it.

## README Pointer Shape

`README.md` should stay human-friendly. Add a short pointer block, not the whole map.

```md
<!-- ORIENT-README:BEGIN v=1 -->
## Repo Map

For structure, routing, and agent guidance, read [`ORIENT.md`](./ORIENT.md).

Agents and contributors should consult `ORIENT.md` before broad repo exploration or large refactors.
<!-- ORIENT-README:END v=1 -->
```

If the map lives outside the same directory, change the link to the correct relative path.

## Managed Block Algorithm

The exact byte-level rule for inserting, updating, and de-duplicating managed blocks. Re-running the algorithm on an unchanged file must produce a byte-identical result (idempotency).

For each target file (e.g., `AGENTS.md`, `CLAUDE.md`, `README.md`):

1. **Preflight.** Read the file (or treat as empty if it does not exist and the user asked to create it). Before parsing, capture three byte-level properties so they can be preserved on write:
   - **BOM:** if the file starts with a UTF-8 byte-order-mark (`0xEF 0xBB 0xBF`), preserve it.
   - **Line ending:** if any line ends in `\r\n`, treat the file as CRLF; otherwise LF. Empty files default to LF. Any block content written must match the file's convention.
   - **Trailing newline:** record whether the file ends with a newline; preserve that on write.

2. **Locate** all opening anchors — `<!-- ORIENT:BEGIN` for adapter files, `<!-- ORIENT-README:BEGIN` for `README.md`. For each opener, find its closing anchor (`ORIENT:END` / `ORIENT-README:END`). Capture the optional `v=N` attribute per occurrence; a missing attribute means `v=0`.

3. **Malformed input.** If any opener lacks a matching closer (or vice versa), or if openers/closers interleave across families, abort the write with a clear error and leave the file untouched. Do not attempt a guess or silent repair.

4. **Decide based on the count of well-formed blocks for this anchor family:**

   - **Zero blocks found** — place a new block, picking the location by file shape (matches `## Thin Adapter Shape § Placement Rule`):
     - File is empty or whitespace-only: write the block at the top.
     - File's first non-blank line is an H1 (`# `): insert the block immediately after that line, separated by one blank line above and one blank line below.
     - Otherwise: append at end. Ensure exactly one blank line above the block and a trailing newline at file end.

   - **Exactly one block found:** replace from the `BEGIN` line through the matching `END` line with the new block at the current schema version. Leave surrounding lines unchanged. Do not adjust blank-line spacing outside the block.

   - **Two or more blocks found:** keep the first; delete each subsequent `BEGIN`-through-matching-`END` range (including any blank line immediately following the removed block); then upgrade the surviving block per the single-block rule.

5. **Anchor matching** is by family name only (`ORIENT` or `ORIENT-README`), case-sensitive on the family, version-agnostic. Pre-1.5 blocks whose anchor family is the same name with a leading `Y` count as the same family and are rewritten to `ORIENT` / `ORIENT-README` at the current schema version. `v=0` (unversioned) and `v=N` blocks are interchangeable on detection.

6. **Forward-compatibility.** If a found block carries `v=M` where `M` is greater than this skill's current schema version, leave it untouched and report it — do not downgrade.

7. **Write.** Emit the result using the captured BOM, line-ending convention, and trailing-newline behavior. Do not normalize, re-encode, or strip whitespace outside the block being inserted or replaced.

8. **Idempotency check.** Re-running step 2 against the freshly written file must return exactly one block at the current schema version. If not, the write produced unexpected duplication and must be reverted.

This algorithm applies symmetrically to the README pointer block, with `ORIENT-README` as the anchor family.

## Relative Link Rule

Always point each file to the **nearest relevant** `ORIENT.md` using a relative path from that file's location.

Examples:
- Root `README.md` -> `./ORIENT.md`
- `docs/README.md` -> `../ORIENT.md` or `./ORIENT.md` depending on where the map lives
- `packages/api/AGENTS.md` -> `./ORIENT.md` if the package has its own local map
- `packages/api/CLAUDE.md` -> `./ORIENT.md` or `../ORIENT.md` based on actual placement

Do not hardcode root-level links when a nearer nested `ORIENT.md` exists.

## Commit Stance

`ORIENT.md` is a committed artifact: humans read it, agents read it, and it should evolve with the repo through normal code review. Treat it like any other source-controlled doc.

- **Do commit:** `ORIENT.md`, nested `<subtree>/ORIENT.md` files, managed pointer blocks in `README.md`, `AGENTS.md`, `CLAUDE.md`.
- **Do not** treat `ORIENT.md` as a generated artifact, do not auto-write it from CI, and do not add it to `.gitignore`.
- **Do** include `ORIENT.md` and adapter file changes in the same PR as the structural change they describe, so the map never lags behind reality.

## Workflow

### 1. Pre-flight gate: scope and safety

Before writing anything:
1. Identify root path and whether it is a git repo, docs tree, or vault.
2. Ask one concise question if target agents are unknown: "Which agents should I emit adapters for?"
3. Discover existing orientation/context files: `ORIENT.md`, `README.md`, `AGENTS.md`, `CLAUDE.md`, MOC/INDEX folders, plus any legacy `orient-map` or obsolete map-filename variants.
4. Check git status when inside a git repo.
5. Do not overwrite user-written context. Patch managed blocks or add new lean files.
6. Exclude vendor/generated/cache directories from exploration: `.git`, `node_modules`, `.venv`, `dist`, `build`, `coverage`, `.next`, `.turbo`, `target`, `vendor`, binary assets, lockfile-only paths.

### 2. Broad scan, no scripts

Do one cheap structural scan using normal file/search tools only.

Look for:
- top-level folders
- existing context files
- README/index/MOC files
- package/build/test manifests
- docs/config/test directories
- obvious domains such as frontend, backend, infra, packages, apps, docs, vault areas

Do not read every source file in the orchestrator. The orchestrator routes; subagents inspect.

For ecosystem-specific entrypoint conventions (where commands live, which manifest files matter, what "src" looks like for the language at hand), consult `references/by-ecosystem.md`.

### 3. Decide subagent fan-out

Pick the smallest fan-out that covers the repo. More agents is not automatically better.

**Measure the repo first.** Inside git: `git ls-files | wc -l`. Outside git: `find . -type f -not -path './.git/*' -not -path './node_modules/*' -not -path './.venv/*' | wc -l`.

| File count | Explorer count | Rule |
|---:|---:|---|
| <300 | 0–1 | Orchestrator can inspect directly. |
| 300–2,000 | 2–4 | One explorer per coherent top-level area. |
| 2,000–10,000 | 4–8 | Shard by domain; respect harness concurrency limit. |
| >10,000 | shard by top-level domain, cap 8 concurrent | Batch sequentially if the harness caps parallelism lower. |
| Obsidian/PARA vault | 2–5 | Projects, Areas, Resources, Permanent/MOCs, Meta. |

Budget per explorer report: keep each return under ~25k tokens. The orchestrator synthesizes; subagents do not synthesize across each other.

Shard by feature/domain, not arbitrary equal file counts. Good shards: `frontend`, `backend`, `docs`, `infra`, `packages/foo`, `vault/Projects`, `vault/Areas`.

Avoid launching separate explorers for generated/vendor/test-only trees unless tests are a major architectural area.

Honor the harness's tool concurrency limit (most cap parallel agents at 4–8). If the repo demands more shards than the cap, batch sequentially rather than queuing too many at once.

### 4. Parallel subtree explorers

Launch read-only subagents. Each explorer gets one path/domain and returns a fixed short Markdown report. Do not let explorers edit files.

Explorer prompt template:

```text
Explore this subtree for a ORIENT map. Read only enough files to answer.

Root: <repo root>
Subtree/domain: <path or domain>
Goal: summarize where future agents should look first, not document everything.

Return exactly:
- Path/domain:
- Purpose:
- Important entrypoints:
- Key files/directories:
- Common tasks -> where to start:
- Commands/tests relevant to this area:
- Conventions/patterns:
- No-go/generated/risky areas:
- Suggested ORIENT.md bullets:
- Confidence and unknowns:

Keep the report under 25k tokens. Do not paste large code. Do not modify files.
```

### 5. Reduce and draft

The orchestrator combines explorer reports into:
- root `ORIENT.md`
- optional nested `<subtree>/ORIENT.md` for large subsystems
- thin adapter files for selected agents
- short `README.md` pointers for humans
- optional MOC/index notes for vaults

Use revision gates:
- If explorer reports conflict, inspect the exact files or dispatch a focused verifier.
- If the generated draft exceeds roughly 150-250 lines, compress it and move detail to nested files.
- If any command is guessed, label it unknown or verify from package/config files.

### 6. Emit files manually with Markdown/file tools

Write Markdown files directly. Do not generate them with scripts.

Rules:
- `ORIENT.md` is canonical.
- `README.md` gets a short pointer block.
- `AGENTS.md` (AGENTS-compatible non-Claude tools) and `CLAUDE.md` (Claude Code) are thin adapters.
- Use managed blocks in existing files (see `## Managed Block Algorithm`).
- Preserve all content outside managed blocks.
- Use the correct relative link/path to the applicable `ORIENT.md`.
- If `README.md` does not exist, only create one when the user explicitly wants that.
- If the user explicitly asks for exact target files, write only those files.

### 7. Reference propagation pass

After drafting `ORIENT.md`, update surrounding files that should point to it.

Minimum pass:
1. Update or append the managed README pointer block.
2. Update or append managed blocks in `AGENTS.md` and `CLAUDE.md` when those files exist or were requested.
3. If nested subsystem maps exist, point nested adapters to the nearest nested `ORIENT.md`, not always the root file.
4. Preserve all non-managed content verbatim.

### 8. Verify

After writing:
- Read each emitted file back.
- Confirm README and adapter files point to the correct `ORIENT.md` path.
- Confirm existing content outside managed blocks remained intact.
- Confirm tree paths in `ORIENT.md` exist or are marked optional/unknown.
- In git repos, show `git diff --stat` and a concise changed-file list.

## Drift Check

Orientation files rot when the repo changes faster than the map. Run this drift check periodically and after large structural changes.

1. **Tree-map paths exist.** Extract each path from the `## Tree Map` section. For each, run `[ -e "$p" ] || echo "MISSING: $p"`. Any `MISSING` lines block a successful drift check.
2. **Commands still parse.** For each command in the `## Commands` section, run a dry-run or `--help` form where available (e.g., `npm test --help`, `cargo test --help`). Note any command that no longer exists.
3. **Top-level diff.** Compare the Tree Map's top-level dirs against `git ls-tree -d --name-only HEAD | head -50` (or `ls -d */`). Note any directories present on disk but absent from the map.
4. **No-go / commands sanity.** Re-confirm the `No-Go / High-Risk Areas` section still names real paths.
5. **Stamp.** Update or append `Last reviewed: YYYY-MM-DD` at the bottom of `ORIENT.md`.

If drift is significant, re-run the broad scan from §2 of the workflow and let subagents refresh affected sections.

## Smoke Test

After installing or updating `orient`, confirm the skill is callable in each agent ecosystem the user targets.

Universal checks:
- `SKILL.md` frontmatter parses (one `---` opening and closing line; `name: orient` and a `version:` are present).
- Skill directory contains `SKILL.md` and `references/` with at least `rebrand-and-pointer-blocks.md`.

Per-agent checks:
- **Claude Code:** `/orient` shows up in the skill list (e.g., session skill enumeration); the description matches the current frontmatter.
- **Hermes:** `skill_view('orient')` returns the body.
- **Codex:** restart Codex, then verify `orient` appears in the skill list.
- **Other agents:** confirm the skill enumerates in that agent's official skill/package UI or command. If the agent only supports project context files, run orient from a supported host and commit the generated Markdown instead.

If a check fails: verify the skill folder lives under that agent's skills directory and that the frontmatter `name:` is exactly `orient`.

## Obsidian / Vault Variant

For vaults, treat `ORIENT.md` as the agent start page and MOCs as domain maps.

Recommended files:

```text
ORIENT.md
00 - Maps of Content/
|___README.md or INDEX.md
|___Projects MOC.md
|___Areas MOC.md
|___People MOC.md
|___Sources MOC.md
99 - Meta/
|___agent-notes.md        # optional operational notes
```

Do not reorganize notes unless asked. Start by mapping existing folders and canonical notes. Use backlinks/search hints rather than moving files.

If the vault uses Obsidian plugins such as Dataview or Templater, mention them in `ORIENT.md` so agents know they can query rather than crawl.

## Integration with Repo-Map Tools

This skill creates durable orientation docs. It does not replace structural tools.

- Aider repo-map: compact symbol/signature context during coding.
- Repomix: whole-repo AI-friendly packed context when explicitly requested.
- CodeGraph/MCP graph tools: live symbol/call/impact queries when already installed.
- `ORIENT.md`: stable human+agent routing, conventions, and "where to start."

If one of these tools exists in the project, mention it in `ORIENT.md` and tell agents to use it before raw search/read loops.

## Common Pitfalls

1. **Creating scripts or helper programs.** This skill is Markdown-first and instructions-only unless the user explicitly asks otherwise.
2. **Writing a giant AGENTS.md or README section.** Keep pointers thin. Put canonical routing in `ORIENT.md`.
3. **Forgetting the README pointer.** Humans should be routed to `ORIENT.md`, not forced to discover it manually.
4. **Over-sharding.** Ten explorers for a small repo creates noise and merge work. Use fewer, better shards.
5. **Letting subagents edit.** Explorers are read-only; only the orchestrator writes final docs.
6. **Guessing commands.** Verify commands from manifests/configs or mark unknown.
7. **Breaking existing context files.** Patch managed blocks; never overwrite hand-written guidance without explicit approval.
8. **Using the wrong link path.** Compute the relative path from each adapter/README file to the right `ORIENT.md`.
9. **Inventing tool-specific adapter files.** Route AGENTS-compatible non-Claude tools through `AGENTS.md` and Claude Code through `CLAUDE.md`; do not create `GEMINI.md`, `.cursor/rules/*.mdc`, or other per-tool files unless the user explicitly requests them.
10. **Mirroring only `SKILL.md`.** If the skill gains `references/`, `templates/`, or other support files, sync those across agent mirrors too — otherwise downstream copies drift and lose helper material.
11. **Leaving old branding behind.** Replace `ORIENT.md` and `orient-map` references when doing a full rebrand (see `references/rebrand-and-pointer-blocks.md`).
12. **Skipping the version tag.** New blocks must use `v=1`; the algorithm in `## Managed Block Algorithm` upgrades unversioned legacy blocks automatically.
13. **Stale maps.** Run `## Drift Check` after large structural changes and add a `Last reviewed:` stamp.

## Verification Checklist

- [ ] No scripts, code files, or non-Markdown helper artifacts created.
- [ ] Target agents confirmed or portable default used.
- [ ] Existing context files discovered before writes.
- [ ] Broad scan completed with normal file/search tools before fan-out.
- [ ] Subagent shards are by coherent domain/subtree, sized to the file-count buckets in §3.
- [ ] Explorer reports are concise (under ~25k tokens) and read-only.
- [ ] `ORIENT.md` is lean, routing-focused, and includes an ASCII tree.
- [ ] `README.md` contains a short pointer block to `ORIENT.md` when requested or present.
- [ ] Adapter files are thin, use managed blocks at `v=1`, and point to the correct `ORIENT.md`.
- [ ] Existing file content preserved outside managed blocks.
- [ ] Commands and paths verified or marked unknown.
- [ ] Final changed-file list reported.
- [ ] Smoke test passes for each targeted agent ecosystem.

## One-Shot User Prompt

If the user wants a simple command to another agent, give them this:

```text
Create and maintain a lean, text-only ORIENT layer for this repo/vault. Use ORIENT.md as the canonical Markdown map, add or update a short README.md pointer plus thin AGENTS.md (AGENTS-compatible agents) and CLAUDE.md (Claude Code) adapters, do one broad structural scan using normal file/search tools only, launch read-only parallel subtree explorers for major domains, synthesize their Markdown reports into concise ASCII-tree routing docs, preserve existing content with managed v=1 blocks, and do not create scripts or non-Markdown helper files.
```

---
> Source: [jqbit/orient](https://github.com/jqbit/orient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
