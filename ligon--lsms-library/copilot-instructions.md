## lsms-library

> Harmonize the *interface*, not the data. Surveys differ in structure; the library provides a uniform API without discarding survey-specific detail.

# LSMS Library

## Design Philosophy
Harmonize the *interface*, not the data. Surveys differ in structure; the library provides a uniform API without discarding survey-specific detail.

## Core API
```python
import lsms_library as ll
uga = ll.Country('Uganda')
uga.waves          # ['2005-06', '2009-10', ...]
uga.data_scheme    # ['people_last7days', 'food_acquired', ...]
food = uga.food_expenditures()  # Standardized DataFrame
```

## Country Symlinks
Root-level symlinks (e.g., `Uganda → lsms_library/countries/Uganda`) are for convenience. Actual config is under `lsms_library/countries/`.

## DVC Repository Root
**The DVC repository is rooted at `lsms_library/countries/`, NOT the top-level repo.** DVC config, remotes, and credentials all live under `lsms_library/countries/.dvc/`. All `dvc` CLI commands (pull, push, status) must be run from `lsms_library/countries/` or they will fail with missing-remote/credential errors. The `get_dataframe()` fallback chain handles this automatically when scripts run from their normal `{Country}/{wave}/_/` working directory.

## Cache Behavior (v0.7.0+)

**Writes** are redirected to `data_root()` by `_resolve_data_path` in `local_tools.py`. Default location `~/.local/share/lsms_library/{Country}/var/{table}.parquet`; override with `data_dir` in `~/.config/lsms_library/config.yml` or `LSMS_DATA_DIR` env var (env var wins).

**Reads** as of v0.7.0: `load_dataframe_with_dvc` in `country.py` does a **best-effort cache read at the top of the function**, before consulting DVC at all. If `cache_path.exists()` and `LSMS_NO_CACHE` is not set, it returns the cached parquet directly. This applies uniformly to all 40 countries: the `if not stage_infos:` branch, the stage-layer branch, and the DvcException fallback all populate the same cache that the top-of-function read picks up next time.

Empirical numbers on Savio/Lustre with `.dta` files local (see `bench/results/`):

| Phase | Pre-v0.7.0 (write-only bug) | v0.7.0+ |
|---|---|---|
| Niger `household_roster` cold rebuild | ~16s | ~14s |
| Niger second call same process | ~7s | **~0.5s** |
| Niger second call fresh subprocess | ~16s (no speedup) | **~0.9s** |
| Uganda `household_roster` second call fresh subprocess | ~12s (no speedup) | **~1.2s** |

The 10–17× cross-process speedup is the headline win. Python import of `lsms_library` (~22s on Lustre, ~30s on first cold filesystem) is now the dominant cost of any fresh-session call.

**No staleness check.** v0.7.0 deliberately ships *without* automatic invalidation. Editing a wave's `data_info.yml`, a `_/{table}.py` script, or an upstream `.dta` file does *not* trigger a rebuild on the next call. Contributors must:

- set `LSMS_NO_CACHE=1` in the environment to bypass the top read for that session, OR
- run `lsms-library cache clear --country {Country}` to remove the parquet, OR
- pass `trust_cache=False` (the default; this still hits the v0.7.0 top read — see below).

**`trust_cache=True`** is a separate, narrower short-circuit at the top of `_aggregate_wave_data` that reads the parquet and then **still calls `_finalize_result`** (so kinship expansion, spelling normalization, and the `_join_v_from_sample` augmentation all apply on the way out). It bypasses the DVC stage check, the in-process ancillary caches, and the entire `load_dataframe_with_dvc` flow including the v0.7.0 top read. Effectively a stricter version of the v0.7.0 behavior for users who *know* the cache is fresh and don't want any cache existence checks beyond the parquet itself.

**`LSMS_BUILD_BACKEND=make`** dispatches `_aggregate_wave_data` directly to `load_from_waves` (`country.py:1830`), bypassing both the v0.7.0 top read and the DVC stage layer. **It does not write to or read from the data_root cache at all.** Every call rebuilds from source. The only writes that happen under `LSMS_BUILD_BACKEND=make` are via the CLI's explicit `--out` path when invoked from a stage's `cmd:` (which is itself currently broken on most Savio nodes — see "Stage layer python3 mismatch" below).

**Stage layer python3 mismatch.** The `dvc.yaml` files for the 7 stage-layer countries (Uganda, Senegal, Malawi, Togo, Kazakhstan, Serbia, GhanaLSS) declare a `cmd:` like `cd _ && LSMS_BUILD_BACKEND=make python3 -m lsms_library.cli materialize ...`. The bare `python3` resolves to system Python, which on Savio is 3.6.8 — too old for `from importlib.metadata import ...` in `lsms_library/__init__.py`. Every `stage.reproduce()` therefore fails with `ModuleNotFoundError`, and `load_dataframe_with_dvc` falls into its outer `except (DvcException, FileNotFoundError)` handler. With the v0.7.0 fix, that handler now writes the `load_from_waves` result to the cache, so the failure is benign for warm-cache reads but still costs a wasted subprocess on cold rebuilds. Fix is to substitute `${PYTHON:-python3}` or `sys.executable` in the dvc.yaml templates; deferred because v0.8.0 retires the stage layer entirely.

**Historical note**: a pre-v0.7.0 revision of this file claimed "Caches auto-invalidate on source/config changes (hash-based)" and "Subsequent calls read cache (<1 sec)". Neither was true for the majority of countries as of the v-migration. The claim was based on how the DVC stage layer works for the 7 countries that have it, generalized incorrectly to the whole library. v0.7.0 makes the second claim true for all 40 countries (via the top-of-function read), but the first claim — auto-invalidation — is still false and is deferred to v0.8.0.

