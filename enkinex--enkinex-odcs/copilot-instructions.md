## enkinex-odcs

> KCL library implementing the **Open Data Contract Standard (ODCS)

# enkinex-odcs

KCL library implementing the **Open Data Contract Standard (ODCS)
v3.1.0** as Governance-as-Code. Published; tracks the standard's JSON
schema (`odcs-json-schema-v3.1.0.json`).

## Repo map

| Path | Purpose |
|---|---|
| `odcs.k` | Root `DataContract` schema composing all modules |
| `common/` `catalog/` `contract/` `iam/` `quality/` `server/` | One KCL module per ODCS section |
| `test/*.yaml` | `kcl vet` fixtures validated against the schemas |
| `docs/library/odcs.md` | Generated schema reference (`just docs`) — regenerate on docstring change |
| `docs/schemas/` | Per-module design rationale |

## Commands

`just init` (kcl mod update) · `just fmt` · `just lint` · `just test` ·
`just docs` · **`just check` — the gate every change must pass** (fmt +
clean-tree + lint + test). Run `just fmt` and commit before `just check`.

## Standards

- Docstrings on every schema and field (they feed `just docs`): attribute
  line format, `required`/`optional` fidelity with the standard, inline
  `Examples:`.
- `check` rules for enums/constraints; mixins for repeated shapes
  (`common/`).
- Contributing rules: [CONTRIBUTING.md](CONTRIBUTING.md) — branch
  `<type>/<short-slug>`, Conventional Commits subset, squash-merge.


<!-- BEGIN GENERATED: enkinex-aiops/AGENTS.shared.md — do not edit here; run "just sync-opencode" in enkinex-aiops -->
## Shared enkinex rules

> GENERATED from enkinex-aiops `AGENTS.shared.md` (ADR-0005). Do not edit
> this block in a sibling repo — change the source in enkinex-aiops and run
> `just sync-opencode`.

Enkinex is an open-source **Semantic & Governance as Code** project: KCL
libraries that implement open standards (ODCS, ODPS, OKF) and platform
configuration surfaces (Databricks Asset Bundles) as typed, modular code.

### Git workflow (locked)

- Branch slug: `<type>/<short-slug>`; `type` ∈ `feat · fix · refactor ·
  docs · chore · test · infra · proj`; slug kebab-case, ≤6 words,
  imperative (e.g. `feat/output-port-retry-policy`).
- Commits: Conventional Commits subset `<type>: <imperative ≤72>`,
  `Refs: <TASK-ID>` footer naming the task delivered, no `Closes:`/
  `Fixes:`/`Resolves:` — issues are closed by hand after the squash merge,
  not as a side effect of a commit message (ADR-0006).
- **No repo-name scope.** A scope is optional and names a *module inside
  this repo* (`catalog`, `quality`, `trust`, `githooks`), never the repo
  itself: `feat(odcs):` inside enkinex-odcs says nothing the repository
  does not already say. Package-name scopes are a monorepo device; these
  are separate repos. The `commit-msg` hook rejects a redundant scope.
- **Never push, merge, or open PRs unless the user explicitly asks.** The
  iteration ends at a local commit. `gh` CLI is the only GitHub surface
  (ADR-0002): no GitHub MCP, no Actions, no Projects, no Releases.
  **Issues are open** (ADR-0006): read them freely, creating or editing one
  is a prompt, and `gh issue delete`/`transfer` are denied outright.
- Never force-push to `main`; never rewrite history.
- Before any repo edit: `git fetch origin`, confirm sync with `main`,
  create the branch. Commit at the end of the iteration.

### Mechanical enforcement

The rules above are enforced by git hooks in `.githooks/`, not by your
compliance: `commit-msg` checks the subject grammar and the `Refs:` footer,
`pre-commit` checks the enkinex remote and scans staged content for
credentials, `pre-push` checks the branch slug and refuses direct pushes to
`main` and history rewrites.

A second layer, `.agents/policy/guard.mjs`, covers what git hooks cannot see:
hook bypasses (`--no-verify`, `core.hooksPath` edits), `git add -A`, `gh pr
merge`, and reads of credential paths. One script; opencode, Claude Code and
Codex each call it through a pointer-only adapter.

