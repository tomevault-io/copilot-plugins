## claude-super-setup

> # Testing Rules


# Testing Rules

- Write tests alongside implementation, not after. Every new function gets a test in the same PR.
- Test BEHAVIOR, not implementation details. If refactoring breaks your test but not the behavior, the test was wrong.
- Structure tests as Arrange-Act-Assert (AAA). One assertion concept per test.
- Name tests: `describe('ModuleName')` > `it('should [expected behavior] when [condition]')`.
- Mock at the boundary: services mock repositories, routes mock services. Never mock the thing you're testing.
- Cover: happy path + at least 2 error paths + 1 edge case per function.
- Never use `test.skip` or `xit` without a TODO comment linking to an issue.
- For async tests, always assert that promises resolve/reject. Don't let unhandled rejections pass silently.
- Target 80% coverage for new code. The CI pipeline enforces this.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Calebmambwe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
