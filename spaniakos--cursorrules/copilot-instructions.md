## cursorrules

> Testing rules


# Testing Rules

All generated or modified code should be accompanied by appropriate tests whenever practical:
- Unit tests.
- Integration / functional tests.
- Smoke tests.
- Playwright or similar E2E tests for UI flows when applicable.

Rules:
- All tests must pass.
- Do not mock away core business logic in a way that hides real behavior.
- Include edge-case and negative tests, not only happy paths.
- Ask which testing framework to use if unclear (PHPUnit, Pest, PyTest, JUnit, xUnit, etc.).
- Propose additional tests if coverage appears low for critical components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spaniakos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
