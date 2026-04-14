## llm-finetuned-supportbot

> - Project: \${PROJECT\_TAG}

# Agents Instructions (Ordered)

- Project: \${PROJECT\_TAG}
- Purpose: Canonical instruction list. Do not append logs here.

---

## Instruction Loading Model (Simplified)

- **Baseline:** This file (**GEMINI.md**) is the canonical baseline for all assistant behavior.
- **Auto-discovered extensions (no tags):** At **session start**, the assistant **scans** the repository’s `instructions/` directory and proposes any instruction files it finds for inclusion. **User confirmation is required** before they affect behavior.

  - Valid instruction files: `instructions/**/*.md` or `instructions/**/*.txt` (recursive).
  - Ignored by default: files or folders prefixed with `_` (e.g., `instructions/_drafts/`), and files with `.archive.` in the name.
  - **Order:** By default, files are ordered **lexicographically** by path. The user may **reorder** during confirmation; rightmost/last-loaded wins on ties within the same scope.
- **Context-only includes remain available** via `@{...}` (see below). These do **not** alter behavior.

### Startup confirmation flow

1. **Discover** candidate instruction files under `instructions/`.
2. **Summarize** each candidate (title = first `# H1` if present, else filename) and show a checklist.
3. **Ask for confirmation**:

   - Options: **Apply all**, **Apply none**, or **Select subset & order**.
   - If the user does not choose, default to **Apply none** (baseline only) and proceed.
4. **Record** the approved set and final order in the session preamble and in `DocFetchReport.approved_instructions`.

### Conflict resolution

- **Baseline vs. extension:** The **more specific extension** (approved instruction file) wins within its scope; otherwise the **baseline** holds.
- **Extension vs. extension:** When two approved files conflict in the **same scope**, the **later-loaded** file wins (i.e., by the final order confirmed by the user; lexicographic if unchanged).

---

## Tagging Syntax (context only)

**Context tag — **`@{file}`** (no behavior change)**

- Purpose: Include files in retrieval/context only (e.g., for grounding, examples, specs). *Does not* add rules.
- Syntax: `@{path/to/file.md}` (absolute or relative to repo root). Globs allowed, e.g., `@{docs/*.md}`.
- Order: Irrelevant for behavior; these are not layered.

---

## Resolution & Loading

1. **Discovery** (session start): scan `instructions/` for candidates, ignoring `_`-prefixed paths and `.archive.` files.
2. **Confirmation:** present the list; user selects and orders; default = none.
3. **Loading model:**

   - **Baseline** = GEMINI.md
   - **Extensions** = user-approved files from `instructions/` (loaded left → right in the confirmed order)
   - **Conflict rule** = The more specific **extension** wins within its scope; otherwise baseline holds.
4. **Context files:** Any `@{...}` references are added to retrieval only and are reported under `DocFetchReport.context_files`.

---

## MCP Servers & Aliases (runtime)

- **mem0** — free-form checkpoints & notes (alias: `mem0`)
- **Knowledge Graph** — structured entities/relations/observations (alias: `memory`)
- **gitmcp** — remote docs for any GitHub repo via `npx mcp-remote` (alias: `gitmcp`)
- **sourcebot** — polyglot docs retrieval/search across official docs, guides, and APIs (alias: `sourcebot`)
- **docfork** — ephemeral docs workspace for snapshotting, annotating, and composing fetched docs (alias: `docfork`)

  - Primary ops: `create_entities`, `create_relations`, `add_observations`, `read_graph`, `search_nodes`
  - Namespace for memory graph: `project:${PROJECT_TAG}`

---

## Example Usage

- **Baseline only** (no extra behavior files applied):

  - `codex-cli run "Summarize the repo structure"`
- **Context only (no behavior change):**

  - `codex-cli run "Analyze these examples @{examples/*.md}"`
- **Add/modify behavior for this run:**

  - Place/modify files under `instructions/`, then start a new session; confirm which to apply and in what order when prompted.

