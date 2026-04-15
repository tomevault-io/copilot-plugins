## variant-viewer

> Evo Variant Effect Explorer. Svelte 5 + DuckDB web app for visualizing Evo2 ClinVar probe predictions.

# variant-viewer (EVEE)

Evo Variant Effect Explorer. Svelte 5 + DuckDB web app for visualizing Evo2 ClinVar probe predictions.

## Three Components (strict boundaries, test-enforced)

```
┌──────────────────────────────────────────────────────────────────┐
│ COMPONENT 1: PIPELINE (merge.py)                                 │
│ Inputs:  ARTIFACTS/{version}/, data/variants.parquet,            │
│          data/heads.feather                                      │
│ Output:  builds/{version}/scores.parquet + heads.feather +       │
│          config.json                                             │
│ Rule:    ONLY place that filters columns (quality + parents).     │
│          NO display logic, NO viewer concepts.                   │
│          VUS scores are OPTIONAL (warn, don't crash)             │
├──────────────────────────────────────────────────────────────────┤
│ COMPONENT 2: BUILD (transform.py, build.py)                      │
│ Inputs:  builds/{version}/scores.parquet + heads.feather +       │
│          config.json                                             │
│ Output:  variants.duckdb, statistics.json, heads.json            │
│ Rule:    Adds display strings + stats. NEVER drops columns.      │
│          Asserts width >= input width.                            │
├──────────────────────────────────────────────────────────────────┤
│ COMPONENT 3: SERVE (serve.py) + FRONTEND                         │
│ Inputs:  DuckDB + heads_config from global_config                │
│ Output:  flat JSON API + UI                                      │
│ Rule:    Dumb pipe. No data transformation. No data/ reads.      │
└──────────────────────────────────────────────────────────────────┘
```

