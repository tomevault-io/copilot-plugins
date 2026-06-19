## daemonstrate

> Identifies a repo's scopes (frontend, backend, jobs, data, integrations, feature slices, …), persists them as a durable catalog, then renders `.drawio` diagrams per scope — Claude picks the best diagram type for each scope from a menu of eight flowchart kinds (Process Flowchart / Map, Swimlane, Data Flow Diagram, Workflow Diagram, System Flowchart, Document Flowchart, EPC, Influence Diagram). Every rendered diagram ships with paired Technical / Plain-English page tabs so the reader picks their audience at view time. An interactive teaching mode lets the user pick an area, then recursively drill into specific components (file → functions → branches → library calls) as deep as the source structure supports, producing focused single-topic diagrams ideal for explaining one piece of the system to a non-technical audience. Trigger this skill whenever the user asks to visualize, diagram, map, chart, or document their project architecture; wants to explain a specific component or feature to a non-technical audience; wants to onboard a recruiter / collaborator / new teammate; mentions their diagrams are stale or missing; mentions draw.io, flowcharts, swim-lanes, DFDs, process maps, workflow diagrams, or influence diagrams; or when no `docs/.daemonstrate-scopes.json` exists in the current repo and the user is working on documentation, onboarding, portfolio, or repo cleanup. Also trigger on explicit phrases like "Daemonstrate", "map the scopes", "refresh the architecture diagrams", "explain how X works", or "show what this project does".


# Daemonstrate

Identifies the scopes of the **target repo**, persists that catalog so it survives between runs, and renders one or more light-theme `.drawio` diagrams. For each scope, Claude picks the diagram type that best fits what the scope *is* (a stepwise process? a handoff between actors? a data flow? a decision?) from a menu of eight well-established flowchart kinds.

**Every rendered diagram ships with paired Technical + Plain-English page tabs** inside the same `.drawio` file. A reader clicks a bottom-tab to switch views — the Technical page uses stack-native vocabulary (HTTP verbs, JWT, file paths, library names); the Plain-English page translates to user-facing language ("app talks to server", "saves the note", "checks your login"), hides tech badges and file-path subtitles, and is optimized for anyone unfamiliar with the codebase. Translation runs once per label and caches to the catalog. Readers choose the audience at view time, not at generation time.

**Interactive teaching mode** activates when `/daemonstrate` is invoked from chat with an existing catalog. The user picks one area from a numbered list, then (recursively) drills into any component as deeply as the source structure supports — file → functions → branches → library calls — and Claude produces a focused single-topic diagram for each level. Ideal for explaining one specific piece of the system (e.g., "how does auth middleware actually work?") to a non-technical audience.

All outputs share one visual language — dashed pastel group containers, orthogonal routed edges, dark text on white, drop shadows — so a reader can move between per-scope diagrams without re-orienting. By default they live under `docs/` in the target repo.

## Path resolution

Resolve these paths once at the start of every run:

- **`SKILL_DIR`** — the directory containing this `SKILL.md`. You already know it because you just loaded this file. All bundled scripts (`drawio_builder.py`, `install-hooks.sh`) live at `$SKILL_DIR/scripts/`. **Never hardcode an absolute path** — this skill must work on any machine.
- **`TARGET_REPO`** — the repo being diagrammed. Default: the current working directory's git root (`git rev-parse --show-toplevel`). Override: if the caller explicitly gave you a different repo path, use that.
- **`OUT_DIR`** — where diagrams and the scopes catalog are written. Default: `$TARGET_REPO/docs/`. Override: if the caller explicitly gave you a different output directory (e.g., during an evaluation run), use that instead — and in that case *do not* also write into `$TARGET_REPO/docs/`.

Before invoking the builder or installer, confirm the resolved paths — mention them in your first user-facing status message, **or** echo them into the run's report file if the context is non-interactive (hook, eval, script). Either way, make the paths visible so a misrouted run is caught early.

**All working artifacts respect `$OUT_DIR`** — not just the diagrams and the scopes file, but the intermediate `graph-spec.json`, any report/log, any scratch files. Never drop working files into `$TARGET_REPO` even if OUT_DIR differs from the default.

## Interactive vs. non-interactive runs

Three invocation patterns — two interactive, one non-interactive. The catalog's presence decides which interactive flow applies.

**Interactive — catalog exists (teaching mode).** User invokes `/daemonstrate` from chat on a repo with `.daemonstrate-scopes.json` already in `$OUT_DIR`. Drop into **Phase 0.5** — area selection → optional recursive drill-down → render one focused diagram for the selection. This is the default for "explain how X works" requests.

**Interactive — no catalog (first run).** User invokes `/daemonstrate` on a repo that's never been mapped. Run full Phase 0 discovery, then fall through to whole-repo generation (portfolio + hybrid-detailed, paired Technical/Plain-English pages). Offer the teaching flow as a follow-up: "Want to drill into a specific area now that everything's mapped?"

**Non-interactive** — triggered by the post-commit hook, a CI job, or an eval harness. No prompts possible. Always run whole-repo generation; never enter Phase 0.5. Apply these defaults:
  - Diagram-type picks: commit Claude's judgment without confirmation.
  - Page structure: **hybrid**.
  - Audience pages: always emit **both** Technical and Plain-English tabs.
  - Hook install: **skip** (assume the caller controls hooks).
  - Hand-edited diagram detected: save proposed output as `architecture-*.proposed.drawio` alongside the original, do **not** overwrite. Log the conflict.
  - All status messages go to `$OUT_DIR/daemonstrate-run.log` instead of chat.

Detect interactive mode by checking whether the caller gave you a real conversational context (a user turn above) vs. a one-shot prompt from `claude -p` / an eval agent. When in doubt, prefer the non-interactive defaults — they're strictly safer.

**Power-user override:** `/daemonstrate all` (or an explicit argument) in interactive mode skips Phase 0.5 and runs the whole-repo generation path, even when a catalog exists. Use this to force a refresh of the portfolio + detailed diagrams without stepping through area selection.

## Workflow

The skill runs in five phases:

