## openccu-data

> This document is the entry point for AI assistants working on the

# CLAUDE.md - AI Assistant Guide for openccu-data

This document is the entry point for AI assistants working on the
`openccu-data` codebase. It is intentionally concise.

## Project Overview

`openccu-data` is the **data-extraction pipeline and source-of-truth
distribution** for Homematic CCU configuration metadata. It parses TCL and
JavaScript files from
[OCCU](https://github.com/eq-3/occu) /
[OpenCCU](https://github.com/jens-maus/RaspberryMatic) into compact, typed
JSON artifacts that downstream Python libraries vendor at runtime.

### Key characteristics

- **Language**: Python 3.14+
- **Runtime dependencies**: stdlib only
- **License**: MIT for code; see [`NOTICE.md`](./NOTICE.md) for data licensing
- **Consumers**:
  - [`aiohomematic`](https://github.com/sukramj/aiohomematic) — vendors the
    `easymode_extract`/`translation_extract` archives + `translation_custom/`
  - [`aiohomematic-config`](https://github.com/sukramj/aiohomematic-config) —
    vendors `profiles/*.json` + `_receiver_type_aliases.json`

## Codebase layout

```
openccu_data/
├── const.py                       single source of truth for VERSION
├── data/                          committed extracted artifacts (source of truth)
│   ├── easymode_extract.json.gz
│   ├── translation_extract.json.gz
│   ├── translation_custom/        curated translation overrides (MIT)
│   └── profiles/                  one .json.gz per receiver channel type
│       └── _receiver_type_aliases.json   (uncompressed; tiny, eager-loaded)
├── easymodes/extractor.py         TCL easymode parser → easymode_extract.json.gz
├── translations/extractor.py      JS + stringtable parser → translation_extract.json.gz
└── profiles/extractor.py          TCL profile parser → profiles/*.json

script/                            CLI wrappers (sys.path shim → main())
tests/                             pytest suite
```

The three extractors are independent. They share no helper module; each is
self-contained and only depends on the standard library.

## Development environment

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -e .[test]
```

### Common commands

```bash
pytest tests/                                       # run tests
pytest tests/ -v                                    # verbose
ruff check openccu_data/ tests/ script/             # lint
ruff format openccu_data/ tests/ script/            # format
mypy                                                # type-check
```

### Console scripts

After `pip install -e .` three commands are on PATH:

```
openccu-extract-easymodes
openccu-extract-translations
openccu-extract-profiles
```

All read `OCCU_PATH` (local checkout), `CCU_URL` (running instance), and
optionally `OUTPUT_DIR` from the environment. `.env` at the repo root is
loaded by each `main()` (existing env vars win).

## Architecture notes

- **No runtime third-party deps.** Each extractor inlines a tiny
  `_load_dotenv()` so we don't pull in `python-dotenv`.
- **Output paths** default to `openccu_data/data/` resolved relative to the
  package directory (not the cwd) — so the scripts work regardless of where
  they are invoked from.
- **OCCU_PATH resolution**: relative paths resolve against the repository
  root (`Path(__file__).parent.parent.parent`).
- **Merging**: `easymodes` and `translations` extractors merge results when
  both `OCCU_PATH` and `CCU_URL` are set. `profiles` prefers `CCU_URL` and
  uses `OCCU_PATH` as a fallback for empty results.

## Workflow / changelog

- Format: `YYYY.M.NN`
- Update both `openccu_data/const.py:VERSION` and `changelog.md` together.
- Branches: `main` is the default branch; PRs land here and releases are cut from it.

## When you regenerate an artifact

1. Run the relevant `openccu-extract-*` command.
2. Diff `openccu_data/data/...` (be ready for large diffs in `*.json.gz`).
3. Commit the change in this repo with a description of the OCCU revision /
   CCU firmware version that produced it.
4. **Sync to the vendored copies** in the consumer repos:
   - `aiohomematic/aiohomematic/ccu_data/`
   - `aiohomematic-config/aiohomematic_config/profiles/`

Open companion PRs in the consumer repos and reference the openccu-data
commit.

## Data licensing reminder

The files in `openccu_data/data/` (excluding `translation_custom/`) are
derivative works of OCCU/RaspberryMatic and remain under their EQ-3 license.
The Python _code_ is MIT. See [`NOTICE.md`](./NOTICE.md) before
redistributing.

## Tips for AI assistants

### Do

- ✅ Keep the extractors stdlib-only.
- ✅ Match the existing style (regex constants at top, helpers below, `main()`
  at the bottom, `if __name__ == "__main__":` block).
- ✅ Update tests when changing parsing logic; tests live next to the
  extractor they cover.
- ✅ Update `NOTICE.md` if a new artifact is added to `data/`.
- ✅ Bump `VERSION` and `changelog.md` for any user-visible change.

### Don't

- ❌ Add runtime third-party dependencies — keep it stdlib.
- ❌ Skip the consumer-repo sync after regenerating artifacts.
- ❌ Touch `data/` files by hand — always regenerate via an extractor.
- ❌ Move modules without updating the console-script entry points in
  `pyproject.toml`.

## When in doubt

1. Read the relevant extractor's module docstring.
2. Look at how the consuming project loads the artifact
   (`aiohomematic/ccu_translations.py`, `aiohomematic/easymode_data.py`,
   `aiohomematic_config/profile_store.py`).
3. Run the tests.

---
> Source: [SukramJ/openccu-data](https://github.com/SukramJ/openccu-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