**Boundary enforcement**: `tests/test_architecture.py` enforces:
- Import boundaries (merge doesn't import transform, etc.)
- Source-of-truth rules (transform/build never drop columns, no name-pattern derivation)
- No format workarounds (no isinstance checks, no legacy fallbacks)
- merge.py is the ONLY file that filters columns

## Key files

```
cli.py              5 commands: train, extract, build, serve, ls
merge.py            Pipeline → builds/ boundary (Component 1)
transform.py        Config-driven display + stats (Component 2, zero internal imports)
build.py            Parquet → DuckDB + neighbors + UMAP (Component 2)
serve.py            DuckDB → flat JSON API (Component 3, dumb pipe)
prompts.py          Claude interpretation: build_prompt(flat, heads_config)

paths.py            Path constants — ALL paths must come from here or CLI flags

pipeline/sequence/  Sequence probe: train, extract, eval, finalize
pipeline/token/     Token probe: train, extract (with inline eval), finalize, build_labels
frontend/           Svelte 5 + Vite + ECharts (Component 3)
```

**Deleted modules** (do not recreate):
- `variant_types.py` — was an unnecessary intermediate type. `build_prompt` takes flat dict directly.
- `display.py` — display names live in `data/heads.feather` column `display`.
- `pipeline/token/positions.py` — position selection is now inside `extract.py` (`select_positions`).
- `pipeline/token/eval.py` — eval is now inline in `extract.py`, per shard. No separate eval step.

## Column naming (the contract)

- `ref_{head}` / `var_{head}` — disruption (token probe, ref vs var allele at argmax position)
- `maxpos_{head}` — genomic distance (bp) where each head's max |delta| occurs
- `eff_{head}` — effect (sequence probe, variant-level predictors)
- `gt_{head}` — ground truth database annotations (nullable)
- `pred_{head}` — categorical predictions (aa_swap, consequence)
- `pathogenicity` — overall score (derived from eff_pathogenic)

No other prefixes. `score_`, `ref_score_`, `var_score_` are **deprecated**.

## Token probe scoring contract (extract.py)

**One canonical representation: per-head probabilities.** No mixed conventions.

| Head type | Output | Columns |
|-----------|--------|---------|
| Binary (n_classes=2) | P(class=1) | 1 per head |
| Categorical (n_classes>2) | full softmax (all classes including "none") | n_classes per group |

`logits_to_probs` is the single scoring function. Binary heads (shape==2) → P(class=1). Categorical heads (shape>2) → full softmax. All class names come from `categorical_groups` in the probe config (written by train.py from heads.feather). 367 probability columns total (262 binary + 89 annotated members + 16 "none" classes).

**Loss function** (`multihead_loss_v3`):
- Equal weight per head — no `log(2)/log(K)` scaling. CE gradient norms are naturally balanced (~1.4× across class counts).
- Groups by `(kind, n_classes)` for batching, but weighted by `len(members)` per group so each HEAD gets equal gradient.
- Hierarchical children processed individually, masked to parent-positive positions.

**Position selection** (`select_positions`):
- K=6: site + 5 top positions by L2 importance
- **Site detection**: uses `dist_from_var == 0`, NOT hardcoded tensor indices. All dist=0 positions excluded from topk.
- **Per-head argmax**: `aggregate_max` returns ref/var/dist at each head's max |delta| position (`maxpos_` columns). Captures sharp single-head disruptions that L2 misses.
- **Categorical importance**: `max(|member_delta|)` per group for L2 reduction.
- **Distances precomputed**: dict lookup by variant_id, no polars joins in the batch loop.

**Eval** (inline in extract.py, per shard):
- No separate eval step or `token_scores/` dataset (was 124 GB).
- Each extraction shard computes per-head metrics from test variants, writes `eval_shard_{id}.json`.
- Finalize aggregates shards with n-weighted mean ± std.
- Categorical eval uses full n_classes argmax (including "none").

**Hierarchy** (BFS topological sort):
- `apply_hierarchy` multiplies P(child) *= P(parent). Pairs sorted root→leaf via BFS, not by column index.

## Coordinate convention (0-based internally, +1 for display)

**All genomic positions are 0-based internally.** The harvest converts VCF 1-based to 0-based (`POS - 1`). The `pos` column and `variant_id` both use 0-based coordinates.

**External services and display use 1-based.** The frontend adds `+1` when:
- Building gnomAD URLs: `v.pos + 1`
- Building UCSC URLs: `v.pos + 1`
- Displaying coordinates to users: `v.pos + 1`

**Never use bare `v.pos` in a URL or display string.** Tests in `test_indexing.py::TestCoordinateConvention` enforce this — they scan VerdictCard.svelte for any `v.pos` in URLs without `+ 1`.

Column renames (variants.parquet → DuckDB) are in `data/config.json` `renames`:
```
gnomad_af → gnomad, vep_hgvsc → hgvsc, vep_hgvsp → hgvsp,
vep_exon → exon, vep_loeuf → loeuf, vep_domains → domains
```

## Type contracts (DuckDB → serve → frontend)

**The frontend calls `JSON.parse()` on array columns** (see `api.ts:92`). These columns MUST be stored as JSON strings in DuckDB, not native Polars lists:

| Column | DuckDB type | Must be |
|--------|-------------|---------|
| `neighbors` | VARCHAR (JSON string) | ✓ correct |
| `acmg` | **VARCHAR[] (native list)** | ✗ must be JSON string |
| `submitters` | **VARCHAR[] (native list)** | ✗ must be JSON string |
| `clinical_features` | **VARCHAR[] (native list)** | ✗ must be JSON string |
| `domains` | **STRUCT[] (native list)** | ✗ must be JSON string |

**Rule**: serialize all list/struct columns with `orjson.dumps().decode()` before DuckDB insert. Never store Polars native lists — the frontend can't handle them.

**prompts.py must handle the actual DuckDB return type** for each column. Currently `isinstance(domains_raw, str)` is always False because DuckDB returns `list[dict]`. Fix: handle both `str` and `list` until the type contract is fixed.

## Registry contract (data/heads.feather)

`data/heads.feather` is the single source of truth for head definitions. These invariants are test-enforced:

- **`role=probe`** — trained head. MUST have labels and produce eval metrics. If it doesn't, it's a phantom head — remove it or change its role.
- **`role=member`** — ME group input column (e.g., `amino_acid_A`, `region_CDS`). MUST have `me_group` set. Not trained individually — `build_labels` collapses members into a categorical parent.
- **`role=display`** — metadata column. Not trained, shown in viewer.
- **`kind`** in feather MUST match the loss function used during training. If the probe trained a head as `continuous` (soft-binned), the feather must say `continuous`, not `categorical`.
- **Adding/removing/changing heads** in feather requires re-running `build_labels.py` and retraining. The labels and the registry must stay in sync.

## Path contract

**No hardcoded absolute paths.** All paths come from `paths.py` constants or CLI flags.

- `paths.py` defines: `ARTIFACTS`, `DECONFOUNDED`, `DATA`, `VARIANTS`, `HEADS`, `CONFIG`
- Pipeline scripts (`build_labels.py`, `extract.py`, etc.) must accept paths as CLI arguments, not hardcode them
- `cli.py extract` must pass `DECONFOUNDED` (the activation storage) as `--activations`, NOT bare `ARTIFACTS` (the parent directory)
- Shard counts in `cli.py` (`--array=0-N`) must match the finalize step's `--n-shards` default

**Prefer CLI flags over defaults.** When a script needs a path (dataset, activations, output dir), make it a required or defaulted CLI flag. This way changes propagate through the CLI layer rather than hiding in module-level constants.

## Serve-layer contract

- **DuckDB write mode**: If serve.py caches interpretations (INSERT), it must NOT open the DB with `read_only=True`
- **prompts.py types**: Must handle all DuckDB return types as-is (`list`, `dict`, `str`, `float`, `None`). Never assume a column is a specific type without checking.
- **No data/ reads at serve time**: prompts.py must not import loaders or read `data/`. All config comes from `heads_config` (DuckDB `global_config` table).
- **No dead column references**: Every `flat.get("X")` in prompts.py must reference a column that exists in DuckDB. Dead references silently drop context from interpretation prompts.

## Data sources

- `data/heads.feather` — single schema: one row per head/metadata column (~721 rows, 18 cols). Categories: `metadata`, `disruption`, `effect`. Roles: `probe`, `member`, `display`. Has: display, group, curated, predictor + predictor config, exclusion flags. Includes `{group}_none` members for each categorical group (e.g., `amino_acid_none`).
- `data/config.json` — global build config (renames, display mappings, thresholds, bins, curated group lists). No per-head data. `gnomad_bins` must cover the full AF range [0, 1] with no gaps.
- `data/variants.parquet` — variant metadata + annotations. merge.py reads only the columns it needs (explicit `metadata_cols` set + predictor heads). If a frontend feature needs a new annotation column, add it to `metadata_cols` in merge.py.

## Token label pipeline (build_labels.py)

**Annotator columns ≠ disruption heads.** The annotator produces raw per-position annotations. `build_labels.py` maps them to probe heads. This is a two-source merge:

```
Source 1: Annotation parquets (per-chromosome)
  → Raw binary columns: amino_acid_A, amino_acid_C, region_CDS, ...
  → ME groups consolidated into categorical: amino_acid (21 classes incl. none), region (5), ...
  → ~80 binary heads + ~16 categorical groups (each with a "none" class from heads.feather)

Source 2: Genome-wide tracks (safetensors, per-chromosome)
  → Chromatin state heads: chromhmm_*, chipseq_*, atacseq_*
  → Conservation: phastcons, phylop
  → Continuous: gc_content, cpg_density, signal/breadth tracks
  → ~200 binary + ~40 continuous heads

build_labels.py merges both → tokens.safetensors (~278 heads, 26 GB)
```

**Do NOT expect a 1:1 match** between annotator parquet columns and `data/heads.feather` disruption head names. The annotator produces ME group *members* (e.g., `amino_acid_A`); the feather lists both the *parent* categorical head (`amino_acid`) and all members including `amino_acid_none`. The `_none` members have no annotation data — they represent "not in this category" and are derived by `collapse_me_group`. The tracks produce heads that don't appear in the annotation parquets at all.

**Effect heads are separate.** Token labels are disruption-only. Effect heads come from the sequence probe (trained on `data/deconfounded-full.parquet` labels, not per-position annotations).

## Interpretation pipeline

```
DuckDB flat row → build_prompt(flat, heads_config) → Claude API → cached in DuckDB
```

`build_prompt` extracts ref_/var_/eff_/gt_ columns directly from the flat dict using heads_config. No intermediate types. Display names from `heads_config["heads"][h]["display"]`. Curated/predictor flags from heads_config.

## Code conventions

- **No inline imports** — all imports at the top of the file
- **loguru** for all logging, never `logging` stdlib
- **rich.progress** for progress bars, never `tqdm`
- **polars** not pandas, **torch** not numpy, **orjson** for JSON serialization
- **uv run** for everything, never `.venv/bin/python` or `pip`
- **No intermediate types** — flat dicts are the contract, column names are the schema
- **No serve-time disk reads** — prompts.py must not import loaders or read data/. All config comes from heads_config (DuckDB global_config).
- **No hardcoded paths** — use `paths.py` constants or CLI flags. Never `Path("/mnt/...")`
- `data/heads.feather` is the single schema source. No hardcoded column sets (old META_COLS pattern is dead).

## Tests

Tests enforce every documented rule. **Run `uv run pytest tests/` before any PR.**

- `tests/test_contracts.py` — data contracts: column names, type compatibility, config plumbing, serve call chain, head consistency, DuckDB roundtrip
- `tests/test_architecture.py` — component boundaries, column prefixes, display-readiness, inline imports, conventions, CLI path contracts
- `tests/test_data_integrity.py` — eval coverage, phantom heads, registry drift, label consistency, build output integrity, shard count consistency
- `tests/test_prompts.py` — prompt building with flat dicts, type handling for DuckDB native types
- `tests/test_column_contract.py` — deprecated prefixes, ME group members
- `tests/test_head_registry.py` — ME groups, hierarchy
- `tests/test_heads_config.py` — config required keys, gnomad bin coverage, predictor config
- `tests/test_indexing.py` — head ordering, label offsets
- `tests/test_labels.py` — token label construction
- `tests/test_loss.py` — probe loss functions
- `tests/test_user_experience.py` — end-to-end variant lookup, JSON field types, display strings

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goodfire-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