1. **Phase 0 — Scope Discovery.** Identify the repo's scopes and persist to `$OUT_DIR/.daemonstrate-scopes.json`. On subsequent runs, reuse the catalog and only refresh scopes whose files changed.
2. **Phase 0.5 — Scope + Drill-down selection (interactive teaching mode only).** User picks one area from a numbered list; optionally drills into a specific component, then recursively deeper (function → branch → call) as far as the source structure supports. Skipped in whole-repo and non-interactive runs.
3. **Phase 1 — Per-scope diagram-type judgment.** For each scope (or drill-down target), Claude picks the best-fitting diagram type. Audience is NOT a factor — every rendered diagram carries paired Technical + Plain-English pages, so readers choose at view time.
4. **Phase 2 — Page structure.** Dev picks single / multi / hybrid for whole-repo runs. Drill-down diagrams are always single-page (with paired audience tabs).
5. **Phase 3 — Generate.** Map (per-scope type × page structure) to builder invocations, always emitting paired Technical + Plain-English pages. Report honestly about unsupported combos — never silently fall back to a different diagram type.

## Phase 0 — Scope Discovery

A **scope** is a bounded area of responsibility in the repo — most commonly a layer (frontend, backend, data, jobs, integrations, infra), but also feature slices that cross layers ("Send-to-IDE", "Capture → AI → render"), or subsystems with their own package (an MCP server, a CLI, a migration tool).

Check for `$OUT_DIR/.daemonstrate-scopes.json`. If missing, run full discovery. If present, load the catalog; run an incremental refresh if commits have landed since `lastSha` (see *Incremental refresh* below).

### Full discovery

1. **Detect tech stack.** Read manifests that exist in the repo root and major subdirectories to populate a shared tech-badge set:
   - JS/TS: `package.json` (all of them, including subdirs like `client/`, `mobile/`, `server/`, `mcp-server/`)
   - Python: `pyproject.toml`, `requirements.txt`, `Pipfile`
   - Go: `go.mod`; Rust: `Cargo.toml`; Ruby: `Gemfile`; Java/Kotlin: `pom.xml`, `build.gradle(.kts)`; Elixir: `mix.exs`; PHP: `composer.json`
   Also glance at `Dockerfile`, `docker-compose.yml`, CI config (`.github/workflows/*`), and `.env.example` for infra + environment shape.

2. **Classify scopes.** Walk the top two directory levels and classify each significant dir into a **scope**. Common layer-kind scopes:

   | Scope | Typical markers |
   |---|---|
   | **Frontend** | `client/`, `web/`, `ui/`, React/Vue/Svelte deps, `public/`, `index.html` |
   | **Mobile** | `mobile/`, `ios/`, `android/`, Expo/React Native deps |
   | **Backend / API** | `server/`, `api/`, `routes/`, Express/Fastify/Django deps |
   | **Data** | `migrations/`, `prisma/`, `schema.sql`, `models/`, SQL files |
   | **Jobs / Workers** | `cron`, `scheduler`, `workers/`, `queues/`, BullMQ/Celery |
   | **Integrations** | SDK deps (Stripe, Twilio, Gemini, OpenAI, Firebase, AWS…) |
   | **Infra** | `Dockerfile*`, `docker-compose*`, `.github/workflows/`, `terraform/`, `k8s/` |
   | **Docs / Assets** | `docs/`, `assets/` — usually *not* rendered as a diagram |

   Then look for **cross-cutting scopes** — scopes whose whole identity is that they *span* multiple layers. These include:

   - **Feature slices** — end-to-end flows that touch frontend, backend, jobs, and integrations (e.g. `send-to-ide` handlers in `main.js`, `api-server.js`, and `mcp-server/index.js`; OAuth sign-in; clip sync)
   - **Cross-cutting concerns** — infrastructure behaviors threaded through every layer (auth, logging, observability, feature flags, rate limiting, error handling)
   - **End-to-end subsystems** — packages that own one coherent responsibility across a stack (a bundled MCP server + its client hooks + its CLI)

   Signals: a shared name across directories, a README section describing an end-to-end feature, or a concern that shows up as cross-layer edges on every other scope's diagram. Promote a cross-cutting concept to its own scope when it'd read better as its own diagram than as noisy edges bleeding across every layer diagram — which is almost always true for feature slices, and often true for cross-cutting concerns. (Cross-cutting scopes tend to pair naturally with **Swimlane** as their diagram type, and are the primary case where **overlays** earn their keep — see Phase 1.)

   Omit empty layers. A one-scope repo still gets a bounded container — clearer than a free-floating set of nodes.

   **Disambiguating adjacent-looking files.** Many projects co-locate orchestration logic with thin SDK adapters (e.g. `server/services/` holding both `note-pipeline.js` and `sms.js`):
   - **Integrations** = files whose job is to *talk to an external system* (Twilio, Stripe, Gemini, Expo Push, OAuth). Usually thin wrappers around a third-party SDK.
   - **Jobs / Workers** = files that *orchestrate our own business logic on a schedule or queue* (categorization pipelines, alert schedulers, cron handlers).

   A pipeline that calls a Twilio wrapper lives in **Jobs**; the wrapper itself lives in **Integrations**. Edges between them are fine and expected.

3. **Explore each scope.** Two strategies; pick based on context, not a hard rule.

   **Inline exploration** (default): Glob + Grep + Read directly, one scope at a time. Fast for small-to-medium repos (under ~50 source files per scope) and budget-constrained runs.

   **Parallel subagent exploration** (opt-in for large repos): Dispatch one Explore subagent per scope in a single message, when **all** of the following hold:
   - The repo is large (many hundreds of source files, multiple services, or a monorepo).
   - The conversation is interactive.
   - You are confident the scope boundaries are clean enough for independent exploration.

   Dispatched subagents explore inline — do not recurse.

   **If you're being invoked from a hook, an eval run, or a script → always explore inline.**

   Same probe either way (inline prompt or subagent brief):

   > You are mapping the **{scope}** of a `{tech}` project for an architecture diagram. Explore `{paths}`. Return JSON matching `references/layer-schema.md`:
   > - `capabilities`: 3-6 short phrases describing what *users* (or callers) can do via this scope
   > - `key_nodes`: 6-12 structural nodes (components, routes, tables, jobs) with `id`, `label`, `file_path`, optional `detail`
   > - `internal_edges`: edges between nodes in this scope
   > - `external_edges`: edges this scope extends to *other* scopes, referenced by scope name + node label
   > - `tech_badges`: the specific tech this scope uses (e.g., `["React 19", "Vite", "React Router 7"]`)
   > Keep labels under 4 words. No prose. JSON only.