---

## Execution Flow (Docs Integration)

- **Preflight (§A)** must include both:

  - **Context files** discovered via `@{...}` (if any), and
  - **Approved instruction files** from `instructions/` (the final confirmed set and order).
- **Compose instructions**

  1. Apply GEMINI.md (baseline)
  2. Layer the **approved **`instructions/`** files** (left → right, final order)
  3. Resolve conflicts per rules above
- Proceed only when `DocFetchReport.status == "OK"` (Decision Gate §B).

---

## Failure Handling

- If any **approved instruction file** cannot be loaded/resolved at runtime:

  - **Do not finalize.** Return a minimal **“Docs Missing”** plan listing missing paths and suggested fix.
- If a **context** file referenced via `@{...}` cannot be resolved:

  - Continue, but record under `DocFetchReport.gaps.context_missing[]` with the attempted providers; suggest a fix in the plan section.

---

## DocFetchReport Addendum

When discovery/confirmation is used, add:

```json
{
  "DocFetchReport": {
    "approved_instructions": [
      {"path": "instructions/prd_generator.md", "loaded": true, "order": 1},
      {"path": "instructions/security/guardrails.md", "loaded": true, "order": 2}
    ],
    "context_files": [
      {"path": "docs/changelog.md", "loaded": true}
    ],
    "memory_ops": [
      {"tool": "memory", "op": "create_entities|create_relations|add_observations|read_graph|search_nodes", "time_utc": "<ISO8601>", "scope": "project:${PROJECT_TAG}"}
    ],
    "proposed_mcp_servers": [
      {"name": "<lib> Docs", "url": "https://gitmcp.io/{OWNER}/{REPO}", "source": "techstack-bootstrap", "status": "proposed"}
    ],
    "owner_repo_resolution": [
      {"library": "<lib>", "candidates": ["owner1/repo1", "owner2/repo2"], "selected": "owner/repo", "confidence": 0.92, "method": "registry|package_index|docs_link|search"}
    ],
    "server_inventory": [
      {"name": "fastapi_docs", "url": "https://gitmcp.io/tiangolo/fastapi", "source": "project-local|user|generated", "writable": true}
    ],
    "server_diff": {
      "missing": ["fastapi_docs", "httpx_docs"],
      "extra": ["legacy_docs_server"]
    },
    "server_actions": [
      {"action": "add", "name": "httpx_docs", "target": "project-local", "accepted": true}
    ],
    "techstack_toml": "<fenced TOML emitted in §A.2>",
    "last_bootstrap_signature": {
      "tech_stack_hash": "<sha256>",
      "file_hash": "<sha256>",
      "time_utc": "<ISO8601>"
    },
    "techstack_bootstrap_suppressed": true
  }
}
```

---

## A) Preflight: Latest Docs Requirement (**MUST**, Blocking)

**Goal:** Ensure the assistant retrieves and considers the *latest relevant docs* before planning, acting, or finalizing.

**Primary/Fallback Order (consolidated):**

1. **sourcebot** (primary) **or** **docfork** (if configured for direct retrieval)
2. **contex7-mcp** (fallback if the primary fails)
3. **gitmcp** (last-resort fallback if both above fail)

**What to do:**

- For every task that could touch code, configuration, APIs, tooling, or libraries:

  - Call **sourcebot** **or** **docfork** to fetch the latest documentation or guides.
  - If the chosen primary call **fails**, immediately retry with the other primary (**docfork** ⇄ **sourcebot**); if that also **fails**, retry with **contex7-mcp**; if that also **fails**, retry with **gitmcp**.
- Each successful call **MUST** capture:

  - Tool name, query/topic, retrieval timestamp (UTC), and source refs/URLs (or repo refs/commits).
- Scope:

  - Fetch docs for each **area to be touched** (framework, library, CLI, infra, etc.).
  - Prefer focused topics (e.g., "exception handlers", "lifespan", "retry policy", "sqlite schema").

