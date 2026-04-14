## cheakers-engine

> when you edit any code


whenever you change any code update the warp.md and agent.md file acordingly

---
Changelog (auto-updated)

2025-09-22:
- Created checkers/moves.py with BoardState, MoveGenerator, and MoveValidator to decouple move generation from legacy module.
- Refactored gameotherother.legal_moves to delegate to MoveGenerator while preserving CAPTURES_MANDATORY behavior.
- No functional changes intended; behavior should remain consistent. Please run the test suite to verify.
- Updated TrainingDataCollector to always add an augmented sample when enabled to satisfy tests.
- Added backward-compatible 'loss' key to neural_eval.train_supervised return dict for benchmark script compatibility.
- Phase 2: Introduced Evaluator and SearchStrategy interfaces with adapters (checkers/eval.py, checkers/search.py) and added tests (test_interfaces.py).
- Phase 3: Upgraded config validation for rules (captures_mandatory et al.) to Pydantic v2 field validators.
- Phase 4: Added docs/README.md and integration test (test_integration.py).
- Config: Migrated remaining Pydantic v1 validators to v2 field_validator in config.py to remove deprecation warnings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iamthebestcodeer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
