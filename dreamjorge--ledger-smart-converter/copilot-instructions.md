## ledger-smart-converter

> **Name**: Ledger Smart Converter

# Gemini CLI Code Context

## Project Identity

**Name**: Ledger Smart Converter
**Type**: Bank statement importer with ML categorization
**Stack**: Python 3.8+, Streamlit, sklearn, pytest
**Domain**: Financial data ETL (PDF/XML/XLSX → Firefly III CSV)

## Quick Reference

**Project Type**: Financial data importer with ML categorization
**Language**: Python 3.8+
**Framework**: Streamlit (web UI)
**Primary Purpose**: Parse bank statements (PDF/XML/XLSX) → Firefly III CSV

## 🤖 Multi-Agent Team Workflow (Gemini + Claude + Codex)

For maximum efficiency and **token balance**, follow the [Multi-Agent Team Workflow Guide](docs/TEAM_WORKFLOW.md).

1.  **Gemini CLI**: **Research & Strategy** (Reading all files/context).
2.  **Claude CLI**: **Execution & Implementation** (High precision coding).
3.  **Codex CLI**: **Validation & Boilerplate** (Shell tasks, data generation).

---

## Architecture is Layered
- **Domain** models enforce validation (`src/domain/`)
- **Services** contain business logic (`src/services/`)
- **UI** is presentation only (`src/ui/pages/`)
- **Importers** are bank-specific parsers (`src/import_*.py`)

### Critical Workflows
1. **Imports**: Input → Parse → Validate → Categorize → Atomic CSV write
2. **Rules**: Stage in `rules.pending.yml` → Conflict check → Backup → Merge → Retrain ML
3. **Analytics**: Load CSVs → Aggregate → Display metrics + drill-down

### Safety First
- **Never** edit `config/rules.yml` directly (use pending workflow)
- **Always** validate domain models (Transaction class)
- **Always** use atomic writes for CSV outputs
- **Check** tests pass before structural changes: `pytest tests/`

### Critical Files

| File | Purpose |
|------|---------|
| `src/generic_importer.py` | CLI entry point, validation flags |
| `src/domain/transaction.py` | Canonical model |
| `src/services/import_service.py` | Import orchestration |
| `src/services/rule_service.py` | Safe rule staging/merge |
| `src/ml_categorizer.py` | sklearn category predictions |
| `src/pdf_utils.py` | PDF text + OCR extraction |
| `src/ui/pages/import_page.py` | Upload interface |
| `src/ui/pages/analytics_page.py` | Dashboard + rule correction |
| `config/rules.yml` | Account config + categorization rules |
| `tests/` | pytest suite (14 tests) |

## Architecture Pattern

**Layer Separation**:
- **Domain** (`src/domain/`): Validated canonical models
- **Services** (`src/services/`): Business logic (import, rules, analytics)
- **UI** (`src/ui/pages/`): Streamlit presentation layer
- **Utilities**: Validation, logging, settings, OCR

**Data Flow**: Input → Parse → Validate → Categorize (Rules + ML) → CSV → Firefly III

## Common Tasks

### Add New Bank Importer
1. Create `src/import_<bank>_firefly.py`
2. Implement parser returning `List[Transaction]`
3. Add bank config to `config/rules.yml`
4. Register in `src/generic_importer.py`
5. Add tests in `tests/test_<bank>.py`

### Modify Transaction Model
1. Edit `src/domain/transaction.py`
2. Update `src/validation.py` validators
3. Update importers to populate new fields
4. Run `pytest tests/test_validation.py`

### Add Categorization Rule
1. **Safe workflow**: Edit `config/rules.pending.yml` (not `rules.yml` directly)
2. Use UI "Apply Pending Rules" or `src/services/rule_service.py`
3. System creates backup in `config/backups/` before merge
4. ML retrains after merge

### Fix OCR Issues
1. Check `src/pdf_utils.py` regex patterns
2. Test with `src/pdf_feedback.py --pdf X.pdf --xml X.xml`
3. Review `data/<bank>/feedback/` outputs
4. Adjust preprocessing/date parsing logic

### "Add support for X bank"
1. Create `src/import_<bank>_firefly.py` parser
2. Add config section in `config/rules.yml`
3. Register in `src/generic_importer.py`
4. Write tests in `tests/test_<bank>.py`

### "Fix OCR parsing"
1. Check `src/pdf_utils.py` regex patterns
2. Run `src/pdf_feedback.py` to compare OCR vs reference data
3. Review feedback files in `data/<bank>/feedback/`
4. Adjust date parsing or preprocessing

### "Modify transaction fields"
1. Update `src/domain/transaction.py` model
2. Update validators in `src/validation.py`
3. Update importers to populate new fields
4. Update analytics queries in `src/services/analytics_service.py`
5. Run `pytest tests/test_validation.py`

### "Add new categorization rule"
- **User-facing**: They use UI rule correction feature
- **Code-level**: Edit `config/rules.pending.yml`, then use rule service to merge

### "Improve ML predictions"
1. Review `src/ml_categorizer.py` model
2. Check training data quality (requires existing categorized transactions)
3. Consider feature engineering (merchant name cleaning, amount ranges)
4. Model retrains automatically when rules are updated

## Testing & CI

