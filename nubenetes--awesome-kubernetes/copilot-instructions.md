## awesome-kubernetes

> This repository uses **Gitflow**. All agents MUST follow this branching model:

# CLAUDE.md — Nubenetes Project Instructions for Claude Code

## Git Workflow: Gitflow

This repository uses **Gitflow**. All agents MUST follow this branching model:

### Branches
- **`master`** — Production. Only receives merges from release branches. Every merge to master gets a **tag** and a **GitHub Release**.
- **`develop`** — Integration branch. All feature branches merge here first.
- **`feat/*`** — Feature branches. Created from `develop`, merged back to `develop` via PR.
- **`release/vX.Y.Z`** — Release branches. Created from `develop` when ready to release, merged to both `master` AND back to `develop`.
- **`gh-pages`** — Deployment. Never touch directly.

### Release Process (mandatory for all releases)
1. Create feature branch from `develop`: `git checkout -b feat/description develop`
2. Implement changes, commit, push feature branch
3. Create PR: `gh pr create --base develop --head feat/description`
4. Merge PR to develop: `gh pr merge N --merge`
5. Create release branch: `git checkout -b release/vX.Y.Z develop`
6. Merge release to master: `git checkout master && git merge release/vX.Y.Z --no-ff`
7. Tag: `git tag -a vX.Y.Z -m "description"`
8. Push master + tag: `git push origin master && git push origin vX.Y.Z`
9. Back-merge master to develop: `git checkout develop && git merge master --no-ff && git push origin develop`
10. Create GitHub Release: `gh release create vX.Y.Z --title "..." --notes "..."`

### Versioning
- Current: `v2.9.2`
- Format: `v{major}.{minor}.{patch}`
- Major: breaking changes or architectural shifts
- Minor: new features (like the digest engine, new modules)
- Patch: bug fixes, config tweaks

### Protected Branches
`master`, `develop`, and `gh-pages` are NEVER deleted. Branch cleanup runs bi-monthly for merged feature branches.

## Repository Structure

### Key Directories
- `docs/` — V1 source (exhaustive archive, source of truth)
- `v2-docs/` — V2 source (AI-curated elite portal, derived from V1)
- `src/` — Python pipeline source code
- `data/` — Inventory (YAML + SQL), config files, digest JSON
- `scripts/` — Utility scripts (backfill, etc.)
- `.github/workflows/` — CI/CD (15 workflows)

### Key Config Files
- `v2-mkdocs.yml` — V2 MkDocs Material configuration
- `mkdocs.yml` — V1 MkDocs configuration
- `data/inventory.yaml` / `data/inventory.sql` — Unified inventory (18K+ entries)
- `data/curation_sources.yaml` — RSS feeds and X/Twitter accounts
- `data/link_rules.yaml` — Curation policies
- `GEMINI.md` — AI mandates and learning roadmap (read by Gemini agents)

### Pipeline Modules
- `src/v2_optimizer.py` — Main rendering engine (V2VisionEngine class)
- `src/news_digest.py` — 26-category temporal digest with Gemini ranking
- `src/enrichment.py` — CNCF Landscape + GitHub activity + license detection
- `src/dedup.py` — URL/hash/title deduplication engine
- `src/agentic_curator.py` — Ingestion pipeline with AI evaluation
- `src/autonomous_discovery.py` — GitHub trending discovery (14 queries)
- `src/gemini_utils.py` — Gemini API wrapper with key rotation
- `src/inventory_manager.py` — Dual YAML+SQL inventory management

## Build and Run

### Local testing (no API keys needed)
```bash
pip install -r requirements.txt
mkdocs serve -f v2-mkdocs.yml          # Preview V2 portal
mkdocs serve -f mkdocs.yml             # Preview V1 portal
```

### Pipeline commands
```bash
python3 -m scripts.backfill_discovered_at   # Backfill discovered_at field
python3 -m src.news_digest                   # Generate digest (needs Gemini API key)
python3 -m src.enrichment                    # CNCF + GitHub enrichment (needs GH_TOKEN)
python3 -m src.dedup                         # Dedup scan (dry-run by default)
python3 -m src.v2_optimizer --render-only    # Render V2 portal (no AI calls)
```

## Coding Conventions
- All Python exceptions must use `except Exception as e: log_event(f"[WARN] context: {str(e)[:100]}")` — never bare `except: pass`
- Use `from src.logger import log_event` for logging
- Use `from src.config import MADRID_TZ` for timezone-aware timestamps
- Inventory fields: `discovered_at` (ISO), `last_ai_eval` (ISO), `company`, `geo_region` must be preserved during merges
- The `update_inventory_entry()` function preserves `discovered_at` — never overwrite it with new data

## URL Policy: Clean URLs, No .html Suffix
Both V1 and V2 MUST use `use_directory_urls: true` in their mkdocs.yml. This produces clean URLs like `/kubernetes/` instead of `/kubernetes.html`. **NEVER** enable the `offline` plugin — it forces `.html` suffixes on all URLs, breaking SEO and existing deep-links. This is a hard rule.

## RSS/Twitter Sources
RSS feeds are limited to those that actually work (many block bots). Don't add new RSS feeds without testing. Current working feeds are defined in `data/curation_sources.yaml`.

---
> Source: [nubenetes/awesome-kubernetes](https://github.com/nubenetes/awesome-kubernetes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