**v0.8.0 (planned)** will add content-hash invalidation: a sidecar `.hash` file recording the MD5 of the country's relevant `data_info.yml` files plus any `_/{table}.py` scripts. On mismatch, rebuild and rewrite both parquet and hash. Contributors editing source data will then no longer need `LSMS_NO_CACHE=1` or `cache clear`. Once that lands, the `dvc.yaml` stage layer becomes redundant and can be retired. See `SkunkWorks/dvc_object_management.org` for the full plan.

**Practical rule for contributors**: when editing a wave's `data_info.yml` or a `_/{table}.py` script, set `LSMS_NO_CACHE=1` for the session OR run `lsms-library cache clear --country {Country}` once before rebuilding. The parquet at `~/.local/share/lsms_library/{Country}/var/{table}.parquet` is the single source of truth for the v0.7.0 top read; deleting it forces a rebuild on the next call.

## Roster-Derived Tables
`household_characteristics` is **auto-derived from `household_roster`** via `roster_to_characteristics()` in `transformations.py`. It should NOT be registered in `data_scheme.yml` --- the `Country` class detects it via `_ROSTER_DERIVED` and applies the transformation automatically when `household_roster` exists. Adding `household_characteristics: !make` to a data_scheme bypasses this and forces legacy scripts to run instead.

## Adding New Surveys
New surveys are added via YAML config files under `lsms_library/countries/`, not Python code.

## Two Build Paths: YAML vs Makefile/Script

There are two ways a table gets built at the wave level:

### YAML path (`data_info.yml`)
The preferred path for simple cases. `Wave.grab_data()` reads `idxvars`/`myvars` from the wave's `data_info.yml`, calls `df_data_grabber()` to extract columns from the source `.dta` file, applies formatting functions, and returns a DataFrame. The Country class aggregates waves and caches to `data_root()`. **No parquet is written at the wave level.**

### Makefile/script path (legacy Python scripts)
Required for cases the YAML structure cannot express. Indicated by `materialize: make` or `!make` in `data_scheme.yml`. `Wave.grab_data()` falls back to running `make` in the wave's `_/` directory, which executes a standalone Python script (e.g., `household_roster.py`) that calls `to_parquet()` to write a `.parquet` file.

**When the script path is necessary** (do not try to replace these with YAML):

- **Multiple observations per wave**: Nigeria has post-planting and post-harvest rounds (`2018Q3`, `2019Q1`) in a single directory (`2018-19/`). The script loads two different source files, assigns different `t` values to each, concatenates, and deduplicates people who leave between rounds. YAML assumes one directory = one `t`.

- **Multi-wave source files**: Tanzania `2008-15/` has a single `.dta` file covering rounds 1--4. The script reads a `round` column and maps it to wave labels. YAML assumes one file = one wave.

- **Complex transformations**: Some tables (Nigeria `food_acquired`, GhanaLSS `food_acquired`) need elaborate unit conversions, label extraction, or cross-file joins that exceed what `df_data_grabber` supports.

**Data separation**: All parquet writes go to `data_root()`, never the repo tree. `to_parquet()` calls `_resolve_data_path()` (in `local_tools.py`), which inspects the call stack to infer country/wave and rewrites relative paths under `data_root()`. This handles three patterns:
- Bare `foo.parquet` from wave-level scripts → `data_root(Country)/wave/_/foo.parquet`
- `../var/foo.parquet` from country-level scripts → `data_root(Country)/var/foo.parquet`
- `../wave/_/foo.parquet` cross-wave refs → `data_root(Country)/wave/_/foo.parquet`

Users can override the location via the `LSMS_DATA_DIR` env var, which `data_root()` reads. Some stale parquets from before this migration still exist in-tree under `_/` directories — they are harmless artifacts.

**Rule of thumb**: If a table can be expressed as column mappings from a single source file per wave, use YAML. If it needs cross-file concatenation, per-row `t` assignment, or multi-wave source files, use a script with `materialize: make`.

## Post-Planting / Post-Harvest (PP/PH) Countries

Several LSMS-ISA countries collect data in two rounds per wave: **post-planting (pp)** and **post-harvest (ph)**. The same household is visited twice --- once after the planting season and once after harvest. This is a cross-cutting structural concern that affects every feature in those countries and is the single most common source of duplicate-index bugs.

### Which countries have dual-round structure

| Country | Program | Waves affected | File naming pattern | Notes |
|---------|---------|----------------|---------------------|-------|
| **Nigeria** | GHS/LSMS-ISA | All waves | `sect*_plantingw*.dta` / `sect*_harvestw*.dta` | Wave directories like `2018-19/` contain both pp and ph data |
| **Ethiopia** | ESS | All 5 waves | `sect*_pp_w*.dta` / `sect*_ph_w*.dta` | Heavy `!make` usage --- most features need scripts |
| **Tanzania** | NPS | `2008-15/` only | Single file with `round` column covering rounds 1--4 | Later waves (`2019-20`, `2020-21`) are single-round |
| **GhanaSPS** | SPS | Some waves | Planting/harvest questionnaires | Less structured than Nigeria/Ethiopia |

### Why YAML cannot express this

The YAML path (`data_info.yml`) assumes **one directory = one `t` value**. In pp/ph countries, a single wave directory (e.g., `Nigeria/2018-19/`) contains two source files that need **different `t` values** (e.g., `2018Q3` for post-planting, `2019Q1` for post-harvest). The YAML path has no mechanism to:

1. Load two different source files from the same directory
2. Assign a different `t` value to each
3. Concatenate the results and deduplicate

This is why pp/ph features **must use `materialize: make`** (or `!make`) with a Python script.

### How pp/ph affects index construction