4. **Persist the catalog.** Write `$OUT_DIR/.daemonstrate-scopes.json`:

   ```json
   {
     "version": 2,
     "lastSha": "<HEAD SHA>",
     "identifiedAt": "<ISO 8601>",
     "techStack": ["React 19", "Express 5", "Postgres 16", "…"],
     "lastStructure": "hybrid",
     "scopes": [
       {
         "id": "frontend",
         "display_name": "Frontend",
         "plain_display_name": "The website",
         "kind": "layer",
         "accent": "#89B4FA",
         "paths": ["client/src/**/*.jsx", "client/package.json"],
         "tech_badges": ["React 19", "Vite"],
         "capabilities": ["Google OAuth sign-in", "Offline-first PWA", "…"],
         "plain_capabilities": ["Sign in with Google", "Works without internet", "…"],
         "key_nodes": [
           {
             "id": "fe-app",
             "label": "App Shell",
             "plain_label": "The whole website",
             "file_path": "client/src/App.jsx",
             "detail": "React Router 7 root",
             "plain_detail": "The outermost layer that holds every page",
             "source_region": {"file": "client/src/App.jsx"},
             "sub_components": null
           }
         ],
         "internal_edges": [
           {"from": "fe-app", "to": "fe-auth", "label": "context", "plain_label": "holds your login state"}
         ],
         "external_edges": [],
         "diagramType": "system_flowchart",
         "diagramTypeReason": "Primarily a component/route map — readers want to see what's there and how it connects.",
         "overlay": null,
         "overlayReason": null
       }
     ]
   }
   ```

   **Field notes:**
   - `paths` stores file glob patterns — this is what enables diff detection next run.
   - `diagramType`, `diagramTypeReason`, `overlay`, and `overlayReason` are populated in Phase 1; leave them null until then. `overlay` stays null for most scopes — it's a conscious add, not a default (see Phase 1 → Overlays).
   - `plain_display_name`, `plain_capabilities`, and per-node / per-edge `plain_label` / `plain_detail` are populated the first time a scope is rendered (or first drilled into). They translate stack vocabulary to user-facing language for the Plain-English page tab. Cache forever; invalidate only when the paired technical label changes. Leave them null or absent until the first rendering pass fills them in — the translator is a small AI call per string.
   - `source_region` on each key_node tells the drill-down probe exactly what text to re-read. For file-level nodes (the default after initial discovery), `{"file": "..."}`. Deeper nodes (functions, branches, blocks) use `{"file": "...", "symbol": "..."}` or `{"file": "...", "start_line": N, "end_line": M}`.
   - `sub_components` is the lazy drill-down tree. `null` = never explored. Populated only when a user asks to drill into this node via Phase 0.5. See *Phase 0.5 → Recursive drill-down probe* for its shape.
   - Schema version is **2**. Runs that encounter a version-1 catalog treat the missing fields as null/absent and populate them lazily — no migration needed.

### Incremental refresh

The catalog has **three cache levels**, each with its own invalidation key. A change at one level doesn't waste work at the others.

**Level 1 — Scope content cache.**

When the scopes file already exists:

```bash
git diff --name-only <scopes.lastSha> HEAD
```

For each changed file, find which scope(s) it belongs to via `paths`. Re-explore only those scopes, passing the previous scope snapshot + the list of changed files so the subagent produces a **delta**:

> You previously produced `{previous scope JSON}`. The following files have changed: `{list}`. Produce an **updated** scope JSON reflecting additions, removals, and label changes. **Preserve existing node IDs where the concept still exists** (so diagram positions stay stable).

Merge deltas, bump `lastSha`, rewrite `.daemonstrate-scopes.json`.

**Level 2 — Sub-component tree cache.**

Each `sub_components` block records a `source_hash` (SHA of the source region's text at exploration time). On every drill-down revisit:

```
current_hash = sha1(read(node.source_region))

if current_hash == node.sub_components.source_hash:
    reuse cached sub_components tree        # free, instant
else:
    re-probe just this level via the drill-down probe
    discard any child sub_components (they point into stale text)
    re-cache with new hash
```

An edit to one function in `auth.js` invalidates that function's sub-components but leaves sibling functions' trees valid. Surgical and cheap.

**Level 3 — Plain-English label cache.**

`plain_label` and `plain_detail` are keyed on the technical `label` and `detail` they translate from. Mechanism:

```
for each field in {label, detail, capabilities, edge labels, display_name}:
    if scope[field] changed vs. the catalog's previous version:
        invalidate scope.plain_<field>
    else:
        reuse the cached plain_<field> byte-identical
```

On invalidation, translate just that one string via a small AI call. If nothing changed, the Plain-English cache is byte-identical and zero AI cost.

**What's NOT caught automatically:**
- **Uncommitted working-tree edits.** By design: commits are the unit of change. For users iterating on uncommitted code during drill-down, the Phase 0.5 menu always offers **"Re-probe this component"** as a manual trigger that bypasses the hash cache at that level.
- **New scopes appearing wholesale** (a brand-new top-level directory). Incremental refresh updates existing scopes but doesn't promote new areas to scope status. Escape hatch: delete `.daemonstrate-scopes.json` and re-run for a full rediscovery.
- **Cross-scope file moves.** If a file moves from `server/` to `workers/` and neither scope's `paths` glob matches the new location, it falls into a gap. Watch for missing nodes in the next generation; the fix is either updating the scope's `paths` or a full rediscovery.

## Phase 0.5 — Scope + Drill-down selection (interactive teaching mode)

This phase runs only when the caller is interactive AND the catalog already exists. It lets the user choose exactly what they want diagrammed — a whole area, a specific component, or a function/branch inside that component — before Phase 1 picks a diagram type.

Skipped in non-interactive runs, first-run interactive runs (no catalog yet), and when the user explicitly invoked `/daemonstrate all`.

### Step 1 — Area selection

Render the scopes as a numbered table. Use `plain_display_name` if populated, else fall back to `display_name`. Derive each row's summary from the first `plain_capabilities` entry (or `capabilities[0]` if plain isn't yet translated).

> Which area of the project do you want to diagram?
>
> | #  | Area            | What it handles                                    |
> |----|-----------------|----------------------------------------------------|
> | 1  | Frontend (PWA)  | The website — sign-in, note list, compose form     |
> | 2  | Mobile App      | The phone app — offline notes, widgets, share      |
> | 3  | Backend API     | The server — handles requests, auth, SMS intake    |
> | 4  | Data            | Where notes, alerts, and patterns are stored       |
> | 5  | Jobs / Workers  | Background work — categorizing, reminders          |
> | 6  | Integrations    | Talks to Twilio, Google AI, Expo push, etc.        |
> | 7  | Infra           | How the app is packaged and deployed               |
>
> Pick a number, or type `all` to render the whole-repo portfolio:

`all` falls through to the whole-repo generation path (same as the non-interactive default). A numeric pick enters Step 2.

**Cardinality.** One scope per run. The conversation stays linear and each run produces one focused artifact. Users re-run `/daemonstrate` for another area.

### Step 2 — Drill-down navigation (recursive)

After a pick, list the current level's components as a numbered table. At level 1 this is the scope's top-level `key_nodes`; at deeper levels this is the parent node's cached `sub_components.sub_nodes` (or a freshly probed result).

If any `plain_label` at the current level is null, run a single batched translation call before rendering the table — this populates `plain_label` + `plain_detail` for every component at this level so the menu reads in the user's chosen vocabulary.

> Backend API has these parts. Pick one to explore, or `0` for the whole area.
>
> | #  | Component             | What it does                                         |
> |----|-----------------------|------------------------------------------------------|
> | 0  | **Whole Backend API** | Full picture — all components + connections          |
> | 1  | Express app           | Every request comes in here first                    |
> | 2  | Auth middleware       | Checks your login cookie before you see your notes   |
> | 3  | Twilio signature      | Verifies that incoming SMS is really from Twilio     |
> | 4  | /api/notes            | Create, read, update, delete your notes              |
> | 5  | /api/mobile/sync      | Mobile offline sync — pulls changes since last sync  |
> | 6  | /api/webhooks/twilio  | Where incoming SMS messages arrive                   |
> | …  |                       |                                                      |
>
> Pick a number, or type `back` / `done`:

`0` jumps to rendering for the whole current-level target. A numeric pick of a child component descends into Step 3 at that level.

### Step 3 — Depth selection (first-time or cache-stale only)

If the picked component has no cached `sub_components` (or the Level 2 staleness check failed), ask:

> You picked **Auth middleware**. How deep do you want to go?
>
> 1. **Focused diagram** — Shows how auth middleware connects to routes, the user's request, and what it hands off to. Uses what I already know. Fast.
> 2. **Deep dive** — I'll re-read `server/middleware/auth.js` to map the exact checks, branches, and error paths inside the file. Takes a few extra seconds but produces an accurate internal diagram.
>
> Pick 1 or 2:

Option 1 renders immediately from the current level's cached data as a focused subgraph (the picked component + its 1-hop neighborhood). Option 2 runs the drill-down probe below, caches the result, then renders.

If `sub_components` is already cached and fresh, skip Step 3 and go straight to render — the user has already decided this component is worth exploring. Always include **"Re-probe this component"** as a menu option at Step 4 so users iterating on uncommitted edits can force a fresh read.

### Recursive drill-down probe

Used for Step 3 option 2 or a cache miss. Narrow probe, one level at a time:

> You are zooming into **{label}** at `{source_region}`. Read that code region and return JSON matching this shape:
>
> - `sub_nodes`: 3-10 direct sub-components inside this region, each with:
>   - `id` (unique; prefix with parent id, e.g. `be-authmw-verify`)
>   - `label` (≤5 words, stack-native vocabulary — Plain-English translation happens later)
>   - `kind`: one of `check` / `decision` / `action` / `error` / `external_call`
>   - `source_region`: where this sub-component lives — use `{"file": "...", "symbol": "..."}` for named functions/classes, `{"file": "...", "start_line": N, "end_line": M}` for anonymous blocks
>   - `detail` (optional, one sentence)
> - `sub_edges`: edges between `sub_nodes`, labeled with what triggers the transition (`valid`, `expired`, `missing`, `error`, `ok`, etc.). Mark error/retry edges with `"is_error": true`.
> - `terminal`: `true` if this region has no further decomposable structure (atomic library call, single-statement operation, constant)
>
> Return `terminal: true` for: library calls, single-statement operations, constants, simple property access. Do **not** hallucinate internal structure where none exists.

Write the result under the parent node's `sub_components`:

```json
"sub_components": {
  "exploredAt": "2026-04-22T03:15:00Z",
  "source_hash": "a3f...",
  "terminal": false,
  "sub_nodes": [
    {
      "id": "be-authmw-readcookie",
      "label": "Read access_token cookie",
      "plain_label": "Look for your login cookie",
      "kind": "check",
      "source_region": {"file": "server/middleware/auth.js", "start_line": 8, "end_line": 14},
      "sub_components": null
    },
    {
      "id": "be-authmw-verify",
      "label": "Verify JWT signature",
      "plain_label": "Make sure the cookie is valid",
      "kind": "check",
      "source_region": {"file": "server/middleware/auth.js", "symbol": "verifyAccessToken"},
      "sub_components": null
    }
  ],
  "sub_edges": [
    {"from": "be-authmw-readcookie", "to": "be-authmw-verify", "label": "cookie present"},
    {"from": "be-authmw-readcookie", "to": "be-authmw-403",    "label": "cookie missing", "is_error": true}
  ]
}
```

`sub_components: null` on a child = not yet explored (lazy). Subsequent drill-downs populate it.

### Step 4 — Render + continue

Render the diagram for the current target, then offer:

> ✓ Rendered: `docs/drilldown-backend-authmw.drawio`
>
> What next?
> 1. `deeper` — Pick a part of **Auth middleware** to zoom into
> 2. `back` — Return to Backend API's component list
> 3. Re-probe this component (force fresh read — useful if you just edited the file)
> 4. `done` — Finish this session
>
> Pick a number, or type `deeper` / `back` / `done`:

- `deeper` loops to Step 2 at the next level down (the just-rendered component's sub_nodes become the new level's menu).
- `back` pops the drill-down stack by one level.
- Re-probe sets this level's `source_hash = ""` and re-enters Step 3 → Deep dive.
- `done` exits the loop and writes the catalog.

If the just-rendered node is `terminal: true`, option 1 is hidden and the UI shows:

> This is as deep as it goes — **jsonwebtoken.verify** is a single library call with no further internal structure I can extract.

### Depth safety rails

- **Hard cap at 6 levels.** If the stack grows to depth 6, ask explicitly: "This is 6 levels deep. Keep going?" Default: no.
- **Per-session probe budget.** Track the number of drill-down AI calls issued this session and include the total in the run report. No hidden spending.
- **Partial-session persistence.** Catalog updates are written after each successful probe, so interrupting (Ctrl-C, network hiccup, session end) doesn't lose explored work. The next run picks up exactly where you left off.