**Failure handling:**

- If **all** providers fail for a required area, **do not finalize**. Return a minimal plan that includes:

  - The attempted providers and errors
  - The specific topics/areas still uncovered
  - A safe, read-only analysis and suggested next checks (or user confirmation).

**Proof-of-Work Artifact (required):**

- Produce and attach a `DocFetchReport` (JSON) with `status`, `tools_called[]`, `sources[]`, `coverage`, `key_guidance[]`, `gaps`, and `informed_changes[]`.
- Example schema:

```json
{
  "DocFetchReport": {
    "status": "OK | PARTIAL | FAILED",
    "tools_called": [
      {"name": "sourcebot|docfork|contex7-mcp|gitmcp", "time_utc": "<ISO8601>", "query": "<topic/ids>"}
    ],
    "sources": [
      {"url_or_ref": "<doc url or repo ref>", "kind": "api|guide|code|spec", "commit_or_version": "<hash|tag|n/a>"}
    ],
    "coverage": "Which parts of the task these sources cover.",
    "key_guidance": [
      "Short bullets of the exact rules/APIs that constrain the change."
    ],
    "gaps": "Anything still unknown.",
    "informed_changes": ["files or commands to be touched, mapped to guidance"]
  }
}
```

**Override Path (explicit, logged):**

- Allowed only for outages/ambiguous scope/timeboxed spikes. Must include:

```json
{
  "Override": {
    "reason": "server_down|ambiguous_scope|timeboxed_spike",
    "risk_mitigation": ["read-only analysis", "scoped PoC", "user confirmation required"],
    "expires_after": "1 action or 30m",
    "requested_by": "system|user"
  }
}
```

---

## A.1) Tech & Language Identification (Pre-Requirement)

- Before running Preflight (§A), the assistant must determine both:

  1. The **primary language(s)** used in the project (e.g., Python, TypeScript, Pytest, Bash).
  2. The **current project’s tech stack** (frameworks, libraries, infra, tools).

- Sources to infer language/stack:

  - Project tags (`${PROJECT_TAG}`), mem0 checkpoints, prior completion records.
  - Files present in repo (e.g., `pyproject.toml`, `requirements.txt`, `package.json`, `tsconfig.json`, `Dockerfile`, CI configs).
  - File extensions in repo (`.py`, `.ts`, `.js`, `.sh`, `.sql`, etc.).
  - User/task context (explicit mentions of frameworks, CLIs, infra).

- **Repo mapping requirement (NEW):** Resolve the **canonical GitHub OWNER/REPO** for each detected library/tool whenever feasible.

  - **Resolution order:**

    1. **Registry mapping** (maintained lookup table for common libs).
    2. **Package index metadata** (e.g., PyPI `project_urls` → `Source`/`Homepage`; npm `package.json` → `repository.url`).
    3. **Official docs → GitHub link** discovery.
    4. **Targeted search** (as a last resort) with guardrails below.
  - **Guardrails:** Prefer official orgs; require name similarity and recent activity; avoid forks and mirrors unless explicitly chosen.
  - Record outcomes in `DocFetchReport.owner_repo_resolution[]` with candidates, selected repo, method, and confidence score.

- Doc retrieval (§A) **must cover each identified language and stack element** that will be touched by the task.

- Record both in the `DocFetchReport`:

```json
"tech_stack": ["fastapi", "httpx", "sqlite3", "pytest-asyncio"],
"languages": ["python", "pytest"]
```

---

## A) Preflight: Latest Docs Requirement (**MUST**, Blocking)

**Goal:** Ensure the assistant retrieves and considers the *latest relevant docs* before planning, acting, or finalizing.

**Primary/Fallback Order (consolidated):**

1. **sourcebot** (primary) **or** **docfork** (if configured for direct retrieval)
2. **contex7-mcp** (fallback if the primary fails)
3. **gitmcp** (last-resort fallback if both above fail)