Each round must receive a **distinct `t` value** so that the same household appearing in both rounds does not create duplicate index entries. The standard patterns are:

- **Nigeria**: Quarter-based `t` values --- `2018Q3` (post-planting) and `2019Q1` (post-harvest)
- **Ethiopia**: Wave label reuse (e.g., `2018-19`) since most features only use one round's data
- **Tanzania `2008-15/`**: The script reads a `round` column and maps values to wave labels (`2008-09`, `2010-11`, `2012-13`, `2014-15`)

### The duplicate-index bug

This is the most common bug in pp/ph countries. It occurs when both pp and ph data are loaded but assigned the **same `t` value**:

```
# BUG: Both rounds get t='2018-19' → household appears twice
pp['t'] = '2018-19'
ph['t'] = '2018-19'
df = pd.concat([pp, ph])   # → 50-87% duplicate indices
```

Symptoms:
- `df.index.duplicated().mean()` returns 0.50--0.87
- `is_this_feature_sane()` reports massive duplicate rates
- Household counts are roughly double the expected number

### How to fix: the script pattern

The correct pattern assigns distinct `t` values to each round, concatenates, and deduplicates:

```python
from lsms_library.local_tools import df_data_grabber, to_parquet

# Post-planting: assign t='2018Q3'
idxvars_pp = dict(i='hhid', t=('hhid', lambda x: '2018Q3'), v='ea', pid='indiv')
myvars_pp = dict(Sex=('s1q2', extract_string), Age='s1q6', Relationship=('s1q3', extract_string))
pp = df_data_grabber('../Data/sect1_plantingw4.dta', idxvars_pp, **myvars_pp)

# Post-harvest: assign t='2019Q1'
idxvars_ph = dict(i='hhid', t=('hhid', lambda x: '2019Q1'), v='ea', pid='indiv')
myvars_ph = dict(Sex=('s1q2', extract_string), Age='s1q4', Relationship=('s1q3', extract_string))
ph = df_data_grabber('../Data/sect1_harvestw4.dta', idxvars_ph, **myvars_ph)

# Concatenate and drop people who left between rounds
df = pd.concat([pp, ph])
df = df.replace('', pd.NA).sort_index().dropna(how='all')

to_parquet(df, 'household_roster.parquet')
```

