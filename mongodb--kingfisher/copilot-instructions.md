## kingfisher

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.

## Project Overview

Kingfisher is an open-source secret scanner and live secret validator written in Rust by MongoDB. It detects, validates, and helps remediate leaked API keys, tokens, and credentials across code repositories, git history, and integrated platforms.

Key capabilities:
- Secret detection with 942 built-in rules (820 standalone detectors + 122 dependent rules; 484 standalone detectors include live validation as of 2026-04-24)
- Live credential validation against provider APIs
- Direct secret revocation from CLI
- Blast radius mapping (AWS, GCP, Azure, GitHub, GitLab, Slack)
- Output formats: TOON, JSON, SARIF, interactive HTML
- Platform integrations: GitHub, GitLab, Azure Repos, Bitbucket, Gitea, Hugging Face, S3, GCS, Docker, Jira, Confluence, Slack, Microsoft Teams, Postman

## Scope
- Applies to the entire repository rooted at this file.
- If a deeper `AGENTS.md` exists in a subdirectory, that file takes precedence for its subtree.

## Repository Structure
- `src/`: main binary source
- `src/cli/commands/`: CLI command implementations
- `src/matcher/`: pattern matching engine
- `src/scanner/`: core scanning logic
- `src/parser/`: language-aware context verification (lightweight lexers, `tl` for HTML, `cssparser` for CSS)
- `src/reporter/`: TOON/JSON/SARIF/HTML report generation
- `src/access_map/`: access mapping analysis
- `crates/kingfisher-core/`: shared types and core logic
- `crates/kingfisher-rules/`: rule loading and rule data
- `crates/kingfisher-rules/data/rules/`: YAML detection rules
- `crates/kingfisher-scanner/`: embeddable high-level scanning API
- `crates/kingfisher-scanner/src/validation/`: shared typed and raw credential validators
- `tests/`: integration/e2e tests
- `testdata/`: test fixtures
- `docs/`: user and developer docs
- `docs/viewer/`: static hosted/local report viewer assets
- `docs-site/`: MkDocs documentation sources, overrides, and generated site output
- `vendor/vectorscan-rs/`: vendored vectorscan bindings

## Toolchain and Environment
- Shell assumptions in build scripts: `bash` with `set -eu -o pipefail`
- Workspace minimum Rust version: `1.94` (`Cargo.toml`)
- `make check-rust` enforces `>= 1.94.1` for build targets
- Windows Makefile targets (`windows-x64`, `windows-arm64`) expect an MSYS2 environment with `pacman` available.
- Prefer `rg` and `rg --files` for fast code/file search.

## Quick Commands
- Development build: `cargo build`
- Release build: `cargo build --release`
- Tests (preferred wrapper): `make tests`
- Tests (direct): `cargo test --workspace --all-targets`
- Nextest (if installed): `cargo nextest run --workspace --all-targets`
- Format: `cargo fmt --all`
- Lint: `cargo clippy --workspace --all-targets -- -D warnings`
- Clean: `make clean`

## Build Targets (Makefile)
- Host convenience:
  - `make linux`
  - `make darwin`
  - `make windows` (Windows host; builds `windows-x64` and `windows-arm64`)
- Explicit platform archives:
  - `make linux-x64`
  - `make linux-arm64`
  - `make darwin-x64`
  - `make darwin-arm64`
  - `make windows-x64` (Windows host only; MSYS2/MinGW flow)
  - `make windows-arm64` (Windows host only; MSYS2 clangarm64 flow)
- Ubuntu bare-metal (Zig/cargo-zigbuild flow):
  - `make ubuntu-x64`
  - `make ubuntu-arm64`

## Code Style
- Rust formatting is defined in `rustfmt.toml` (`max_width = 100`, 4-space indentation, Unix newlines, reordered imports).
- Keep edits minimal and scoped; preserve existing conventions in touched files.
- Prefer clear, maintainable fixes over broad refactors unless requested.

## Architecture Notes
- Rules are YAML-driven and loaded from `crates/kingfisher-rules/data/rules/`.
- Allocator feature flags are in root `Cargo.toml`:
  - `use-mimalloc` (default)
  - `use-jemalloc`
  - `system-alloc`
- Validation modules live in `crates/kingfisher-scanner/src/validation/`; optional validation feature sets are defined in `crates/kingfisher-scanner/Cargo.toml` (e.g., `validation-raw`, `validation-aws`, `validation-gcp`, `validation-database`, `validation-all`).

## Validation and Revocation Policy
- Default rule: define validation logic in rule YAML (`validation:` block), especially `Http` or `Grpc`, not Rust code.
- Typed validators are first-class schema variants (`AWS`, `AzureStorage`, `Coinbase`, `GCP`, `MongoDB`, `MySQL`, `Postgres`, `Jdbc`, `JWT`) for stable, reusable validation families.
- Raw validators use `validation: { type: Raw, content: <name> }` and are the ad-hoc exception path for provider-specific or protocol-specific validation that cannot be expressed reliably in YAML alone. Implement them in `crates/kingfisher-scanner/src/validation/raw.rs`.
- Treat Rust validation additions as rare; prefer extending YAML-based validation first.
- If a Rust exception path is required, prefer adding a raw validator before introducing a new typed validator. Add a new typed validator only when it represents a reusable schema-level validation family.
- Do not convert existing typed validators to `Raw` just for consistency.
- For rules that include validation, add a `revocation:` section whenever the third-party API safely supports revocation.