### Output filenames

Drill-down diagrams are path-named by the trail:

```
docs/drilldown-{scope}.drawio                                   # whole area
docs/drilldown-{scope}-{component}.drawio                       # level 1
docs/drilldown-{scope}-{component}-{sub}.drawio                 # level 2
docs/drilldown-{scope}-{component}-{sub}-{subsub}.drawio        # level 3
```

Slug each segment from the node's `id` (lowercase, dashes). Each file is single-page with paired Technical/Plain-English tabs — self-contained; a reader can open any level's file without needing its siblings.

## Phase 1 — Per-scope diagram-type judgment

For each scope in the catalog (or the current drill-down target from Phase 0.5), pick one of these eight diagram types based on what the target actually *is*. Audience is **not** a factor — every rendered diagram carries paired Technical + Plain-English pages, so readers choose at view time.

Depth shifts the natural pick, though: deep drill-downs (inside a function, branch, or block) almost always read best as a Process Flowchart regardless of what the containing scope picked. See *How to choose → Depth awareness* below.

Read the rubric as "use this type when the target's primary story is …":

| Type | Use when the scope is… | Typical examples |
|---|---|---|
| **Process Flowchart / Map** (high- or low-level) | …a sequence of steps with decision points and branches | Categorization pipeline, capture flow, retry/backoff logic |
| **Swimlane Flowchart** | …a process that crosses actors/services, where handoffs matter more than the steps themselves | Send-to-IDE (app / filesystem / MCP / IDE agent), OAuth sign-in, any cross-boundary orchestration |
| **Data Flow Diagram (DFD)** | …primarily about data moving between stores and processes (inputs → transforms → outputs → storage) | Sync pipelines, ETL, clipboard → DB → AI → renderer |
| **Workflow Diagram** | …a business/team process involving tools and roles | Multi-agent orchestration (architect / builder / reviewer), release pipeline |
| **System Flowchart** | …infrastructural — data moving through physical components and services | Infra scope, network/process topology, DB replication, deployment pipeline |
| **Document Flowchart** | …document movement between units | PR → review → merge audit trail, generated-report flow |
| **EPC (Event-driven Process Chain)** | …ERP-style event → function → event chains | Rare in typical codebases; skip unless the scope genuinely matches |
| **Influence Diagram** | …primarily decisions, goals, and risks | Rule-chain categorization (priority chain), policy engine, routing decisions |

### How to choose

Lead with what the scope *does*, not the tech stack it's built on. A scope that "talks to an external API" could be a process flowchart (happy-path + error branches), a swimlane (client / server / third-party as lanes), or a DFD (request/response payloads) — the right pick depends on what a reader would find most useful to *see*.

Two helpful tiebreakers:

- **Lower reader effort wins.** Swimlane beats process flowchart for handoffs because the lanes make responsibility visible without reading labels. DFD beats system flowchart when the story is about data volume / shape rather than which service holds it.
- **Match the audience you expect.** If the dev's goal is "onboard a recruiter", Process Flowcharts read faster than Influence Diagrams. If the goal is "debug the pipeline", Swimlane + DFD beat System Flowchart.