- **Never pass `--no-verify`.** If a hook refuses, fix the cause.
- Stage explicit paths. `git add -A`, `git add .` and `git add -u` are denied.
- Hooks are inert until a clone is pointed at them. If
  `git config --get core.hooksPath` is empty, run
  `git config core.hooksPath .githooks` before committing.
- Unattended runs use the headless profile (`opencode.headless.json`), where
  push, rebase, PR creation and PR merge are denied outright rather than
  prompted. Launch through `scripts/opencode-headless.sh` in enkinex-aiops.

### Project lifecycle

**Planning is centralised and private.** Plans live in the sibling
`enkinex-pm`, one folder per repository — `../enkinex-pm/plan/<repo>/`
from a repo checkout — as small numbered task files. **A repo with no
local `plan/` is correct, not misconfigured**; do not create one, and do
not plan in the repo you are editing.

There is no `discovery/` stage. Analysis feeding a plan is an input to
planning and belongs in `enkinex-pm`, not beside the code.

`architecture/` stays at each repo root. ADRs record one-way decisions
only — procedural workflows are defined as executable artefacts (agents,
commands, loop tasks, plugin hooks), never as ADR prose (ADR-0004,
executable governance). A repo's ADRs are public with its code, so an ADR
citing a plan cites something the reader may not be able to open: say so
at the citation rather than leaving a path that resolves for nobody.

A commit's `Refs:` footer names the task it delivers by its **stable task
ID** — a project prefix plus the task's file number, `AIOPS-10`, `MGR-16`,
`PM-04` — resolving to `../enkinex-pm/plan/<repo>/<NN>-<slug>.md`.
Separate several with commas. Never write the path: the ID survives a file
being renamed or re-numbered, and a path encodes one machine's checkout
layout into permanent history. The ID resolves only for someone holding the
`enkinex-pm` clone — a cost this org has accepted deliberately.

Use `No-Plan-Ref: <reason>` when a commit advances no task. It is the
**correct footer, not a bypass** — repo hygiene, tooling and dependency
bumps legitimately advance no plan, and inventing a task ID to satisfy the
hook is the failure this escape hatch prevents.

`commit-msg` enforces both shapes. **Footers changed shape on 2026-08-13**
(AIOPS-10): commits before that date point at `plan/…` paths that no longer
resolve, commits after name task IDs. Both are readable; neither is
machine-resolvable across the boundary. History is not rewritten and old
footers are not errors — the hook validates only the message being written.

### Model tiers (OpenRouter)

| Tier | Models | Use |
|---|---|---|
| Free | `:free` suffixed IDs | **interactive, bounded lookups only** — never an agent in the loop |
| Mid | `moonshotai/kimi-k2`, `deepseek/deepseek-v3.2`, `google/gemini-3.5-flash` | code edits, docs, tests, exploration |
| Frontier | `moonshotai/kimi-k3` (default), `anthropic/claude-opus-5`, `openai/gpt-5.6` family | plans, reviews, ADRs |

Do not switch tiers silently; model pins change only via PR.

**No agent is pinned to the free tier, and that is the decision, not an
oversight** (AIOPS-12). The row above said "explore/triage", and the one agent
that took it at its word could not do the job: it failed to finish a broad
exploration step in 10 minutes on two occasions, it spends its output budget
reasoning before it answers, and it returns `502 ResourceExhausted` from the
upstream provider often enough to have done so during unrelated work. The
first is a prompt problem; the other two are not, and a step that never
returns is the most expensive failure an unattended run has — nobody is
watching, and it has to be killed.

Free is therefore for questions **you** ask, bounded, with a human reading the
answer and nothing depending on it arriving. Anything the loop runs is mid or
frontier. The evidence sits at the pin in `explore-enkinex.md`, so re-pinning
to free means arguing with it rather than rediscovering it.

### Code standards

- KCL libraries: one module per concern, docstrings on every schema and
  field (they feed `just docs`), `check` rules for enums/constraints,
  `kcl vet` fixtures under `test/`. Gate: `just check` (fmt + lint + test).
- Stage explicit paths only — never `git add -A` / `git add .`; skip
  anything that looks like a secret.
<!-- END GENERATED -->

---
> Source: [enkinex/enkinex-odcs](https://github.com/enkinex/enkinex-odcs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