**What to do:**

- For every task that could touch code, configuration, APIs, tooling, or libraries:

  - Call **sourcebot** **or** **docfork** to fetch the latest documentation or guides.
  - If the chosen primary call **fails**, immediately retry with the other primary (**docfork** ⇄ **sourcebot**); if that also **fails**, retry with **contex7-mcp**; if that also **fails**, retry with **gitmcp**.
- Each successful call **MUST** capture:

  - Tool name, query/topic, retrieval timestamp (UTC), and source refs/URLs (or repo refs/commits).
- Scope:

  - Fetch docs for each **area to be touched** (framework, library, CLI, infra, etc.).
  - Prefer focused topics (e.g., "exception handlers", "lifespan", "retry policy", "sqlite schema").

**Failure handling:**

- If **all** providers fail for a required area, **do not finalize**. Return a minimal **“Docs Missing”** plan listing missing paths and suggested fix.

**Proof-of-Work Artifact (required):**

- Produce and attach a `DocFetchReport` (JSON) with `status`, `tools_called[]`, `sources[]`, `coverage`, `key_guidance[]`, `gaps`, and `informed_changes[]`.

---

## A.1) Tech & Language Identification (Pre-Requirement)

- Before running Preflight (§A), the assistant must determine both:

  1. The **primary language(s)** used in the project (e.g., Python, TypeScript, Pytest, Bash).
  2. The **current project’s tech stack** (frameworks, libraries, infra, tools).

- Sources to infer language/stack:

  - Project tags (`${PROJECT_TAG}`), mem0 checkpoints, prior completion records.
  - Files present in repo (e.g., `pyproject.toml`, `requirements.txt`, `package.json`, `tsconfig.json`, `Dockerfile`, CI configs).
  - File extensions in repo (`.py`, `.ts`, `.js`, `.sh`, `.sql`, etc.).
  - User/task context (explicit mentions of frameworks, CLIs, infra).

- **Repo mapping requirement (NEW):** Resolve the **canonical GitHub OWNER/REPO** for each detected library/tool whenever feasible.

  - **Resolution order:**

    1. **Registry mapping** (maintained lookup table for common libs).
    2. **Package index metadata** (e.g., PyPI `project_urls` → `Source`/`Homepage`; npm `package.json` → `repository.url`).
    3. **Official docs → GitHub link** discovery.
    4. **Targeted search** (as a last resort) with guardrails below.
  - **Guardrails:** Prefer official orgs; require name similarity and recent activity; avoid forks and mirrors unless explicitly chosen.
  - Record outcomes in `DocFetchReport.owner_repo_resolution[]` with candidates, selected repo, method, and confidence score.

- Doc retrieval (§A) **must cover each identified language and stack element** that will be touched by the task.

- Record both in the `DocFetchReport`:

```json
"tech_stack": ["fastapi", "httpx", "sqlite3", "pytest-asyncio"],
"languages": ["python", "pytest"]
```

---

## A.2) **One-time Tech Stack MCP Bootstrap (TOML emitter)**

**Goal:** On the **first session for a new project**, generate a **ready-to-paste TOML** block that wires detected tech-stack docs into **GitMCP** servers for use by codex-cli (or any MCP-compatible client).

**When:** Run **after** §A.1 (tech stack identified & repo mapping attempted) and **before** §A (Preflight) proceeds. This step is **idempotent** and **one-time per project**.

**Idempotency / Re-run Control (UPDATED):**

- Generate only if **both** are true:

  - No file exists at `config/mcp_servers.generated.toml`, **and**
  - `DocFetchReport.techstack_toml` is empty for this project.