If no type in the menu fits well (e.g. a scope that's purely static relationships — tables, components, dependencies), pick the closest structural option (**System Flowchart** is usually it) and set `diagramTypeReason` to say so explicitly. Do not invent types outside this menu.

### Depth awareness (drill-down targets)

A whole scope's natural diagram type may not apply to one of its internals. Use this table when Phase 0.5 picked a specific drill-down target rather than a whole scope:

| Depth | Typical natural pick | Why |
|---|---|---|
| Level 0 (whole scope) | Any of the 8, per the scope's story | As before. |
| Level 1 (a file / component within a scope) | **System Flowchart (focused)** or **Swimlane** if the component's story crosses a boundary | Readers want to see how this one thing connects to its neighborhood. |
| Level 2+ (function, branch, block) | **Process Flowchart (Journey)** | Deep internals are almost always a sequence: check → decide → act → exit. Rendered via `--out-journey`. |
| Any level, cross-process handoff | **Swimlane** | If the target involves talking to an external system or another scope, lanes make the responsibility split obvious. |
| Terminal node | — (no diagram) | Show "this is as deep as it goes" message with the source snippet; no rendering. |

Record the picked type in the drill-down node's `diagramType` field (same as scope-level) so next run reuses it or offers it as a default.

### Overlays (layering a second dimension)

An **overlay** is what happens when a diagram is loaded with more than one kind of information at once. The base diagram tells one story — *who does what when*, or *what the steps are*, or *where data flows*. An overlay adds a second dimension on top of that single picture, encoded visually via color, border style, annotation badges, or row shading. You're not drawing two diagrams side by side — you're encoding multiple stories into one picture.

Overlay is **not a diagram type**. It's a visual technique that can be applied to *any* of the eight types in the menu. Claude has creative freedom to reach for it whenever a scope has a second dimension worth telling in the same frame. Most scopes don't; for those, leave `overlay` null and let the base diagram do its one job well.

**Cross-cutting scopes are the primary case.** A scope that crosses layers (feature slice, end-to-end flow, cross-cutting concern) almost always has a useful second story — which layer owns each step, where failures hop the boundary, which steps are shipped vs. planned, which require a paid tier. A pure intra-layer scope (a single component tree, the rules engine, a database schema) rarely does. When in doubt: if the scope touches ≥3 layers, ask what the second story is; if it touches 1, default to no overlay.

A non-exhaustive menu of overlay dimensions worth considering — use these as starting points, and invent others when a scope's story calls for one:

| Overlay dimension | What it encodes | Visual channel | Pairs naturally with |
|---|---|---|---|
| **Data state** | Shape/kind of data at each step (raw image → extracted text → categorized clip) | Node fill color | DFD, Process Flowchart, Swimlane |
| **Error / retry paths** | Which edges are happy-path vs. failure/retry | Edge style (solid / dashed / red) | Any type with edges |
| **User emotional arc** | How the user feels at each step (confident → frustrated → relieved) | Row shading or badge | Swimlane, Process Flowchart, Workflow |
| **Subsystem boundary** | Which process / service / device owns each step | Row shading (one band per subsystem) | Swimlane, System Flowchart, Process Flowchart |
| **Shipped vs. planned** | What exists today vs. roadmap | Opacity or dashed border | Any type |
| **Billing / tier / permission gate** | Which steps require a paid tier, role, or feature flag | Badge on the step | Process Flowchart, Workflow, Swimlane, DFD |
| **Latency / cost** | Step-level timing or $/call | Numeric annotation badge | DFD, System Flowchart, Swimlane |
| **Environment / region** | Which steps run in prod / staging / edge / a specific region | Row shading or border color | System Flowchart, DFD |
| **Risk / uncertainty** | Confidence in a decision or failure probability of an edge | Badge or edge weight | Influence Diagram, Process Flowchart |
| **Ownership / team** | Which team owns each step — useful in cross-team scopes | Badge with team initials | Workflow Diagram, Swimlane |

**When to add an overlay:** only when the second dimension answers a question the base diagram can't. If a reader would want to know *which steps fail* or *where the data changes shape* or *which steps are paid-tier-gated* at the same time as the base story, the overlay is earning its keep. If you can't name the question the overlay answers, don't add one — dense diagrams lose readers.

**Visual encoding discipline.** Pick one channel per overlay dimension and apply it consistently:
- **Color fill** — best for per-node categorical state (data shape, tier, environment)
- **Edge style** — best for per-edge binary/ternary state (happy-path vs. failure)
- **Row / band shading** — best for grouping nodes across lanes (subsystem boundary, environment)
- **Badge / annotation** — best for per-step scalar data (latency ms, cost $, owner initials)
- **Opacity or dashed border** — best for shipped-vs-planned or provisional status

Always include a small legend on the page explaining the overlay. A diagram with unexplained color coding is noise, not signal.

**Cap: at most one overlay dimension per page.** Two dimensions on top of a base diagram is three stories in one picture, and readers will stop parsing. If two dimensions are both load-bearing, make two pages (same base diagram, different overlay) rather than one crowded page. Hybrid page-structure makes this trivially cheap — the second page costs the reader one click.

Record the chosen overlay in the scope entry:

```json
{
  "diagramType": "swimlane",
  "overlay": "error_paths",
  "overlayReason": "Send-to-IDE has meaningful retry + failure branches (write fails, MCP delete fails); readers debugging the pipeline need to see the failure edges."
}
```

Other realistic examples:

```json
// Process Flowchart of the capture pipeline, overlaid with where the data transforms
{"diagramType": "process_flowchart", "overlay": "data_state",
 "overlayReason": "Clip data morphs across steps (image → metadata → categorized clip); color-coding makes the transform points obvious."}

// DFD of the sync pipeline, overlaid with per-step latency
{"diagramType": "dfd", "overlay": "latency",
 "overlayReason": "The AI enrichment step is the hot spot; surfacing timing badges points readers at the right optimization target."}

// System Flowchart of infra, overlaid with environment (prod / staging / local)
{"diagramType": "system_flowchart", "overlay": "environment",
 "overlayReason": "Half these services only exist in prod; row shading prevents onboarding readers from chasing ghosts in their local setup."}
```

Set `overlay` to `null` when the plain base diagram is the right call — which it often is.

### Interactive override pass

Present the per-scope picks as a short table and let the dev override in one pass:

> Here's the diagram type I'd pick for each scope. Say which (if any) you want to change:
>
> | Scope | Type | Overlay | Why |
> |---|---|---|---|
> | Frontend | System Flowchart | — | Components + routing; readers want to see structure. |
> | Capture Flow | Process Flowchart | — | Linear steps with decision points (new image? / hash match?). |
> | Send-to-IDE | Swimlane | error_paths | Handoffs across app / filesystem / MCP / IDE agent; failure branches matter for debugging. |
> | Rules Engine | Influence Diagram | — | Priority-chain decisions over categorization. |
> | Infra | System Flowchart | — | Services / containers / network. |

Non-interactive mode: commit Claude's picks without prompting; log them to `$OUT_DIR/daemonstrate-run.log` with `diagramTypeReason` so a human can audit later.

Record each scope's chosen type and reason into `.daemonstrate-scopes.json` so later runs reuse the pick (or offer it as the default).

## Phase 2 — Page structure

Applies to **whole-repo runs only**. Drill-down targets from Phase 0.5 always render as a single-page file (with paired Technical/Plain-English tabs); page-structure doesn't apply.

For whole-repo runs the dev chooses how per-scope diagrams are laid out across pages:

| Structure | Shape |
|---|---|
| **Single** | Every scope rendered on one page. Forces a single unified diagram type (usually System Flowchart) — heterogeneous per-scope types don't fit here. Use only when there are ≤2 scopes, or the dev explicitly wants a poster. |
| **Multi** | One page per scope, each rendered as its chosen type. No overview. Heavy but deep. Good for reference documents that readers navigate by table of contents. |
| **Hybrid** | Page 1 is a single "everything" overview (a high-level System Flowchart or Process Map showing all scopes + their top-level connections); pages 2..N each drill into one scope rendered as its chosen type. Best of both — scannable entry plus deep detail. |

Note: each "page" in these structures is actually a **pair** of pages in the output file — a Technical variant and a Plain-English variant with identical layout. A hybrid structure with 7 scopes produces 1 + (7 × 2) = 15 page tabs. See Phase 3 for the paired-tab emission.

Interactive mode: ask.

Non-interactive mode: **hybrid**. It serves the most audiences with a single artifact.

Record as `lastStructure` in the scopes file.

## Phase 3 — Generate

Assemble a single **graph spec** (JSON) describing what each page contains, then pipe it to `scripts/drawio_builder.py`. **Do not hand-write the full XML** for builder-native types — it's repetitive and error-prone; the builder handles IDs, layout math, multi-page wiring, and style strings.

### Paired Technical / Plain-English tab emission

Every rendered logical page emits as **two `<diagram>` elements** in the output `.drawio` file — one with the technical labels, one with the plain-English labels. The pair shares identical shape IDs, positions, accent colors, and edge routing; only the `value=` text attributes differ, plus visibility of tech-badge rows and file-path subtitles.

**Tab naming.** Use `(T)` and `(P)` suffixes so tabs stay scannable along drawio's bottom tab bar:

```
Overview (T) | Overview (P) | Backend (T) | Backend (P) | Mobile (T) | Mobile (P) | …
```

**Starting page ordering** (which tab a reader lands on first when they open the file):

| Artifact | First tab | Why |
|---|---|---|
| Whole-repo portfolio / detailed | **Technical** first | Default reader is a developer scanning structure. |
| Drill-down diagrams from Phase 0.5 | **Plain-English** first | Teaching context — reader is here to understand one feature, often non-technically. |

**Plain-English label generation.** When rendering, for every label/detail/edge-label/capability at the target depth:

```
if node.plain_label is null:
    plain_label = translate(label, detail, context_hint)    # single AI call
    cache to catalog
else:
    reuse node.plain_label
```

Translation context hint passed to the AI:

> Translate this technical label into user-facing language for a reader who doesn't know the stack. Keep it under 5 words. Omit library names, HTTP verbs, file extensions, and protocol acronyms. Favor verbs that describe what happens ("saves", "checks", "asks") over nouns ("middleware", "handler", "endpoint"). Input: `{label}` — detail: `{detail}` — context: `{scope name, parent component name}`.

**What differs between the two tabs visually:**

| Element | Technical tab | Plain-English tab |
|---|---|---|
| Node label | `apiFetch` | "App asks the server" |
| Node file-path subtitle | `client/src/api/client.js` | (hidden) |
| Node detail (hover tooltip) | `401 auto-refresh` | "If your login expired, quietly logs you back in" |
| Edge label | `POST /api/notes` | "Saves the note" |
| Scope container header | `Frontend (PWA) • React 19 · Vite · React Router 7` | "The website" (no tech badges) |
| Capability pills | "Cookie-auth + 401 refresh" | "Signs you back in when your login expires" |
| Overlay legends | Full technical labels | Plain-English translations |

### Builder coverage

The bundled builder natively renders **five** of the eight diagram types, plus the overlay framework. Builder-level work to emit the Technical/Plain-English pair is wrapping: every builder flag gets invoked twice with different label inputs, and the two resulting `<diagram>` elements are concatenated into one `.drawio` file with the `(T)` / `(P)` tab names. Existing flags remain:

| Type | Builder flag | Notes |
|---|---|---|
| System Flowchart | `--out-portfolio` (single-page) or `--out-detailed` (multi-page with drill-down) | Portfolio = capability pills + tech badges; detailed = key-nodes grid per scope. |
| Process Flowchart (high-level) | `--out-journey` | The vertical-spine layout approximates a top-to-bottom process map with user/app actors. |
| Workflow Diagram | `--out-detailed` per-scope page | The detailed layout's actor / node grid reads well as a workflow diagram. |
| **Swimlane Flowchart** | `--out-swimlane` | Vertical lanes (one column per actor), time flows top-to-bottom, handoffs via cross-lane arrows. Overlay-aware (all 5 channels). |
| **Data Flow Diagram (DFD)** | `--out-dfd` | Classic three-column layout — externals (rectangles) / processes (ellipses) / stores (open-sided rects). Overlay-aware. |

**Overlay support:** Swimlane + DFD accept an `overlay` block in the spec with `color_fill`, `edge_style`, `row_shading`, `badge`, or `opacity` channels. The builder auto-emits a legend when an overlay is applied. Extending overlay support to portfolio / detailed / journey is a minor follow-up — the mutate-in-place framework (`apply_overlay_*` helpers) works on any already-built Cell list.

### Builder invocation

Resolve paths from `$SKILL_DIR`, not a hardcoded absolute:

```bash
python "$SKILL_DIR/scripts/drawio_builder.py" \
  --spec "$OUT_DIR/graph-spec.json" \
  [--out-portfolio "$OUT_DIR/architecture-portfolio.drawio"] \
  [--out-detailed  "$OUT_DIR/architecture-detailed.drawio"] \
  [--out-journey   "$OUT_DIR/architecture-journey.drawio"] \
  [--out-swimlane  "$OUT_DIR/architecture-swimlane.drawio"] \
  [--out-dfd       "$OUT_DIR/architecture-dfd.drawio"]
```

Which flags to pass depends on the page-structure choice:

- **Single** → emit one file. If *all* scopes' `diagramType` is `system_flowchart`, use `--out-portfolio`. If the dev picked Process Flowchart for the single page (e.g. "just show me the main flow"), use `--out-journey`. Otherwise fall through to "unsupported combo" handling below.
- **Multi** → `--out-detailed` with no overview page 1. Each detail page uses that scope's `diagramType`.
- **Hybrid** → `--out-detailed` with the overview page 1 enabled. Overview is always a System Flowchart (or Process Map) showing every scope + cross-scope edges; detail pages each render as the scope's chosen type.

Python binary resolution:
- Unix/macOS: `command -v python3 || command -v python`
- Windows (Git Bash): `command -v python`; or `where python` in CMD/PowerShell.
- Minimum: Python 3.8 with stdlib only. The builder has no external dependencies.

### Journey spec (when `--out-journey` is used)

Add a top-level `user_flow` array to `graph-spec.json`:

```json
{
  "title": "…", "subtitle": "…", "scopes": [...],
  "user_flow": [
    {
      "id": "s1", "actor": "user", "label": "Sign in",
      "detail": "Google or email — takes 10 seconds",
      "description": "Friendly 1–3 sentence explanation of what happens on this step.",
      "what_you_see": "A short sentence describing the visual cue (button, screen, animation).",
      "tips": ["Optional plain-language tip 1", "Optional tip 2"]
    }
  ]
}
```

`actor` is `"user"` (renders left in sky blue) or `"app"` (renders right in mint green). Keep `label` ≤4 words and `detail` ≤8 words. `description`, `what_you_see`, and `tips` are merged into the step's hover tooltip so the page stays scannable.

### Unsupported-combo handling

When a scope's `diagramType` is one the builder can't render natively (Document Flowchart, EPC, Influence Diagram), **do not silently fall back to a different type**. Pick one of:

1. **Hand-write** the scope's XML for that page using `references/drawio-format.md`, then inject the page into the builder output. Slower, more error-prone — but it delivers what the dev asked for. Good when there are only 1-2 such scopes.
2. **Generate with the closest supported type as a placeholder and flag the gap.** Report it in the run output:

   > Scope **Audit trail** wanted a Document Flowchart, but the builder doesn't render that yet. I generated a Process Flowchart approximation on that page as a placeholder. The Document Flowchart version is queued — reply with `daemonstrate: build document-flow support` if you want it next.

3. **Skip the scope's page and log the gap.** Useful when hand-writing isn't worth the cost and an approximation would mislead. The overview still includes the scope.

Whichever path you take, write it into the run report so the choice is auditable.

### Content rules by layout

**Single-page System Flowchart (portfolio):**
- Header: project name + one-line pitch (pulled from README H1 + first paragraph, or `package.json` `description`).
- One dashed pastel container per scope, stacked vertically.
- Inside each container: 3-6 **capability pills** (not file names). Examples: "Google OAuth + cookie auth", "SMS intake via Twilio", "Offline SQLite + delta sync".
- Tech badge row along each container's top edge.
- 4-8 cross-scope edges showing headline data flows (labeled: "JWT cookie", "REST /api", "SMS webhook", etc.).
- Leave whitespace. A recruiter spends under 15 seconds.

**Hybrid (overview + drill-down):**
- Overview is intentionally thin: scope headers, tech badges, 3 headline node teasers, "View N components →" CTA, plus cross-scope edges. Each scope container on the overview is clickable and links to its detail page.
- Each detail page shows content appropriate to its `diagramType`. For System Flowchart: the full `key_nodes` grid, all internal edges, and a **"Connects to" sidebar** with clickable pills jumping to linked scope pages. For Process Flowchart: ordered steps + decisions. For Workflow: actor lanes + tasks. Etc.
- File paths render below node labels in smaller text. Hover for long-form `detail`.
- Edges labeled with *what* flows (HTTP verb + path, event name, DB table).
- Overview and detail pages share IDs and palette so a reader switching back and forth never loses orientation.

## Phase 4 — Post-run: hook, state, user edits

### Offer the post-commit hook (first run only)

Ask once:

> Want me to install a post-commit hook that runs Daemonstrate after each commit? It'll use incremental mode, so cost per commit is low.

If yes, run `bash "$SKILL_DIR/scripts/install-hooks.sh" "$TARGET_REPO"`. The script:
1. Backs up any existing `.git/hooks/post-commit` to `.post-commit.bak`.
2. Writes a new hook that calls `claude -p "..."` (natural-language prompt matching this skill's description) in the background, non-blocking, with a lock file to prevent overlap.
3. Makes it executable (`chmod +x`).

Only install into `$TARGET_REPO`, never into `$OUT_DIR` when they differ. If the dev declines, note in the state so we don't ask again.

### Preserving user edits

Before overwriting any `architecture-*.drawio`, check:

```
mtime($OUT_DIR/architecture-*.drawio) > scopes.identifiedAt + 60s ?
```

If yes, the diagram has been hand-edited. Ask before overwriting:

> `architecture-detailed.drawio` looks hand-edited since last generation. Overwrite, or save my proposed version as `architecture-detailed.proposed.drawio` for you to merge manually?

Default to the non-destructive option on unclear input. Non-interactive runs always save as `.proposed.drawio` and log the conflict.

### Run report

Use the report shape that matches the invocation pattern.

**First run (whole-repo):**

```
Daemonstrate: first run
- Detected stack: Node/Express 5, React 19, Postgres 16, Expo mobile
- Scopes identified: Frontend, Mobile, Backend, Data, Jobs, Integrations, Infra, Send-to-IDE (feature slice)
- Per-scope types + overlays:
    Frontend     → System Flowchart
    Mobile       → System Flowchart
    Backend      → System Flowchart
    Data         → System Flowchart
    Jobs         → Process Flowchart          (overlay: data_state — shows where clips morph)
    Integrations → Data Flow Diagram
    Infra        → System Flowchart           (overlay: environment — prod/staging/local)
    Send-to-IDE  → Swimlane                   (overlay: error_paths)
- Structure: hybrid
- Plain-English translation: 7 scopes translated (45 labels, 12 edges, 38 node details)
- Output: docs/architecture-detailed.drawio (1 overview + 8 scope pages × 2 audience tabs = 18 tabs)
- Catalog: docs/.daemonstrate-scopes.json
```

**Incremental (whole-repo refresh after a commit):**

```
Daemonstrate: incremental update
- Changed since 7ec88ef: Frontend (4 files), Backend (2 files)
- Refreshed scopes: Frontend, Backend
- Sub-component cache invalidated: be-authmw (file changed) — 1 level
- Plain-English cache: 3 labels re-translated, 42 byte-identical reuses
- Types unchanged; other scopes untouched.
- Output: docs/architecture-detailed.drawio (pages 2 and 4 regenerated, both audience tabs)
```

**Interactive teaching mode (Phase 0.5):**

```
Daemonstrate: teaching mode session
- Trail: Backend API → Auth middleware → verifyAccessToken → JWT signature check (level 3)
- New drill-down probes this session: 3 (Auth middleware, verifyAccessToken, JWT signature check)
- Plain-English translations added: 14 labels across the trail
- Diagrams rendered:
    docs/drilldown-backend.drawio                                    (System Flowchart)
    docs/drilldown-backend-authmw.drawio                             (Process Flowchart — focused)
    docs/drilldown-backend-authmw-verifyaccess.drawio                (Process Flowchart — deep dive)
    docs/drilldown-backend-authmw-verifyaccess-jwtcheck.drawio       (Process Flowchart — terminal)
- Each file has paired Technical (T) and Plain-English (P) tabs, Plain-English first.
- Session probe budget used: 3 AI calls (drill-down) + 14 (translations) = 17 total
- Catalog updated: sub_components trees persisted for every explored node
```

## References

- `references/drawio-format.md` — minimal XML cheatsheet + copy-paste group-container / node / edge fragments + multi-page link syntax. Needed when hand-writing pages for builder-unsupported diagram types.
- `references/light-theme-palette.md` — the palette + per-scope accent assignments.
- `references/layer-schema.md` — the JSON contract a scope-explorer subagent must return (the field still uses `layer` names for back-compat; treat as "scope schema").

## Scripts

- `scripts/drawio_builder.py` — graph spec JSON → `.drawio` files. Renders: **System Flowchart** (`--out-portfolio` / `--out-detailed`), **Process Flowchart** (`--out-journey`), **Workflow Diagram** (inside `--out-detailed`), **Swimlane Flowchart** (`--out-swimlane`), **Data Flow Diagram** (`--out-dfd`), plus the overlay framework (color_fill / edge_style / row_shading / badge / opacity) consumable by Swimlane + DFD. Document Flowchart / EPC / Influence Diagram remain hand-write for now — extending the builder for those is a deliberate next-pass task; the mutate-in-place overlay helpers already work on any Cell list.
- `scripts/install-hooks.sh` — installs the post-commit hook in the target repo.
- `scripts/post-commit.sh` — the hook body (copied into `.git/hooks/post-commit`).

---
> Source: [wyofalcon/daemonstrate](https://github.com/wyofalcon/daemonstrate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
