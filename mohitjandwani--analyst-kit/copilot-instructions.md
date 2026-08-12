## analyst-kit

> **Analyst Kit is not an application — it is a packager and distributor of *skills* for

# Analyst Kit — equity-research skills for AI agents

**Analyst Kit is not an application — it is a packager and distributor of *skills* for
AI coding agents** (Claude Code, Codex, and any runtime that can read a folder of
instructions). A "skill" is a self-contained folder under `skills/<name>/` whose
`SKILL.md` holds agent instructions, optionally alongside runnable `scripts/`,
`references/`, `templates/`, and `assets/`. The skills are the product; the Node code
under `src/` is plumbing that catalogs, resolves, and installs them.

There are two ways to consume the skills:

1. **Claude Code / Cowork plugin** — add the marketplace, install a persona plugin.
2. **Node installer** (`bin/analyst-kit.js` → `src/`) — copy skills into any runtime.

This file is the entry point for an AI agent working in this repo. If you only want to
*use* the skills, read **[Quick start](#quick-start--install-the-skills)**. If you are
*modifying* the repo, read **[Architecture](#architecture--how-a-skill-becomes-installable)**
onward. See also [`README.md`](README.md) (end-user docs), [`CLAUDE.md`](CLAUDE.md)
(contributor rules), and [`compatibility.md`](compatibility.md) (per-runtime behavior).

---

## Quick start — install the skills

### Option A · Marketplace plugin (primary — no clone, no Node)

Claude Code and Claude Cowork install the **single `analyst-kit` plugin** from the
same marketplace. It bundles every skill, the `research-auditor` subagent, and a
SessionStart runtime hook.

```
/plugin marketplace add mohitjandwani/analyst-kit
/plugin install analyst-kit@analyst-kit
```

In **Cowork** (desktop app), no terminal:

1. **Customize → Plugins → Add ▾ (top right) → Add marketplace.**

   ![Cowork — Plugins panel: Add ▾ → Add marketplace](docs/images/cowork-1-add-marketplace.png)

2. In **Add marketplace**, enter and select your repo: `mohitjandwani/analyst-kit`.

   ![Cowork — Add marketplace: enter mohitjandwani/analyst-kit](docs/images/cowork-2-enter-repo.png)

3. Add the **analyst-kit** plugin, then enable **Settings → Capabilities → Code execution**.

### Option B · Node installer (any runtime)

One command installs *all* skills into a runtime and wires them into the agent's
system/common prompt. Needs only **Node ≥ 18**:

```bash
npx github:mohitjandwani/analyst-kit claude-code     # or: codex · openclaw · cowork
```

From a clone, the same flows run through the bundled CLI (the package isn't published to npm
yet — the name `analyst-kit` is available — so invoke the file directly, not `npx analyst-kit`):

```bash
node bin/analyst-kit.js list                                              # browse skills + personas
node bin/analyst-kit.js claude-code                                       # install ALL skills (user scope)
node bin/analyst-kit.js install <skill|persona> --platform claude-code    # install one + its deps
node bin/analyst-kit.js install <skill> --platform codex --scope project --dry-run
node bin/analyst-kit.js doctor --platform claude-code                     # check runtimes + API keys
node bin/analyst-kit.js uninstall <skill|persona> --platform claude-code
```

Add `--scope project` to install into the current project (`./.claude/skills`, …)
instead of your home directory.

### API keys

The plugin's `userConfig` prompts for keys at enable time and stores them in the OS
keychain; the SessionStart hook bridges each non-empty value into `$AK_HOME/.env`
(which the per-skill preamble sources), so scripts read `os.environ[...]` unchanged.
**`FMP_API_KEY` is required**; `FINMIND_TOKEN` and `SERPAPI_API_KEY` are optional
(add them in the config UI anytime, or let the per-skill onboarding ask when the
skill runs and soft-disable until provided). On Codex/Node the onboarding prompts and
writes the same `.env`.

| Variable | Used by | Required | Get it |
|----------|---------|----------|--------|
| `FMP_API_KEY` | `financialmodellingprep`, `company-wiki`, `company-universe-manager` | **Yes** — plugin config | <https://site.financialmodelingprep.com/developer/docs> |
| `FINMIND_TOKEN` | `finmind` | Optional (Taiwan) | <https://finmindtrade.com/> (free) |
| `SERPAPI_API_KEY` | `market-intelligence` | Optional (Google Trends) | <https://serpapi.com/> (free tier: 100/mo) |
| `SEC_EDGAR_UA` | `sec-filings`, `13f-analysis` | Auto-generated | — |

`13f-analysis` and `sec-filings` read SEC EDGAR directly with no key — `SEC_EDGAR_UA`
is auto-generated per install from a one-time user id (`analyst-kit-setup
ensure-identity`; `edgar.py` derives the UA from it). Skills that run code bootstrap
their own dependencies on first use; **Python** and **Bun** are per-skill
prerequisites the installer does not install for you.

### Verify it loaded

Ask a trigger phrase — e.g. *"deep dive on NVDA"* — and the matching skill loads.
Agents discover skills by `name` + `description` only, so triggers live in the
description's `Triggers:` clause.

---

## Available skills

18 skills split into **capabilities** (one atomic job — a data source, an engine, a
deliverable, or reusable knowledge) and **workflows** (an engagement entry point that
orchestrates capabilities via `requires:`). Every skill depends on the
`analyst-kit-core` runtime (auto-installed); the **Needs** column lists *additional*
runtimes, API keys, and skill dependencies.

### Workflows — engagement entry points

| Skill | What it does | Needs |
|-------|--------------|-------|
| **single-stock-deep-dive** | Forensic deep dive on one stock: thesis, valuation, catalysts, variant perception, value-chain adjacencies | `wiki-builder` |
| **thematic-investing** | Map a theme into an investable value chain — who benefits, where value accrues, what's mispriced | `company-universe-manager` |
| **technical-analysis** | Disciplined TA with concrete entry/exit levels: regime classification, confluence stack, ATR-based stops/sizing | Python · `charting` |
| **company-wiki** | Build a multi-page company-research wiki (overview, financials, model, competitors, citations) as a deployed web app | `FMP_API_KEY` · `wiki-builder`, `company-universe-manager` |

### Capabilities — data sources, engines, deliverables, knowledge

| Skill | What it does | Needs |
|-------|--------------|-------|
| **analyst-playbook** | How to structure any analysis *before* fetching a number: pick the deliverable, align fiscal calendars, normalize units, route series, apply per-sector conventions | — |
| **13f-analysis** | Fetch & read U.S. institutional **13F-HR** holdings from SEC EDGAR as a normalized, ranked CSV | Python (stdlib) |
| **sec-filings** | Fetch & read U.S. SEC filings (10-K/10-Q/8-K, any form) with ticker→CIK resolution + BM25 search | Python (stdlib) |
| **financialmodellingprep** | Call the Financial Modeling Prep REST API — prices, news, profiles, screener, statements, transcripts | Python · `FMP_API_KEY` |
| **finmind** | Pull Taiwan (TWSE/TPEx) market data — prices, revenue, financials, dividends, institutional flows | Python · `FINMIND_TOKEN` |
| **market-intelligence** | Nowcast a quarter / predict a revenue segment from Google Trends search interest (via SerpAPI) | Python · `SERPAPI_API_KEY` |
| **company-universe-manager** | Own a watchlist of companies **and their key dates** (earnings, ex-div, AGMs): roster CRUD, daily monitor, daily brief | Python · `financialmodellingprep`, `reporting` |
| **analyzing-financial-statements** | Calculate & interpret financial ratios from statement data, with industry benchmarking | Python (stdlib) |
| **creating-financial-models** | DCF valuation, M&A accretion/dilution, sensitivity analysis, scenario planning | Python · numpy/pandas |
| **charting** | Financially-correct charts: Python/Polars normalizes data → TypeScript emits Highcharts options + a self-contained HTML page | Node · Python |
| **reporting** | Assemble charts, tables, and text into a branded PDF — A4 report or 16:9 deck | Node · `charting` |
| **wiki-builder** | Serve any folder of markdown as a navigable browser wiki (sidebar, ToC, frontmatter chips, ECharts) | Bun |
| **data-analysis** | End-to-end analysis of a structured dataset (CSV/JSON/Excel/SQL) with reproducible code | — |
| **analyst-kit-core** | **Shared runtime** (auto-installed): the `~/.analyst-kit` data home, config store, local usage analytics with opt-in telemetry, daily update checks, and a per-user learnings log every skill reads and appends to | — |

---

## The plugin (`plugins/analyst-kit/`)

One self-contained plugin (under `plugins/`, advertised by `.claude-plugin/marketplace.json`)
bundles **all 18 skills**, the `research-auditor` subagent (`agents/`), and a SessionStart
runtime hook (`hooks/`). Its `skills/` are a generated, committed copy of the top-level
`skills/` source of truth (`npm run build:plugin`; CI fails on drift) — required because a
marketplace plugin is installed into an isolated cache and can't reach files outside its own
folder. Run `node bin/analyst-kit.js install analyst-kit --dry-run` to see the full closure.

---

## Repository layout

```
skills/<name>/SKILL.md       the product — agent instructions + frontmatter (single source of truth)
skills/<name>/{scripts,references,templates,assets}/   optional runnable/support files
skills/analyst-kit-core/     the shared runtime skill (bin/ scripts, templates/, references/)
plugins/<persona>/.claude-plugin/plugin.json   persona bundles (must list each skill's full closure)
.claude-plugin/marketplace.json                marketplace manifest
bin/analyst-kit.js           CLI entry point → src/cli.js
src/                          the install pipeline (frontmatter → registry → resolve → install/adapters)
scripts/                      validate.js, build-registry.js, sync-preamble.js, build-routing-table.js
registry.json                GENERATED catalog (build from frontmatter; never hand-edit)
VERSION                       single source of truth for the release version
.env.example                 every declared API key must appear here
```

---

## Architecture — how a skill becomes installable

**Skill frontmatter is the single source of truth.** The YAML block atop each `SKILL.md`
drives `registry.json`, the installer's dependency resolution, and the validator. Never
hand-edit `registry.json` — edit frontmatter and run `npm run build:registry`.

The pipeline (`src/`):

- **`frontmatter.js`** — a minimal, purpose-built YAML parser (*not* general). It handles
  only the shapes the contract allows (scalar/block `description`, inline or block
  `requires`/`env`). Frontmatter outside those shapes silently misparses — stay in contract.
- **`registry.js`** — `scanSkills()` walks `skills/`, parses frontmatter, and infers
  `runtime` from disk (`package.json` → `bun`; a `scripts/*.py` → `python`; else `none`).
  `getSkills()` scans **live disk first**, falling back to `registry.json` only if the
  tree is unavailable, so the CLI always reflects reality.
- **`resolve.js`** — `resolveClosure()` walks the `requires` graph depth-first
  (dependencies-first, cycle-detecting). Personas are read from the **plugin manifests**,
  which are the source of truth for what a persona bundles.
- **`install.js` + `adapters/`** — `install()` resolves the closure, then each platform
  adapter (`claude-code.js`, `codex.js`, …) decides install dir, env-file location, and
  `write()` behavior. `copy.js#copyTree` recursively copies, skipping `node_modules`,
  `.venv`, `__pycache__`, `.git`, `tests`. Codex has no native skill format, so its
  adapter also emits a slash-prompt under `prompts/`.
- **`env.js`** — unions the `env:` keys across a closure, resolves them from
  `process.env` + an `.env` file, prompts for the rest, and persists with `chmod 600`.
- **`paths.js`** — repo-root-relative paths and `EXCLUDED_SKILLS` (skills present on disk
  but withheld from the shippable registry/plugins — reported as "skipped", not a failure).

---

## The skill contract (enforced by `scripts/validate.js`)

- `name` **must equal the folder name** and be kebab-case.
- `type` is `capability` (a requirable unit that may itself `require:` other capabilities)
  or `workflow` (an entry point that orchestrates capabilities). **The one dependency
  rule: nothing may require a workflow.** Dependency cycles are rejected.
- `description` ≥ 40 chars and **must contain a `Triggers:` clause** of natural-language
  phrases — agents see only `name` + `description`, so discoverability lives there.
- `SKILL.md` body must be non-empty; no **dependency-manager** dirs tracked by git
  (`node_modules`, `.venv`, `__pycache__`). Deliberately-vendored *first-party runtime
  assets* a skill must ship to function (e.g. `skills/charting/vendor/highcharts/`,
  inlined for offline PDF-safe rendering) are allowed and **should** be committed.
- Plugin manifests must reference existing skills **and include each referenced skill's
  full dependency closure** (a workflow without its capabilities fails validation).
- Every declared `env:` key must also appear in `.env.example`.

Skill frontmatter shape:

```yaml
---
name: 13f-analysis          # kebab-case; must equal the folder name
type: capability            # capability | workflow
description: >              # ≥40 chars; what it does + a "Triggers:" clause
  Fetch & read U.S. 13F-HR holdings … Triggers: "get the 13F for X", "what does <fund> own", …
requires: [analyst-kit-core]  # capabilities this builds on (never a workflow)
env: [FMP_API_KEY]          # API keys the skill needs (must be in .env.example)
---
```

---

## The `analyst-kit-core` runtime

Every shipped skill `requires: [analyst-kit-core]` and carries generated
`<!-- analyst-kit:preamble -->` / `<!-- analyst-kit:epilogue -->` blocks (onboarding,
analytics, update check, learnings). Per-user state lives in `~/.analyst-kit/`:

- `.env` — API keys (chmod 600, shared across projects)
- `config` — settings (`analyst-kit-core/bin/analyst-kit-config get|set|list`)
- `analytics/skill-usage.jsonl` — **local** usage log (skill, time, outcome, duration)
- `learnings.jsonl` — durable patterns/preferences the skills append, read back next run

Telemetry is **on by default (anonymous, opt-out)**: skill name, version, outcome, and
duration only — never repo names, paths, tickers, or content. Tiers: `community`
(default), `anonymous`, `off`. Opt out with
`analyst-kit-core/bin/analyst-kit-config set telemetry off`.

> ⚠️ **Never hand-edit between the `analyst-kit:preamble` / `analyst-kit:epilogue`
> markers.** Edit `skills/analyst-kit-core/templates/*.md.tmpl` and run
> `npm run sync:preamble`. The same script stamps the root `VERSION` into
> `analyst-kit-core/VERSION`, plugin manifests, and `package.json` — bump only the root
> `VERSION` file. New skills must `require: [analyst-kit-core]`.

---

## Build & validate commands

No build step, no JS test suite at the root — individual skills ship their own tests.

```bash
npm run validate         # lint skills + plugin manifests + preamble-sync check (what CI runs)
npm run build:registry   # regenerate registry.json from skill frontmatter
npm run check:registry   # CI check: fail if registry.json is stale (writes nothing)
npm run sync:preamble    # regenerate the analyst-kit-core blocks in every SKILL.md
npm run build:routing    # regenerate the routing table in compatibility.md
npm run test:integration # real per-platform installs + the path.win32 path-layer check

# Per-skill tests (run inside the skill folder):
bash skills/analyst-kit-core/tests/run.sh    # runtime (bin scripts, ~/.analyst-kit home)
cd skills/charting && bun test               # charting formatter + renderer
cd skills/finmind  && pytest                 # finmind (Python)
```

`npm run validate` + `npm run check:registry` are exactly what CI runs
(`.github/workflows/validate.yml`) on every push and pull request.

---

## Platform support

The skill runtime is **POSIX/bash**, so:

- **macOS / Linux** — fully supported.
- **Windows** — supported **only inside WSL2** (native PowerShell/cmd cannot run the bash
  runtime; Git Bash runs the scripts but without enforced `.env` permissions or
  sandboxing). This matches the agents themselves — Claude Code's sandbox and Codex's
  Linux mode both run on WSL2, not native Windows. `node bin/analyst-kit.js doctor
  --platform claude-code` warns when run on native Windows.

State paths resolve through `analyst-kit-core/bin/_analyst-kit-home.sh`, which honors an
explicit `AK_HOME` env override, then the `~/.analyst-kit/home-path` pointer, then the
`~/.analyst-kit` default — so a relocated data home works uniformly across the runtime.

---

## Conventions — making changes

- **Edit frontmatter, then regenerate.** Adding/removing a skill or changing frontmatter →
  `npm run build:registry` then `npm run validate`. Add the skill (with its dependencies)
  to the relevant `plugins/*/.claude-plugin/plugin.json` if it should ship in a persona.
- **Promoting a work-in-progress skill** → remove it from `EXCLUDED_SKILLS` in
  `src/paths.js` and rebuild the registry.
- **Adding an API key** → declare it in `.env.example` *and* the consuming skill's `env:`
  list, or validation fails.
- **Adding a platform** → add an adapter in `src/adapters/` and register it in
  `adapters/index.js`; the interface is `installDir(scope)`, `envFile(scope)`,
  `write(skill, scope)`.
- **Generated artifacts are off-limits by hand:** `registry.json`, the
  `<!-- analyst-kit:preamble/epilogue -->` blocks in each SKILL.md, and the routing table
  in `compatibility.md`. Edit the source (frontmatter, `templates/`) and run the matching
  generator.
- **Internal identifiers:** env vars use the `AK_` prefix (`AK_HOME`, `AK_POSTHOG_KEY`),
  and the charting renderer's inlined JS global is `AK` (`AK.fmt`). Keep these consistent
  when touching the runtime or the charting layer.
- **Commits:** Conventional Commits, one logical change per commit, stage files explicitly
  (never `git add -A`), never commit secrets. See [`CLAUDE.md`](CLAUDE.md) for the full
  commit rules.

---
> Source: [mohitjandwani/analyst-kit](https://github.com/mohitjandwani/analyst-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