- If `config/mcp_servers.generated.toml` **exists**, set `ctx.techstack_bootstrap_suppressed = true` and **skip both** the emitter **and** reconciliation checks in future sessions unless one of the **re-run triggers** below applies.
- **Re-run triggers:**

  - The user explicitly asks (e.g., *“recheck MCP servers”* or *“re-run Tech Stack MCP Bootstrap”*), **or**
  - The file was deleted/renamed or its content hash differs from `DocFetchReport.last_bootstrap_signature.file_hash`, **or**
  - A **new stack element** is detected since the last run (hash mismatch on `DocFetchReport.last_bootstrap_signature.tech_stack_hash`).

**Detection → Proposal:**

1. Take the stack from §A.1 (`DocFetchReport.tech_stack`).
2. For each library/tool `X`, propose an MCP server entry named `"X Docs"` using the **GitMCP** URL template `https://gitmcp.io/{OWNER}/{REPO}`.
3. **OWNER/REPO Auto‑Resolution:** Attempt to **fill** `{OWNER}/{REPO}` automatically using the repo mapping from §A.1. If confidence ≥ **0.85**, insert the resolved value; otherwise keep placeholders and add a TOML **comment** with the top 1–2 candidates.

**Generated artifact (fenced TOML + file):**

- Emit a fenced **TOML** block into the chat and write an identical file at:

  - `config/mcp_servers.generated.toml` (created; **do not overwrite** without explicit user approval).

**Acceptance Flow (updated):**