Key details:
- The `t` lambda assigns a constant string to every row from that file
- Variable names may differ between pp and ph files (e.g., Nigeria's `s1q6` vs `s1q4` for Age)
- `dropna(how='all')` removes individuals who appeared in one round but have no data in the other (attrition between rounds)
- The `data_scheme.yml` must use `materialize: make` for these features

### Reference implementations

- **Nigeria `household_roster`**: `Nigeria/2018-19/_/household_roster.py` --- canonical pp/ph pattern with distinct `t` values and attrition handling
- **Nigeria `food_acquired`**: `Nigeria/2018-19/_/food_acquired.py` --- pp/ph with food item and unit harmonization across rounds
- **Tanzania `2008-15/`**: Multi-round single-file pattern with `round` column mapping

## Joining `v` (cluster) onto tables that lack it

Many roster source files (e.g., Uganda's gsec2) don't carry a cluster column. Join `v` from the survey cover page (e.g., gsec1) using the `dfs:` merge in `data_info.yml`:

```yaml
household_roster:
    dfs:
        - df_roster
        - df_cluster
    df_roster:
        file: ../Data/HH/gsec2.dta
        idxvars:
            i: hhid
            pid: pid
        myvars:
            Sex: h2q3
            Relationship: h2q4
            Age: h2q8
    df_cluster:
        file: ../Data/HH/gsec1.dta
        idxvars:
            i: hhid
        myvars:
            v: s1aq04a        # <-- v as myvar, NOT idxvar
    merge_on:
        - i
    final_index:
        - t
        - v
        - i
        - pid
```

Key details:
- Put `v` in `myvars` of the sub-df, not `idxvars`. An `idxvars`-only sub-df with empty `myvars` fails in `df_data_grabber`.
- The cluster column name changes across waves within a country (e.g., Uganda uses `comm`, `h1aq4a`, `parish_code`, `parish_name`, `s1aq04a` across its 8 waves) because sampling schemes evolve.
- **The `data_scheme.yml` must include `v` in the index** (e.g., `index: (t, v, i, pid)`). If it doesn't, `_normalize_dataframe_index` silently drops `v` from the result.

## Two Makefiles
- Top-level `Makefile`: Poetry setup, pytest, build.
- `lsms_library/Makefile`: Country-specific operations (test, build, materialize, demands).
  Use `make -C lsms_library help` for details.

## Data Access
Underlying microdata must be obtained from the [World Bank Microdata Library](https://microdata.worldbank.org/) under their terms of use. Contributors need GPG/PGP keys for repository write access.

### User configuration (`~/.config/lsms_library/config.yml`)

Library settings live in a YAML file at the platform-appropriate user config directory (on Linux: `~/.config/lsms_library/config.yml`):

```yaml
microdata_api_key: your_key_here
# data_dir: /path/to/override   # same as LSMS_DATA_DIR env var
```

**Lookup order** for each setting: environment variable → config file → None. For example, `MICRODATA_API_KEY` env var takes precedence over `microdata_api_key` in the config file. The `lsms_library.config` module handles this transparently; see `config.get()` for details.

### Reading data files: `get_dataframe()`

**Always use `get_dataframe()` from `local_tools` to read `.dta`/`.csv`/`.parquet` files.** It is the single entry point for reading data and handles all access modes transparently:

```python
from lsms_library.local_tools import get_dataframe

df = get_dataframe('../Data/sect1_hh_w5.dta')
```

The fallback chain is:
1. **Local file** on disk
2. **DVC filesystem** (`DVCFileSystem`) --- streams from the configured DVC remote
3. **`dvc.api.open()`** --- legacy DVC streaming
4. **`get_data_file()`** from `data_access.py` --- downloads from the World Bank Microdata Library NADA API as a last resort (requires `microdata_api_key` in the config file, or `MICRODATA_API_KEY` env var)

This means a script written with `get_dataframe('../Data/file.dta')` works whether the file is already on disk, cached in DVC, or has never been downloaded at all.

### Anti-patterns (do not use)

| Anti-pattern | Why it's wrong |
|---|---|
| `dvc.api.open(fn, mode='rb')` + `from_dta(f)` | Couples to DVC internals, skips the WB fallback |
| `pd.read_stata('/absolute/path/to/file.dta')` | Breaks on other machines, no DVC/WB fallback |
| `pyreadstat.read_dta(path)` directly | Same --- bypasses all access layers |
| `from_dta('lsms_library/countries/...')` with absolute path | Non-portable; use relative `../Data/` paths |

Scripts in `{Country}/{wave}/_/` run from that directory, so `../Data/file.dta` is the standard relative path convention. `get_dataframe` (via `_resolve_data_path`) knows how to resolve these.

### Writing data files: `to_parquet()`

Use `to_parquet(df, 'feature_name.parquet')` from `local_tools`. It writes to `data_root()` (not the repo tree) via `_resolve_data_path()`, which infers country/wave from the call stack.

### Adding new waves: `data_access` module

The `data_access` module provides functions for discovering and downloading new survey waves:

```python
from lsms_library.data_access import discover_waves, add_wave

discover_waves("Ethiopia")          # What's new on the WB?
add_wave("Ethiopia", "6161")        # Download, dvc add, dvc push
```

`push_to_cache_batch()` handles batched `dvc add` + `dvc push` (dramatically faster than per-file). See CONTRIBUTING.org for the manual workflow.

## Cross-Country Feature Class
The `Feature` class assembles a single harmonized DataFrame across all countries that declare a given table:

```python
import lsms_library as ll

roster = ll.Feature('household_roster')
roster.countries          # ['Ethiopia', 'Mali', 'Niger', 'Uganda', ...]
roster.columns            # required columns from global data_info.yml

df = roster()                       # all countries
df = roster(['Ethiopia', 'Niger'])  # specific countries
```

The returned DataFrame has a `country` index level prepended. `Feature` discovers countries by scanning each `data_scheme.yml` for the table name, then calls `Country(name).{table}()` for each.

This is the preferred way to do cross-country comparisons, validation, and diagnostics.

### Cross-Country Label Harmonization (design intent, not yet implemented)

Within a country, `categorical_mapping.org` maps wave-specific labels to a "Preferred Label" so that the same concept has a consistent name across survey rounds. The cross-country analogue would map country-specific labels (e.g., French `"Sécheresse"` vs English `"Drought"`) to a global canonical label.

**Design:** Top-level mapping tables in `lsms_library/categorical_mapping/`:
- `shocks.yml` — shock type labels across countries
- `food_items.yml` — food item names across countries/languages
- `units.yml` — measurement unit names across countries/languages

Same org-table format as per-country `categorical_mapping.org`:
```
#+NAME: harmonize_shocks
| Preferred Label | Ethiopia    | Niger (EHCVM) | Mali (EHCVM) | Uganda          |
|-----------------+-------------+---------------+--------------+-----------------|
| Drought         | Drought     | Sécheresse    | Sécheresse   | Drought         |
| Flood           | Flood       | Inondation    | Inondation   | Flood           |
```

**API:** The `harmonize` parameter selects which column from the mapping table to use:

```python
import lsms_library as ll

# Raw labels (default) — each country's own labels preserved
shocks = ll.Feature('shocks')()

# Harmonize to canonical English
shocks = ll.Feature('shocks')(harmonize='Preferred')

# Harmonize to French
shocks = ll.Feature('shocks')(harmonize='French')

# Harmonize to aggregate categories (coarser grouping)
food = ll.Feature('food_acquired')(harmonize='Aggregate')
```

This mirrors the within-country `categorical_mapping.org` pattern: the org table has multiple named columns (``Preferred Label``, ``Aggregate``, ``French``, etc.) and the caller chooses which mapping to apply. Uganda's food item tables already use ``Preferred`` vs ``Aggregate`` columns for different granularities.

The mapping table determines what columns are available. If a table has only ``Preferred Label``, that's the only option. A richer table might offer ``Preferred``, ``Aggregate``, ``French``, ``FAO Code``, etc.

**Principle:** The library preserves what the survey says. Cross-country label harmonization is a form of aggregation — the analyst's decision, not the pipeline's. These mappings are convenience tools, not data transformations.

## Canonical Schema (`data_info.yml`)
`lsms_library/data_info.yml` is the single source of truth for cross-country conventions:
- **Required columns** per table (e.g., `household_roster` requires `Sex`, `Age`, `Generation`, `Distance`, `Affinity`)
- **Accepted values** (e.g., `Sex: [M, F]`, `Affinity: [consanguineal, affinal, step, foster, unrelated, guest, servant]`)
- **Rejected spellings** (e.g., `Relation` → use `Generation, Distance, Affinity`; `Effected` → `Affected`)

Tests in `test_schema_consistency.py` read from this file — never hardcode schema rules in tests.

## Kinship Decomposition
`household_roster` uses a decomposed representation of kinship (Kroeber 1909) instead of a single `Relationship` string. Four columns replace one:

| Column | Type | Description |
|--------|------|-------------|
| `Sex` | str | `M` or `F` |
| `Generation` | int | Vertical distance from head (0=same, +1=parent, -1=child) |
| `Distance` | int | Collateral distance (0=lineal, 1=sibling line, 2=cousin) |
| `Affinity` | str | `consanguineal`, `affinal`, `step`, `foster`, `unrelated`, `guest`, `servant` |

The runtime automatically expands any `Relationship` column into these three via `_expand_kinship()` in `_finalize_result()`, using the dictionary in `lsms_library/categorical_mapping/kinship.yml`. Per-wave `data_info.yml` files continue to produce a `Relationship` string from raw data — the decomposition happens transparently.

**Adding new labels:** If a survey has an unrecognized relationship string, a warning is emitted. Add the label to `lsms_library/categorical_mapping/kinship.yml` with its `[Generation, Distance, Affinity]` tuple.

## Automatic Categorical Mappings
If a column or index name in a returned DataFrame matches a table name in the country's `categorical_mapping.org` (case-insensitive), and that table has a `Preferred Label` column, the mapping is applied automatically. No YAML `mappings:` declaration needed.

For tables whose names don't match column names (e.g., `harmonize_food` for index `j`), use the explicit `mappings:` syntax in `data_info.yml`.

## Cross-Country Value Normalisation (`data_info.yml` spellings)
Columns in `data_info.yml` can declare a `spellings` inverse dictionary that the runtime enforces automatically. Each key is the canonical value; its list contains accepted variant spellings:
```yaml
Sex:
  type: str
  required: true
  spellings:
    M: [Male, male, Masculin, masculin, Homme, homme, m]
    F: [Female, female, Féminin, feminin, Femme, femme, f]
```
The canonical values are simply `spellings.keys()`. The runtime replaces variants with canonical forms in `_finalize_result()` via `_enforce_canonical_spellings()`. This applies to both column values and index levels.

## Pandas Conventions (>=3.0)
This codebase targets pandas 3.0+. Follow these rules in all new and modified code:

- **No `inplace=True`**: Use `df = df.set_index(...)` instead of `df.set_index(..., inplace=True)`. The `inplace` parameter is removed in pandas 3.0.
- **Use `pd.NA`, not `np.nan`, for missing values in string columns**: Pandas 3.0 defaults to `StringDtype` (PyArrow-backed) where `np.nan` is not a valid sentinel. Use `pd.NA` when replacing/filling missing values in string, ID, or categorical columns (e.g., `.replace('', pd.NA)`, `.replace('nan', pd.NA)`). `np.nan` is still fine for numeric (float) columns.
- **Use `pd.isna()` / `pd.notna()`, not `np.isnan()`**: `np.isnan()` raises `TypeError` on `pd.NA`. The pandas functions handle both `np.nan` and `pd.NA`.
- **Use `.bfill()` / `.ffill()`, not `.fillna(method=...)`**: The `method` parameter was removed in pandas 2.0.
- **No chained indexing for writes**: Copy-on-Write (CoW) is default in pandas 3.0. `df[mask]['col'] = val` silently fails. Use `df.loc[mask, 'col'] = val` instead. For reads, prefer `df.loc[mask, 'col'].iloc[0]` over `df.loc[mask, :]['col'][0]`.
- **No mutating views**: Do not modify DataFrames obtained from `_get_numeric_data()`, `select_dtypes()`, or `groupby()` and expect changes to propagate. Work on `df` directly or assign results back explicitly.
- **Use `.iloc[]` for positional Series access**: `value[0]` is deprecated for integer keys on Series. Use `value.iloc[0]`. This affects all formatting functions that receive multi-column composite values (e.g., `i()`, `pid()`, `Age()` in EHCVM countries).

## `other_features` Is Obsolete --- Use `cluster_features`

Legacy scripts used `other_features.parquet` to join market/region (`m`) into food and shocks data. This is **fully replaced** by `cluster_features` + `_add_market_index()` at query time. Do not read `other_features.parquet` in new code. The `m` index should NOT be baked into cached parquets --- it is added on demand when the user passes `market='Region'` to a table method.

If a wave-level script genuinely needs region for data processing (e.g., Malawi's region-specific unit conversion factors), read from the cover page `.dta` file directly, not from `other_features.parquet`.

## EHCVM Countries: `v` Is Just `grappe`

In EHCVM surveys (Senegal, Mali, Niger, Burkina Faso, Benin, Togo, Guinea-Bissau), each `grappe` (cluster) is visited in exactly one passage (`vague`). The split-sample design means `vague` is redundant for identifying clusters. Use `v: grappe` (not `v: [vague, grappe]`). Similarly, household IDs are `i: [grappe, menage]` (not `[vague, grappe, menage]`).

## `format_id` and Numeric myvars

`format_id` is auto-applied to `idxvars` but **NOT to `myvars`**. If a numeric column (like a cluster ID) comes through as a myvar, it will retain float type and get `.0` suffixes when stringified. Fix by adding a formatting function:

```python
# In {wave}.py
from lsms_library.local_tools import format_id
def v(value):
    return format_id(value)
```

The `is_this_feature_sane()` diagnostic now checks for this via `_check_float_stringified_index`.

## Dispatching Subagents

When using the Agent tool to dispatch work to subagents (especially with `isolation: "worktree"`):

- **Run tests first.** Before dispatching any agents, run `pytest tests/` to establish a baseline. Know what's passing.
- **Worktree agents run stale code.** Worktrees snapshot the branch at creation time. If you commit a fix and then dispatch a worktree agent, it won't have the fix. Either: (a) commit all fixes before dispatching, or (b) don't use worktrees for build-only tasks --- use `LSMS_BUILD_BACKEND=make` to avoid DVC lock contention instead.
- **Non-worktree agents can overwrite committed changes.** After agent work completes, always verify with `git diff HEAD` that the working tree matches what's committed. Restore with `git checkout HEAD -- path/` if needed.
- **S3 credentials don't propagate to worktrees.** The decrypted `s3_creds` file is `.gitignore`d. Agents that need DVC data access must copy it: `cp /main/repo/lsms_library/countries/.dvc/s3_creds $WORKTREE/lsms_library/countries/.dvc/s3_creds`
- **Clean up worktrees promptly.** Remove worktrees and their branches as soon as the agent's work is merged. Stale worktrees accumulate and confuse git operations.
- **Subagents do NOT inherit `.claude/skills/`**. They only see what's in their prompt. Tell agents to read the relevant skill files as their first step: e.g., "Read `.claude/skills/add-feature/SKILL.md` before starting."
- **Subagents share the parquet cache** at `~/.local/share/lsms_library/` --- each country writes to its own directory, so concurrent agents building different countries won't conflict.
- **Subagents should stay in their worktree**. Do not modify the main checkout. If a cross-cutting change is needed, document it and let the manager merge.
- **Prefix heavy Python with `nice -n 10`** to keep the node responsive for interactive work.
- **The Python venv is at the repo root** (`/path/to/LSMS_Library/.venv/bin/python`), not in the worktree. Set `PYTHONPATH` to the worktree so development-branch code is picked up.
- **Use the message channel** (`slurm_logs/build_{date}/MESSAGES.txt`) for steering instructions. Agents should check it periodically.
- **DVC lock contention**: Parallel agents sharing the DVC root (`lsms_library/countries/.dvc/`) can leave stale locks. If you see "Unable to acquire lock", check `lsms_library/countries/.dvc/tmp/lock` — if no `dvc` process is running (`ps aux | grep dvc`), delete the lock file. The library gracefully falls back to manual aggregation but is slower.
- **Scatter-gather for multi-country work**: When applying the same feature to many countries, dispatch **one agent per country** in a single message (all launch in parallel). Never batch multiple countries into one agent — that forces sequential processing on one core while others sit idle. The coordinator commits results as notifications arrive and re-dispatches failures with more context.

## `sample()` and Cluster Identity

The `sample` table (`index: (i, t)`, columns: `v`, `weight`, `panel_weight`, `strata`, `Rural`) is the single source of truth for mapping households to their sampling cluster. It encodes the survey's sampling design: which PSU each household was drawn from, the household's sampling weight (cross-sectional and panel), stratification domain, and urban/rural classification.

**As of 2026-04-10, `v` is joined from `sample()` at API time, not baked into feature parquets.** The framework hook is `_join_v_from_sample()` in `country.py`, called from `_finalize_result()` for any household-level table (features indexed by `(i, t, ...)` except `sample`, `cluster_features`, `panel_ids`, `updated_ids`) when the country has `sample` in its `data_scheme` and `v` is not already present. The sample lookup is cached per Country instance to amortize across feature calls.

- **Do NOT put `v` in feature `data_scheme.yml` indexes** other than `cluster_features` (which legitimately owns `v`). The framework adds it at API time.
- **Do NOT bake `v` into feature parquets**. Wave-level scripts that previously wrote `(t, v, i, ...)` indexes should write `(t, i, ...)` and let the framework join `v`.
- **Do NOT use `dfs:` merge blocks just to join `v` from a cover page** in `data_info.yml`. Collapse the table to a simple single-file extraction.
- **`_add_market_index(market='Region')`** joins `(t, v) → Region` from `cluster_features`. With Phase 2 in place, `v` is guaranteed present; the method joins directly on `(t, v)`. If a caller hits `_add_market_index` with a DataFrame lacking `v` (e.g., a derived table that bypassed `_finalize_result`), it calls `_join_v_from_sample()` as a backup.
- **Two weight types**: `weight` (cross-sectional, positive for all interviewed HH including refreshment) and `panel_weight` (longitudinal, NaN/zero for refreshment sample). Pre-refreshment waves have the same value in both columns.
- **Skill**: `.claude/skills/add-feature/sample/SKILL.md` documents the full process for adding `sample` to a new country.
- **Migration history**: See `slurm_logs/PLAN_sample_v_migration.org` for the Phase 2/3/4 migration plan and `slurm_logs/DESIGN_sample_as_v_source.org` for the original design intent.
- **Country caveat**: `Country(name).household_roster()` requires that the country has `sample` in its `data_scheme.yml` to get `v` in the index. Countries without `sample` (e.g., GhanaSPS at time of writing) return `(i, t, pid)` without `v`.
- **Legacy scripts with `v` as a non-index column**: `_join_v_from_sample()` skips when `v` is already in `df.columns`, not just when it's in `df.index.names`. This handles legacy features like `locality` that write `v` alongside other columns. The check was added after a use_parquet regression comparison surfaced a name-collision crash. If you write a new legacy-style script, prefer putting `v` in the index (or nowhere) rather than as a non-index column, so the framework can do its job cleanly.

## Panel ID Transitive Chains and the `attrs` Flag

`_finalize_result()` runs `id_walk()` to apply the country's
`updated_ids` mapping, and sets `df.attrs['id_converted'] = True`
to mark the DataFrame as already-converted. Any step between
`id_walk` and the downstream consumers that touches the DataFrame
must **preserve `attrs`** or the flag is lost. Two operations that
drop `attrs` in pandas 2.x are `merge()` and `set_index()` — both
of which appear in `_join_v_from_sample()`.

When `attrs` is dropped, `_finalize_result` sees no flag and runs
`id_walk` a second time on already-converted data. For countries
whose `updated_ids` contain transitive chains (A → B → C, where
intermediate IDs are themselves keys in the mapping), the second
pass produces household-level ID collisions and duplicate index
entries. The most visible case is Burkina Faso 2021-22, whose 95
two-step chains produced 392 duplicate `(i, t, v, pid)` tuples
(~0.5% of rows) before the fix landed in commit `4db41a27`.

**Rule**: any framework method that touches a DataFrame in
`_finalize_result()` downstream of `id_walk()` must explicitly
copy `attrs`:

```python
result = flat.set_index(new_idx)
result.attrs = dict(df.attrs)  # preserve id_converted flag
return result
```

Do not rely on `merge()` or `set_index()` to preserve metadata.
They don't.

## Housing Schema: Categorical, Not Binary

Historically, Uganda (and Malawi) `housing` tables carried binary
indicator columns like `Thatched roof` and `Earthen floor`. As of
the 2026-04-10 overhaul, housing is now categorical: the columns
are `Roof` and `Floor`, and the values are the material names as
reported in the survey (`Thatch`, `Iron Sheets`, `Cement`,
`Concrete`, `Tiles`, `Earth`, etc.), mapped through
`categorical_mapping.org` for cross-wave consistency.

This follows the "preserve the detail" principle. Consumers who
want binary indicators can derive them trivially
(`df['Roof'] == 'Thatch'`) but the reverse is not possible. If
your code relies on the old binary columns, update it to use the
categorical strings.

## Cache vs API Transformations

The cached parquets under `data_root()` store **pre-transformation** data. Kinship expansion (`_expand_kinship`), canonical spelling enforcement (`_enforce_canonical_spellings`), and dtype coercion (`_enforce_declared_dtypes`) all happen in `_finalize_result()` at API read time, not at cache write time. This means:

- `Country('X').household_roster()` returns clean, decomposed data (Sex as M/F, kinship as Generation/Distance/Affinity)
- Reading `~/.local/share/lsms_library/X/var/household_roster.parquet` directly with `pd.read_parquet` gives raw data with `Relationship` strings and unnormalized values

The cache is closer to the source data; the API applies the harmonization layer on every read.

**`trust_cache=True` does NOT skip `_finalize_result`.** A pre-v0.7.0 revision of this section claimed it did. The actual code at `country.py:1293-1299` reads the cache via `get_dataframe`, runs `map_index`, then ends with `return self._finalize_result(df_cached, scheme_entry, method_name)` — so kinship expansion, spelling normalization, dtype coercion, and the `_join_v_from_sample` augmentation all still apply. The difference between `trust_cache=True` and the v0.7.0 default top read is narrower than the old wording suggested:

| | `trust_cache=False` (default) | `trust_cache=True` |
|---|---|---|
| Where the read happens | `load_dataframe_with_dvc` top-of-function | `_aggregate_wave_data` short-circuit before any DVC code |
| Skips DVC stage layer | yes | yes |
| Skips ancillary caches setup | no | yes (constructor doesn't preload) |
| Honors `LSMS_NO_CACHE=1` | yes | no |
| Calls `_finalize_result` | yes (via `_aggregate_wave_data`) | yes (explicit `return self._finalize_result(...)` inside the short-circuit) |

In short: `trust_cache=True` is a more aggressive bypass for clusters where the caches are known fresh and the user doesn't even want the existence check of `LSMS_NO_CACHE` to fire. For diagnosing data quality, use `trust_cache=False` (the default) so any debug instrumentation in the stage layer or finalize chain actually runs.

## Derived Tables

`household_characteristics` and the food-derived tables (`food_expenditures`, `food_prices`, `food_quantities`) are **not registered in `data_scheme.yml`**. They are auto-derived at runtime from `household_roster` and `food_acquired` respectively, via `_ROSTER_DERIVED` and `_FOOD_DERIVED` in `country.py`. The `__getattr__` dispatch handles them alongside `data_scheme` entries.

Do NOT add these to `data_scheme.yml` — doing so would bypass the derivation logic and try to load them as standalone features (which will fail unless legacy `!make` scripts exist).

## `panel_ids` Is a Property, Not a Method

`panel_ids` and `updated_ids` are `@property` attributes on `Country`, not dynamic methods. They return dicts, not DataFrames. Code that iterates over `data_scheme` entries and calls `getattr(c, name)()` must special-case these. Use `diagnostics.load_feature(c, name)` which handles both.

## Categorical Columns from `.dta` and `.sav` Files

`get_dataframe()` returns categorical columns from Stata/SPSS files. These can cause issues:

- **`select_dtypes(exclude=['object']).max()`** crashes on unordered categoricals. Exclude `'category'` dtype too.
- **`groupby().first()`** crashes on unordered categoricals. Convert to string with `.astype(str).replace('nan', pd.NA)` first.
- **YAML mapping keys**: when `get_dataframe()` returns categorical labels as strings, mapping dicts must use string keys (e.g., `'urbana': Urban`) not numeric keys (e.g., `1: Urban`).

## Countries Without Microdata

Some countries have configs but no source `.dta` files in the repository:

| Country | Reason | Data source |
|---------|--------|-------------|
| Nepal | NSO hosts data, not WB | https://microdata.nsonepal.gov.np/ (free registration) |
| Armenia | No data files downloaded | WB catalog, external hosting |
| Timor-Leste 2001 | No `_/` config for this wave | WB catalog |
| Guatemala | No PSU/cluster variable in data | ENCOVI 2000 design |

## Three-Tier Credential Model

The library's data access is gated in three tiers, with **the World Bank Microdata API key as the sole real gate**. The S3 bucket is a *cache* over the authoritative WB NADA downloads, not an independent access layer.

| User has | Gets |
|---|---|
| Nothing | Library warns on import; data-access calls raise `RuntimeError` |
| Valid WB Microdata API key (via `MICRODATA_API_KEY` env var or `microdata_api_key` in `~/.config/lsms_library/config.yml`) | (a) Direct WB NADA downloads; (b) S3 read cache (auto-unlocked on import because having the WB key = having accepted the ToU) |
| WB API key + S3 write creds (`LSMS_S3_WRITE_KEY` + `LSMS_S3_WRITE_KEY_ID`, or `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, or a `s3_write_creds` file) | (a) + (b) + push access to the S3 cache, for RAs materializing new waves |

The obfuscated passphrase at `data_access.py` (`_S3_UNLOCK_PASSPHRASE = "QnVubnkgbXVmZmlu"` = base64 for `"Bunny muffin"`) is **cosmetic anti-grep**, not a real gate. The source comment is honest about this: *"The real access gate is WB API key validation; this just keeps the passphrase from being trivially discoverable via grep."* Future maintainers should not try to "fix" the obfuscation by strengthening it — it is intentionally weak because the WB API key check in `_validate_wb_api_key` is the authoritative policy gate.

When `MICRODATA_API_KEY` is present and valid, `_auto_unlock_s3` decrypts `s3_reader_creds.gpg` with the obfuscated passphrase and writes the plaintext creds to the configured location (historically `countries/.dvc/s3_creds`, moving to `~/.config/lsms_library/s3_creds` in v0.7.0). This happens at `import lsms_library` time via `__init__.py`. The interactive `ll.authenticate()` path is a rarely-needed fallback for users without a WB account.

## DVCFileSystem Runtime Config Override

`DVCFileSystem` accepts `config` and `remote_config` kwargs at construction time that override the on-disk `.dvc/config` without modifying any files. This is what makes pip-install scenarios possible: the bundled `.dvc/config` can have placeholder relative paths, and `local_tools.DVCFS` patches them to user-writable absolute paths at import time.

```python
from dvc.api import DVCFileSystem

fs = DVCFileSystem(
    countries_dir,
    config={
        "remote": {"ligonresearch_s3": {"credentialpath": "/home/user/.config/lsms_library/s3_creds"}},
        "cache":  {"dir": "/home/user/.cache/lsms_library/dvc-cache"},
    },
)
```

Two important properties of this override verified empirically (2026-04-10):

1. **DVC is lazy about credential validation.** The credential file does not need to exist at `DVCFileSystem` construction time; DVC only reads it when you actually open a file backed by the remote. This enables the import-time sequencing in `__init__.py`: `DVCFS` can be constructed before `_auto_unlock_s3` has written the plaintext creds, and the subsequent data access still works.

2. **DVC does not require a git ancestor.** A `.dvc/` directory in a `/tmp/...` scratch dir with no `.git/` anywhere above it is a valid DVC repo root. This is what makes the `site-packages/lsms_library/countries/.dvc/` layout work in pip installs.

## `_log_issue` Pollutes the Working Tree

`lsms_library/country.py:168` defines `_log_issue()`, which appends an entry to `lsms_library/ISSUES.md` whenever a cache/materialization build fails. **The file is tracked in git**, has no deduplication, and has no rotation. Any test run or interactive session that encounters an expected data-unavailability condition (e.g., Nepal has no `.dta` files on disk) appends an entry, which makes the working tree dirty as a side effect.

Mitigations until this is actually fixed (tracked in GH #148):

- When committing mid-session, use explicit `git add <specific files>` rather than `git add -A`/`git add .`
- If `ISSUES.md` shows as modified after a test run and the entries are just duplicates of known "no data on disk" conditions (Nepal, Armenia, etc.), revert with `git checkout HEAD -- lsms_library/ISSUES.md`
- The curated entries in the file (with `**Status:** FIXED` markers) are human-written; the auto-appended entries from `_log_issue` are not. Don't conflate them when editing by hand.

## Release Tooling Gotchas

### `poetry-dynamic-versioning` must be installed as a poetry plugin

The `[tool.poetry.requires-plugins]` declaration in `pyproject.toml` is **not enough** in Poetry 2.x. Without the plugin actually installed, `poetry version` reports the static `0.0.0` from `pyproject.toml` and `poetry build` produces a mis-versioned wheel that looks like it worked.

Install once per machine:

```sh
poetry self add "poetry-dynamic-versioning[plugin]>=1.0.0,<2.0.0"
```

If that fails with `Permission denied: 'INSTALLER'` (which happened on the 2026-04-10 session's Savio login node due to a read-only system `pycparser`), fall back to:

```sh
python3 -m pip install --user --ignore-installed "poetry-dynamic-versioning[plugin]>=1.0.0,<2.0.0"
```

Verify with `poetry self show plugins` — the plugin should be listed. Then `poetry version` should report the dynamic version from the latest git tag, not `0.0.0`.

### `poetry build` hangs on Linux keyring without a TTY

Poetry 2.x pulls in `keyring` + `SecretStorage` as transitive dependencies, and on Linux hosts without a graphical session (e.g., Slurm compute nodes, CI containers, headless login sessions), `poetry build` can hang indefinitely on an internal keyring lookup. The subprocess never makes progress and never errors out.

Workaround: set `PYTHON_KEYRING_BACKEND=keyring.backends.null.Keyring` in the environment before calling `poetry build`:

```sh
PYTHON_KEYRING_BACKEND=keyring.backends.null.Keyring poetry build
```

Slurm submission scripts for release builds should export this env var unconditionally.

### Compute nodes have no outbound internet (Savio)

`poetry build` reaches out to `pypi.org` during the build (for build-isolation dependency fetching, even for a simple library). On Savio compute nodes this fails with DNS resolution errors and the build aborts. Release builds should run on the login node or from a system with outbound internet. Regular `pytest` runs do not need internet and can run on compute nodes via Slurm.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ligon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
