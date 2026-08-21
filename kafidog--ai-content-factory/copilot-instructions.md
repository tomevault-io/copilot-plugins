## ai-content-factory

> - Current status: public OSS release v0.1.0 on GitHub.

# Repository agent instructions

## Product boundary

- Current status: public OSS release v0.1.0 on GitHub.
- Keep the default Python 3.11 runtime standard-library-only and offline.
- Preserve unrelated worktree changes. Never reset, clean, stash, or overwrite
  another contributor's work.
- Do not perform live API calls, remote publishing, browser automation,
  credential setup, payment, deployment, or release publication without a
  separate explicit authorization.
- The v0.1.0 release is already public; future releases and public publication
  actions still require separate explicit authorization.
- Never add real secrets, private paths, private assets, personal data, account
  identifiers, or private brand names to tracked files.

## Architecture

- Core code owns generic contracts, orchestration, integrity, QA, and local
  packages. Vendor behavior belongs behind provider/publisher interfaces.
- The fixture registry is the default. Optional adapters must be explicitly
  selected and may not become hidden fallbacks.
- Product-native tools, model runtimes, and local media tools are optional
  adapters. The base demo cannot depend on them.
- Brand assets and production configuration belong in an external private
  layer and must not be copied into this repository.

## Media boundary

- Call still-image transforms `MOTION_RENDER`; do not claim synthesized subject
  motion.
- Imported media requires explicit provenance and rights status. Unknown rights
  block materialization into a review package.
- Model weights, caches, browser state, and private media never belong in the
  source release candidate.

## Security and provenance

- Keep scanner output redacted. Store private brand denylist entries only as
  SHA-256 fingerprints.
- Record non-trivial source, dependency, and design decisions in
  `PROVENANCE_LEDGER.md`.
- A local scan or fixture test is evidence for that boundary only, not proof of
  live provider, rights, quality, or public-release approval.
- Build public candidates only with `public_release_manifest.json` and
  `scripts/build_release_candidate.py`; never copy the whole worktree.

## Verification

```text
python -B -m unittest discover -s tests -p "test_*.py"
python -B scripts/public_ci.py
python -B scripts/security_scan.py --root . --brand-hash-file scripts/public_brand_hashes.sha256
python -B -m ai_content_factory demo --output output
python -B -m ai_content_factory inspect --output output
python -B -m ai_content_factory validate --output output
```

Apache-2.0 covers project code only. External adapters, tools, models, fonts,
and user assets retain their own terms and must be documented separately.

---
> Source: [kafidog/ai-content-factory](https://github.com/kafidog/ai-content-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