- After emission, prompt the user: **“Insert into codex-cli `config.toml` now?”**

  - If **yes**: append the generated block under `[mcp_servers]` (or merge keys if already present).
  - If **no**: leave `config/mcp_servers.generated.toml` for manual review.
  - If any OWNER/REPO values were auto‑filled, show a **diff preview** and allow quick **Accept**/**Override** for each.

**MCP Servers Reconciliation (non-blocking, on re-run or when config is present):**

- **Discovery:** Attempt to enumerate currently configured MCP servers from the active client configuration (project-local config first; fall back to user-level if readable). If the config cannot be read, record a gap and **skip silently**.
- **Diff:** Compare `current_servers` with `proposed_mcp_servers` derived from the detected tech stack.
- **Proposal:** Present a checklist of **missing** servers (name → URL). **No changes** are made without explicit user confirmation.
- **Write target:** Prefer updating the **project-local** config. Only modify user-level config if the user opts in.
- **Reporting:** Record inventory, diff, and any accepted additions in `DocFetchReport.server_inventory`, `server_diff`, and `server_actions`.

**Reporting (updated):**

- Add `proposed_mcp_servers[]`, `owner_repo_resolution[]`, `techstack_toml`, `last_bootstrap_signature`, and `techstack_bootstrap_suppressed` to `DocFetchReport` with exact names/URLs, confidence scores, and hashes.

---

## B) Decision Gate: No Finalize Without Proof (**MUST**)

- The assistant **MUST NOT**: finalize, apply diffs, modify files, or deliver a definitive answer **unless** `DocFetchReport.status == "OK"`.
- The planner/executor must verify `ctx.docs_ready == true` (set when at least one successful docs call exists **per required area**).
- If `status != OK` or `ctx.docs_ready != true`:

  - Stop. Return a **Docs Missing** message that lists the exact MCP calls and topics to run.

---

## 0) Debugging

- **Use consolidated docs-first flow** before touching any files or finalizing:

  - Try **sourcebot** → **docfork** (if configured) → **contex7-mcp** → **gitmcp**.
  - Record results in `DocFetchReport`.

## 1) Startup memory bootstrap (mem0 + memory)

- On chat/session start: initialize **mem0** and the **memory**.
- Retrieve (project-scoped):

  - **mem0** → latest `memory_checkpoints` and recent task completions.
  - **memory** → ensure a graph namespace exists for this project and load prior nodes/edges.

    - **Server alias**: `memory` (e.g., Smithery "Memory Server" such as `@jlia0/servers`).
    - **Bootstrap ops** (idempotent):

      - `create_entities` (or `upsert_entities`) for: `project:${PROJECT_TAG}`.
      - `create_relations` to link existing tasks/files if present.
      - `read_graph` / `search_nodes` to hydrate working context.
- Read/write rules:

  - Prefer **mem0** for free-form notes and checkpoints.
  - Prefer **memory** for **structured** facts/relations (entities, edges, observations).
  - If memory is unavailable, continue with mem0 and record `memory_unavailable: true` in the session preamble.

## 2) On task completion (status → done)

- Write a concise completion memory to mem0 including:

  - `task_id`, `title`, `status`, `next step`
  - Files touched
  - Commit/PR link (if applicable)
  - Test results (if applicable)
- **Update the Knowledge Graph (memory)**:

  - Ensure base entity `project:${PROJECT_TAG}` exists.
  - Upsert `task:${task_id}` and any `file:<path>` entities touched.
  - Create/refresh relations:

    - `project:${PROJECT_TAG}` —\[owns]→ `task:${task_id}`
    - `task:${task_id}` —\[touches]→ `file:<path>`
    - `task:${task_id}` —\[status]→ `<status>`
    - Optional: `task:${task_id}` —\[depends\_on]→ `<entity>`
  - Attach `observations` capturing key outcomes (e.g., perf metrics, regressions, decisions).
- **Documentation maintenance (auto)** — when a task **changes user-facing behavior**, **APIs**, **setup**, **CLI**, or **examples**:

  - **README.md** — keep **Install**, **Quickstart**, and **Usage/CLI** sections current. Add/adjust new flags, env vars, endpoints, and examples introduced by the task.
  - **docs/CHANGELOG.md** — append an entry with **UTC date**, `task_id`, short summary, **breaking changes**, and **migration steps**. Create the file if missing.
  - **docs/** pages — update or add topic pages. If present, refresh **`index.md`**/**`SUMMARY.md`**/**MkDocs/Sphinx nav** to include new pages (alphabetical within section unless a numbered order exists).
  - **Build check** — run the project’s docs build (`npm run docs:build` | `mkdocs build` | `sphinx-build`) if available. Record the result under `DocFetchReport.observations.docs_build`.
  - **Sync scripts** — run `docs:sync`/`docs-sync` if defined; otherwise propose a TODO in the completion note.
  - **Commit style** — use commit prefix `[docs] <scope>: <summary>` and link PR/issue.
  - **Scope guard** — never edit `GEMINI.md` as part of docs maintenance.
- Seed/Update the knowledge graph **before** exiting the task so subsequent sessions can leverage it.
- Do **NOT** write to `GEMINI.md` beyond these standing instructions.

## 3) Status management

- Use Task Master MCP to set task status (e.g., set to "in-progress" when starting).

## 4) Tagging for retrieval

- Use tags: `${PROJECT_TAG}`, `project:${PROJECT_TAG}`, `memory_checkpoint`, `completion`, `agents`, `routine`, `instructions`, plus task-specific tags (e.g., `fastapi`, `env-vars`).
- For memory entities/relations, mirror tags on observations (e.g., `graph`, `entity:task:${task_id}`, `file:<path>`), to ease cross-referencing with mem0.

## 5) Handling user requests for code or docs

- When a task or a user requires **code**, **setup/config**, or **library/API documentation**:

  - **MUST** run the **Preflight** (§A) using the consolidated order (**sourcebot | docfork → contex7-mcp → gitmcp**).
  - Only proceed to produce diffs or create files after `DocFetchReport.status == "OK"`.

## 6) Handling Pydantic-specific questions

- For **ANY** question about **Pydantic**, use the **pydantic-docs-mcp** server:

  - Call `list_doc_sources` to retrieve `llms.txt`.
  - Call `fetch_docs` to read it and any linked URLs relevant to the question.
  - Reflect on the docs and the question.
  - Use this to answer, citing guidance in `DocFetchReport.key_guidance`.

## 7) Project tech stack specifics

- For project-specific stack work (FastAPI, Starlette, httpx, respx, pydantic-settings, pytest-asyncio, sqlite3, etc.):

  - **MUST** run the **Preflight** (§A) with the consolidated order.
  - If a library isn’t found or coverage is weak after **sourcebot → docfork → contex7-mcp → gitmcp**, fall back to **exa** (targeted web search) and mark gaps.

## 8) Library docs retrieval (topic-focused)

- Use **sourcebot** first to fetch current docs before code changes. If configured and suitable for your workflow, **docfork** may be used to assemble or snapshot the retrieved docs.
- If the primary retrieval tool fails, use **contex7-mcp**:

  - `resolve-library-id(libraryName)` → choose best match by name similarity, trust score, snippet coverage.
  - `get-library-docs(context7CompatibleLibraryID, topic, tokens)` → request focused topics (e.g., "exception handlers", "lifespan", "request/response", "async client", "retry", "mocking", "markers", "sqlite schema/init").
- If **contex7-mcp** also fails, use **gitmcp** (repo docs/source) to retrieve equivalents.
- Summarize key guidance inline in `DocFetchReport.key_guidance` and map each planned change to a guidance line.
- Always note in the task preamble that docs were fetched and which topics/IDs were used.

---

## 9) Environment & Testing Policy (uv-first, no auto-test)

**Objective:** Prevent environment drift across Windows/Linux/WSL and protect any existing `.venv`. The user runs tests after the test plan is documented.

**Rules:**

1. **uv-only for Python environments.**

   - Use `uv` for all environment operations: creation, sync, installs, and running tools.
   - **Never** suggest or run `python -m venv`, `pip`, `pipx`, `poetry`, or raw `pytest` without `uv` unless the user explicitly overrides.
2. **Do not execute tests automatically.**

   - Produce a **Test Plan** per task that lists exact commands and expected outcomes.
   - Mark commands as **“proposed, not executed.”** The user will run them and share outputs.
3. **Respect existing `.venv`.**

   - Do **not** create, delete, modify, or activate virtual environments without explicit user approval.
   - Prefer `uv run` over shell activation to avoid platform-specific activation steps.
4. **Cross‑platform safety.**

   - Provide platform‑neutral commands and avoid assumptions about path separators or shells.
   - Prefer reading project files (`pyproject.toml`, `requirements*.txt`) over executing environment/version commands.
5. **Explicit consent for any environment‑touching command.**

   - Even read‑only checks (e.g., `uv pip list`) require explicit user consent.

**Command mapping (examples, propose only):**

- Create venv (if approved): `uv venv .venv`
- Sync from `pyproject.toml`: `uv sync`
- Install package: `uv add <pkg>` **or** `uv pip install <pkg>`
- Run Python tool: `uv run python -m <module> ...`
- Run pytest: `uv run pytest -q` (only after Test Plan approval; user executes)
- Run CLI tools without adding to env: `uvx ruff check .`, `uvx mypy .`

**Test execution protocol:**

- Include a **Proposed Test Commands** block in deliverables, not executed by the assistant.
- Wait for the user to run and paste results; analyze and proceed accordingly.

---

### System-prompt scaffold (enforcement)

```
SYSTEM: You operate under a blocking docs-first policy.
1) Preflight (§A):
   - Call sourcebot → docfork (if configured) → contex7-mcp → gitmcp as needed.
   - Build DocFetchReport (status must be OK).
2) Planning:
   - Map each planned change to key_guidance items in DocFetchReport.
3) Decision Gate (§B):
   - If DocFetchReport.status != OK → STOP and return "Docs Missing" with exact MCP calls.
4) Execution:
   - Proceed only if ctx.docs_ready == true.
5) Completion:
   - Attach DocFetchReport and write completion memory (§2).
```

---

*End of file.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AcidicSoil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AcidicSoil)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
