## awesome-localdb

> A curated catalog of ~43 local/embedded databases — databases you can import as a library without a separate server. Each entry has a `metadata.json` descriptor, a `README.md` with code snippets, and self-contained language examples under `databases/<slug>/<lang>/`.

# Awesome Local DB

A curated catalog of ~43 local/embedded databases — databases you can import as a library without a separate server. Each entry has a `metadata.json` descriptor, a `README.md` with code snippets, and self-contained language examples under `databases/<slug>/<lang>/`.

The SPA at `site/` is a client-side filtering UI deployed to GitHub Pages.

## Directory Structure

```
awesome-localdb/
├── databases/           # One subdirectory per database
│   └── <slug>/
│       ├── metadata.json    # Canonical descriptor (follows schemas/metadata.schema.json)
│       ├── README.md        # Human-readable with inline code snippets
│       └── <lang>/          # Language example: python/ rust/ go/ js/
│           ├── pyproject.toml | Cargo.toml | go.mod | package.json
│           ├── src/
│           └── tests/
├── schemas/
│   ├── metadata.schema.json # JSON Schema for databases/*/metadata.json
│   └── index.schema.json    # Schema for site/index.json
├── scripts/
│   ├── build-index.sh       # Aggregate metadata.json → site/index.json + filter-options.json
│   ├── validate-metadata.py # Validate all metadata.json against the schema
│   ├── update-stars.py      # Fetch live github_stars from GitHub API
│   └── check-staleness.py   # Warn on metadata.json stale > 6 months
├── site/                    # Static SPA (client-side search/filter/decision tree)
│   ├── index.html
│   ├── index.json           # Built by build-index.sh — never edit manually
│   ├── filter-options.json  # Built by build-index.sh — never edit manually
│   ├── decision-tree.json   # Interactive wizard steps
│   ├── js/app_new.js        # Current UI code
│   └── css/
├── templates/               # README and recipe templates for contributors
├── .github/workflows/
│   ├── deploy.yml           # GitHub Pages deploy on push to main
│   ├── validate.yml         # Schema validation + index freshness on PR
│   ├── test-examples.yml    # Run recipe tests (Python + Rust)
│   ├── staleness-check.yml  # Monthly warning for stale metadata
│   └── update-stars.yml     # Weekly refresh of github_stars via API
└── .pre-commit-config.yaml
```

## Key Scripts

| Script | Purpose | Run when |
|--------|---------|----------|
| `bash scripts/build-index.sh` | Validates all metadata, rebuilds `site/index.json` and `site/filter-options.json` | After any metadata change |
| `python3 scripts/validate-metadata.py` | Validates `metadata.json` files against the JSON Schema | Before committing metadata changes |
| `python3 scripts/update-stars.py` | Fetches live star counts from GitHub API via `gh api` | To refresh stale `github_stars` |
| `python3 scripts/update-stars.py --dry-run` | Preview what would change without writing | Before applying star updates |
| `python3 scripts/check-staleness.py` | Warns on metadata.json untouched > 6 months | Monthly CI advisory |

To serve the SPA locally: `python3 -m http.server -d site 8080`

## Pre-commit Hooks

Installed via `pre-commit install`. Run on every commit:

- `trailing-whitespace`, `end-of-file-fixer`, `check-yaml`, `check-json` — formatting
- `detect-private-key`, `gitleaks` — security
- `validate-metadata` — runs `validate-metadata.py`
- `build-index` — runs `build-index.sh` and checks for uncommitted changes

## Common Workflows

### Adding a new database
1. Create `databases/<slug>/metadata.json` following `schemas/metadata.schema.json`
2. Create `databases/<slug>/README.md` using `templates/DATABASE_README.md`
3. Add at least one language example in `databases/<slug>/<lang>/` with tests
4. Run `bash scripts/build-index.sh` and commit the updated `site/` files
5. Open a PR — CI will validate metadata, run tests, and check index freshness

### Updating an existing database
1. Edit `databases/<slug>/metadata.json`
2. Run `python3 scripts/validate-metadata.py && bash scripts/build-index.sh`
3. Commit and PR

### Fixing stale stars
Run `python3 scripts/update-stars.py` then `bash scripts/build-index.sh`.

### Modifying the schema
When changing `schemas/metadata.schema.json`:
- Update `scripts/validate-metadata.py` if validation logic changes
- Update `site/decision-tree.json` if tags/categories/languages change
- Run `bash scripts/build-index.sh` and regenerate

### Testing recipe examples
```bash
# Python
cd databases/<slug>/python && pip install -e ".[dev]" && python -m pytest -v

# Rust
cd databases/<slug>/rust && cargo test && cargo fmt --check
```

## Constraints

- `site/index.json` and `site/filter-options.json` are generated — never edit them by hand
- `github_stars` is updated by `update-stars.py` using exact API values
- Database must be embeddable (no mandatory daemon) to qualify for inclusion
- Only `databases/*/metadata.json` is the source of truth; the SPA reads from `site/index.json`
- Pre-commit hooks must pass (they run in CI and block merge)
- The `build-index` pre-commit hook will fail if you modified metadata but didn't rebuild the index

## CI

- **Push to main (site/ or databases/ changes):** deploys to GitHub Pages
- **PR:** validates metadata, rebuilds index, checks for drift, runs recipe tests
- **Weekly (Monday):** `update-stars.yml` fetches live star counts, opens a PR
- **Monthly (1st):** `staleness-check.yml` warns on stale metadata

---
> Source: [sdkks/awesome-localdb](https://github.com/sdkks/awesome-localdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
