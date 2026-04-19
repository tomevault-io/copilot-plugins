## cartograph

> AI-assisted code context engine. Three-phase pipeline: map (index files), lens (select context), run (assemble + generate).

# cartograph Development Guidelines

## Project Overview

AI-assisted code context engine. Three-phase pipeline: map (index files), lens (select context), run (assemble + generate).

## Tech Stack

- Python 3.12+, Click (CLI), litellm (model calls), Pydantic (schemas)
- asyncio + aiohttp (concurrent I/O), xxhash (content hashing), tiktoken (token counting)
- pathspec (.gitignore parsing), PyYAML (config)

## Project Structure

```text
src/cartograph/          -- Main package (src layout)
  cli.py                 -- Click command group
  config.py              -- Three-layer config loading
  mapper/                -- Phase 1: code map generation
  lens/                  -- Phase 2: context selection
  assembler/             -- Phase 3: prompt assembly + budget
  docs/                  -- Auto-docs generation from code maps
  deps/                  -- Dependency analysis (no LLM, mechanical)
  dead/                  -- Dead code detection via AST (no LLM)
  style/                 -- Style consistency detection (LLM-assisted)
  search/                -- Semantic search via embeddings (numpy + litellm)
  experiment/            -- Automated experiment loop (propose, test, keep/discard)
  review/                -- Diff-aware code review via LLM
  lat/                   -- lat.md knowledge graph integration (parse, scan, context, bootstrap, sync)
  workspace/             -- Cross-repo workspace orchestration
  models/                -- litellm client + cost tracking
  cache/                 -- Content-addressed cache + index
tests/                   -- pytest test suite
  fixtures/              -- tiny-repo, mixed-repo, workspace, experiment-repo, lat-repo test data
  test_mapper.py         -- Scanner, schema, prompt tests
  test_lens.py           -- LensResult, query hash, selector tests
  test_assembler.py      -- Budget, context assembly tests
  test_cache.py          -- Cache store + index tests
  test_config.py         -- Config loading tests
  test_cli.py            -- CLI command tests (init, map, lens, run, status, docs, onboard, deps, dead, style, search, experiment, review, lat, workspace)
  test_experiment.py     -- Experiment loop: schema, git helpers, proposer, single cycle tests
  test_review.py         -- Review: diff parsing, source detection, schema tests
  test_dead.py           -- Dead code extractor, analyzer, confidence, renderer tests
  test_style.py          -- Style sampling, extraction, detection, diffing, renderer tests
  test_search.py         -- Searcher, fallback, index management tests
  test_lat.py            -- lat.md parser, scanner, graph, context, bootstrap, sync, review/experiment integration tests
  test_workspace.py      -- Workspace config, resolver, unified index, cross-repo search/lens tests
  test_deps.py           -- Dependency analyzer + renderer tests
  test_docs.py           -- Docs index, scope hashing, generation tests
  test_models.py         -- Cost tracking + tier selection tests
  test_integration.py    -- Full pipeline tests with mocked models
```

## Commands

```bash
uv run pytest                    # run tests (416 tests)
uv run ruff check src/ tests/   # lint
uv run ruff check --fix src/ tests/  # auto-fix lint
```

## CLI Contract

- All commands output JSON to stdout, progress/errors to stderr
- Exit codes: 0 success, 1 no results, 2 not initialized, 3 budget too small
- `cartograph run` streams raw model output (not JSON) to stdout
- `cartograph onboard --stdout` outputs raw Markdown (not JSON) to stdout
- `cartograph deps --stdout` outputs raw Markdown (not JSON) to stdout
- `cartograph dead --format mermaid` outputs raw Mermaid (not JSON) to stdout
- `cartograph dead` exit code 1 = no dead code found (clean bill of health)
- `cartograph style` outputs JSON to stdout; exit 0 = outliers found, 1 = no outliers/patterns
- `cartograph style --format markdown` outputs raw Markdown (not JSON) to stdout
- `cartograph search` outputs JSON to stdout; exit 0 = results found, 1 = no results
- `cartograph search --rebuild` rebuilds embedding index without querying
- `cartograph experiment` outputs JSON to stdout; exit 0 = experiments completed, 1 = no experiments, 2 = not initialized, 3 = pre-flight test failed
- `cartograph experiment --stdout` outputs raw Markdown (not JSON) to stdout
- `cartograph experiment` requires `--test-cmd` flag (no default test command)
- `cartograph review` outputs JSON to stdout; `--stdout` outputs Markdown
- `cartograph lat init` generates mechanical lat.md/ from code maps; exit 0 = created, 2 = not initialized, 4 = already exists
- `cartograph lat generate` uses LLM to produce concept-oriented lat.md/ with design rationale and cross-references
- `cartograph lat init --force` / `lat generate --force` additive merge (only adds new files, preserves existing)
- `cartograph lat sync` detects stale sections, orphaned refs, uncovered modules; exit 0 = issues found, 1 = in sync, 2 = not initialized
- `cartograph lat sync --apply` writes proposed changes to lat.md/
- `cartograph lat export --vault PATH` exports lat.md/ to Obsidian vault with converted links, frontmatter, tags, and MOC index
- When lat.md/ exists, `run`/`review`/`experiment` automatically include knowledge graph context (15% budget cap, stderr summary)
- `cartograph config` outputs JSON with provider, models, concurrency, budget
- `cartograph config --set KEY=VALUE` updates a single config field (dot notation)
- `cartograph config --provider NAME` switches provider (warns about model compat)
- `cartograph config --defaults` resets models to provider preset defaults
- `cartograph config --migrate` converts old per-tier config to new provider format
- `cartograph init` prompts for provider, API key, and per-phase models interactively
- `cartograph init --non-interactive` skips prompts, uses openrouter defaults
- `cartograph workspace init/add/remove/status` manages cross-repo workspaces
- All commands support `--repo`, `--cross`, `--no-cross` flags for workspace scoping
- Cross-repo mode auto-activates when `cartograph-workspace.yaml` is found (walk-up discovery)

