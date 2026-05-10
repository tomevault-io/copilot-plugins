## defensive-coding

> when to do defensive and when not to do defensive coding


Follow these instructions when writing any python code

1. Keep code minimal by focusing only on the logic needed for the task.
2. Trust internal invariants in self-contained projects—don't check for impossible conditions.
3. Avoid unnecessary type checks or error handling when inputs are fully controlled.
4. Use assertions during development to catch unexpected states, but remove or reduce them in production if they're redundant.
5. Apply defensive coding only when dealing with:
    a. External inputs (user data, files, APIs)
    b. Unreliable dependencies
    c. Long-lived or reused code (e.g. libraries, shared modules)
6. Prioritize readability and maintainability over protecting against unrealistic edge cases.
7. Do not use try catch paradigm as much as possible. It should be used only when it is actually required.

---
> Source: [skylight-org/sparse-attention-hub](https://github.com/skylight-org/sparse-attention-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