## Common Development Tasks
- Add a detection rule: follow the workflow below and validate with relevant tests.
- Add a CLI command: implement under `src/cli/commands/` and register in the CLI command wiring.
- Add a validator (rare exception path): implement it in `crates/kingfisher-scanner/src/validation/`, prefer `raw.rs` for one-off provider flows, and wire the narrowest feature/dependencies in `crates/kingfisher-scanner/Cargo.toml` only when YAML validation cannot express the required logic.
- Update docs-site rule counts: use `uv run '/Users/mickg/src/kingfisher/data/default/rule_cleanup/count_rules.py'` and update `docs-site/overrides/` plus `docs-site/mkdocs.yml` to match the reported totals before rebuilding the docs site.

## Rule Authoring Workflow
Use this when creating or updating rules in `crates/kingfisher-rules/data/rules/`.

1. Pick a nearby reference rule file in the same provider family and copy its structure.
2. Define a stable rule id (`id`, prefixed with `kingfisher.`) and detection regex (`pattern`) under `rules:`.
3. Include `examples` that must match. These can be tested with `cargo test check_rules` or `kingfisher rules check --rules-path crates/kingfisher-rules/data/rules/slack.yml --load-builtins=false --no-update-check`
4. Set guardrails:
   - `min_entropy` for high-entropy tokens.
   - `pattern_requirements` (e.g., `min_digits`, `min_uppercase`, `min_lowercase`, `min_special_chars`, `ignore_if_contains`) when format constraints are known.
   - `pattern_requirements.checksum` when provider formats include check digits/signatures.
5. Add `validation` only when a reliable provider/API check exists.
6. Put validation in YAML by default. If YAML cannot express the check, use an existing typed validator or `type: Raw` exception path; add new Rust validator logic only for rare, justified cases.
7. Add `revocation` when the provider API supports safe revocation and the flow is well understood.
8. If a rule needs context from another match (for example ID + secret pair), use `depends_on_rule` and consider `visible: false` on the helper rule.
9. Verify locally:
   - `cargo test -p kingfisher-rules`
   - `cargo test --workspace --all-targets`
   - `kingfisher scan ./testdata --rule <rule-family-or-id> --rule-stats`
   - If validation is implemented: `kingfisher validate --rule <rule-id> <token-or-secret>`
10. Confidence for rules should be set at `confidence: medium`
11. The `pattern` field must contain a valid Hyperscan/Vectorscan regular expression. Lookahead and lookbehind assertions aren’t supported. Because inefficient or overly broad regex can degrade performance, patterns should be as specific as possible and written to minimize false positives.
    1.  **Writing `pattern`**: Start with `(?x)` (free-spacing). Use one unnamed capture `( ... )` around the secret—it becomes `{{ TOKEN }}`. Use `\b` word boundaries and `(?: ... )` for non-capturing structure. For flexible context between keywords and token, use `(?:.|[\n\r]){0,N}?`. Hyperscan doesn't support `(?=...)`; use `pattern_requirements` (e.g. `min_digits`) instead.

## Rule Docs (Read Before Editing)
- `docs/RULES.md`:
  - `Rule Schema` for required/optional fields
  - `Character Requirements` for `pattern_requirements`, `ignore_if_contains`, and `checksum`
  - `Templating with Liquid` for request signing/transforms
  - `Multi-Step Revocation` for complex revoke flows
  - `Writing Custom Rules` and examples for best practices

## Workflow Expectations for Agents
- Do not revert user-authored or unrelated in-progress changes.
- Prefer targeted patches.
- After changes, run the narrowest relevant tests first, then broader checks when practical.
- If validation commands cannot be run, report exactly what was skipped and why.
- Prefer `kingfisher scan --format toon` when invoking Kingfisher from an LLM or agent workflow; keep `pretty` for interactive human CLI use unless the task explicitly calls for a different format.
- After markdown/doc changes, verify local documentation links when practical.
- After `docs-site/` source changes, rebuild with `docs-site/.venv/bin/mkdocs build -f docs-site/mkdocs.yml` when practical so checked-in generated output stays in sync.

## Documentation Pointers
- `docs/USAGE.md`
- `docs/ADVANCED.md`
- `docs/ARCHITECTURE.md`
- `docs/ACCESS_MAP.md`
- `docs/DEPLOYMENT.md`
- `docs/RULES.md`
- `docs/INSTALLATION.md`
- `docs/INTEGRATIONS.md`
- `docs/LIBRARY.md`
- `docs/PYPI.md`

---
> Source: [mongodb/kingfisher](https://github.com/mongodb/kingfisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