## Key Patterns

- Content-addressed cache: `.cartograph/maps/<hash>.json`
- Atomic writes via temp file + rename
- Model phases: map (cheap), lens (cheap), docs (cheap), embed (cheap), style (cheap), ask (premium), generate (premium)
- Provider-based config: single provider + per-phase model assignments
- Known providers: openrouter, openai, anthropic, ollama, custom
- API key resolution: config file > CARTOGRAPH_API_KEY env > provider-specific env
- Config merge: defaults < `.cartograph/config.yaml` < CLI flags
- Cost tracking: append-only `.cartograph/costs.jsonl`
- Workspace: `cartograph-workspace.yaml` defines multi-repo workspaces
- Workspace data: `.cartograph-workspace/unified-index.json` tracks per-repo state
- Cross-repo: thin CLI orchestration; core modules remain workspace-unaware
- lat.md integration: auto-detected when `lat.md/` exists at project root; no runtime dependency on lat CLI
- lat.md section selection: embedding similarity (async) with keyword fallback (sync)
- lat.md context: inserted as "Knowledge Graph Context" block before code files in prompts
- @lat: backlinks: source files with `# @lat: [[section-id]]` get relevance boost in lens selection

## Code Style

- ruff with line-length 88, target py312
- All files start with two ABOUTME comment lines
- Pydantic models for all schemas
- asyncio for I/O-bound model calls

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

## Active Technologies
- Python 3.12+ + Click (CLI), litellm (LLM calls), Pydantic (schemas), xxhash (hashing) (002-auto-docs)
- JSON files in `.cartograph/docs/` + `docs/index.json` for state tracking (002-auto-docs)
- Python 3.12+ + Click (CLI), litellm (model calls), Pydantic (schemas), xxhash (hashing) (003-onboarding-guide)
- `.cartograph/docs/ONBOARDING.md` (output), `.cartograph/docs/index.json` (tracking) (003-onboarding-guide)
- Python 3.12+ + Click (CLI), Pydantic (schemas), xxhash (hashing) (004-dependency-graph)
- `.cartograph/docs/DEPENDENCIES.md` (output), `.cartograph/docs/index.json` (tracking) (004-dependency-graph)
- Python 3.12+ + Python `ast` module (stdlib), pathspec (glob matching), xxhash (caching), Pydantic (schemas), Click (CLI) (005-dead-code-detection)
- JSON files in `.cartograph/refs/<hash>.json` (extraction cache), `.cartograph/dead.json` (report) (005-dead-code-detection)
- Python 3.12+ + Click (CLI), litellm (model calls), Pydantic (schemas), xxhash (hashing), pathspec (glob matching) (006-style-consistency)
- JSON files in `.cartograph/style/` (patterns.json, report.json, baseline.json) (006-style-consistency)
- Python 3.12+ + Click (CLI), litellm (embedding API), numpy (vector math), Pydantic (schemas), pathspec (glob matching), xxhash (hashing for staleness detection) (007-semantic-search)
- `.cartograph/search/embeddings.npz` (numpy compressed), `.cartograph/search/metadata.json` (file path mapping + index hash) (007-semantic-search)
- Python 3.12+ + Click (CLI), litellm (model calls), Pydantic (schemas), PyYAML (config + workspace YAML), pathspec (glob), numpy (embeddings), xxhash (hashing), tiktoken (tokens) (008-cross-repo-workspace)
- YAML (workspace config), JSON (unified index), numpy .npz (combined embeddings), per-repo `.cartograph/` directories (008-cross-repo-workspace)
- Python 3.12+ + Click (CLI), Pydantic (schemas), PyYAML (config), litellm (model calls) (009-provider-model-config)
- YAML files (`.cartograph/config.yaml`) (009-provider-model-config)
- Python 3.12+ + Click (CLI), litellm (model calls), Pydantic (schemas), pathspec (glob matching) (010-diff-code-review)
- JSON output to stdout; no new persistent storage (uses existing `.cartograph/` cache) (010-diff-code-review)
- Python 3.12+ + Click (CLI), litellm (model calls via models/client.py), Pydantic (schemas), pathspec (glob matching) (011-auto-experiment)
- Git working tree (changes applied/reverted via subprocess git calls); no new persistent storage (011-auto-experiment)

- Python 3.12+ + Click (CLI), Pydantic (schemas), PyYAML (frontmatter parsing), numpy (embedding similarity) (012-latmd-integration)
- lat.md/ directory (read-only except for lat init/sync --apply); no new persistent storage (012-latmd-integration)

## Recent Changes
- 012-latmd-integration: Added lat.md knowledge graph integration (parser, scanner, graph, context, bootstrap, sync)
- 002-auto-docs: Added Python 3.12+ + Click (CLI), litellm (LLM calls), Pydantic (schemas), xxhash (hashing)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vaporeyes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