**Run Tests**: `python -m pytest -q`
**CI**: `.github/workflows/ci.yml` (compile + pytest on push/PR)
**Coverage**: Import services, validation, settings, healthcheck

## Testing Strategy

**Unit tests**: Domain models, validation logic
**Integration tests**: Import service workflows
**Coverage areas**: Import services, validation, settings, healthcheck
**Run**: `python -m pytest -q` (14 tests currently passing)
**CI**: GitHub Actions on push/PR (`.github/workflows/ci.yml`)

## Code Style

- Type hints for service layer functions
- Docstrings for public functions
- Validation at domain boundary
- Structured logging (`logging_config.py`)
- Error types in `src/errors.py`

## Code Conventions

- **Type hints**: Use for service layer functions
- **Validation**: At domain boundary (Transaction creation)
- **Errors**: Use typed errors from `src/errors.py`
- **Logging**: Use `logging_config.py` structured logger
- **Atomicity**: Write temp file → rename (see utilities)

## Environment

**Config**: `.env` file (see `.env.example`)
**Key Settings**: OCR paths, data directories, test mode flag
**Healthcheck**: `python src/healthcheck.py` validates dependencies

## Configuration Files

**`config/rules.yml`**: Account definitions, closing days, categorization rules
**`config/rules.pending.yml`**: Staged rule changes (safe workflow)
**`.env`**: Environment settings (see `.env.example`)
**`requirements.txt`**: Python dependencies

## Deployment Modes

**CLI**: `python src/generic_importer.py --bank <name> --data <file> --out <csv>`
**Web**: `./scripts/run_web.sh` → `http://localhost:8501`
**Flags**: `--strict` (fail fast), `--dry-run` (no writes), `--log-json` (audit manifest)

## Known Patterns

- **Atomic writes**: Write to temp file, rename on success
- **Safe rules**: Stage in `.pending.yml`, detect conflicts, backup on merge
- **Fuzzy search**: Use `smart_matching.py` for merchant lookup
- **ML predictions**: sklearn model retrains on rule changes
- **Statement periods**: Auto-tag `period:YYYY-MM` based on `closing_day` config

## Known Gotchas

- **OCR dependency**: Tesseract required for scanned PDFs (optional otherwise)
- **Rule conflicts**: System detects before merge and prompts resolution
- **CSV encoding**: Handle UTF-8 with BOM for Excel compatibility
- **Date parsing**: Different banks use different formats, check `pdf_utils.py`
- **Statement periods**: Calculated from `closing_day` config, tagged as `period:YYYY-MM`

## Anti-Patterns to Avoid

- Don't mutate `config/rules.yml` directly (use pending workflow)
- Don't skip validation layer (domain models enforce contracts)
- Don't use `cat`/`sed` for CSV writes (use atomic write utilities)
- Don't hardcode paths (use `settings.py` config)

## Future Direction (Roadmap)

**Next Phase**: SQLite persistence, account unification, hash-based deduplication
**See**: `docs/plan_mejoras.md` for detailed roadmap

## Roadmap Context

**Recently completed**: Validation layer, service architecture, safe rules workflow, CI
**Next up**: SQLite persistence, account unification, hash-based deduplication
**Future**: Firefly API sync, automated monthly reports
**See**: `docs/plan_mejoras.md` for details

## Quick Diagnostics

```bash
# Validate environment
python src/healthcheck.py

# Test imports work
python -m pytest tests/ -v

# Dry run import
python src/generic_importer.py --bank santander_likeu --data test.xlsx --dry-run

# Check rule conflicts
# (Use UI "Apply Pending Rules" button)
```

## When You're Stuck

1. **Run healthcheck**: `python src/healthcheck.py`
2. **Check logs**: Import service creates structured logs
3. **Dry run**: Use `--dry-run` flag to test without writing
4. **Feedback tool**: Use `src/pdf_feedback.py` for OCR debugging
5. **Ask about**: Architecture decisions (check service layer), roadmap priorities (check plan_mejoras.md)

## Token-Saving Tips

- Read `docs/project-index.qmd` for comprehensive overview
- Check `src/domain/transaction.py` for data model
- Review `config/rules.yml` for configuration schema
- Use `src/services/` for business logic reference
- Consult `docs/plan_mejoras.md` for roadmap context

## Token-Efficient References

- **Full overview**: `docs/project-index.qmd`
- **Agent quick ref**: `AGENTS.md`
- **Data model**: `src/domain/transaction.py` (single source of truth)
- **Config schema**: `config/rules.yml` (commented with examples)
- **Roadmap**: `docs/plan_mejoras.md`

## What Users Care About

1. **Accuracy**: Correct categorization (Rules + ML working together)
2. **Safety**: No duplicate imports, no data loss (validation + atomic writes)
3. **Ease**: Simple UI for upload and rule correction
4. **Privacy**: 100% local processing (no cloud calls)
5. **Reliability**: Consistent results, good error messages

## My Goal When Helping

- **Preserve safety**: Don't bypass validation or atomic writes
- **Maintain architecture**: Keep layers separate (domain/service/UI)
- **Test changes**: Run pytest before declaring done
- **Document decisions**: Update comments/docstrings for non-obvious logic
- **Stay focused**: Don't over-engineer, solve the specific problem

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dreamjorge)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dreamjorge)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
