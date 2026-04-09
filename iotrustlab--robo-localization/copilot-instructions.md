## robo-localization

> - Write unit tests BEFORE implementing functionality

- Write unit tests BEFORE implementing functionality
- Tests must verify mathematical correctness, not curve-fit to specific outputs
- Use pytest fixtures for sensor data generation
- Test edge cases: sensor dropouts, high noise, initialization
- Verify coordinate frame transformations with known values
- Assert filter convergence properties, not exact numerical outputs
- If tests fail, fix the implementation logic, never adjust test expectations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iotrustlab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/iotrustlab)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
