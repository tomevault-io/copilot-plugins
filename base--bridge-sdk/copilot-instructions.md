## coding-workflow

> Coding workflow cursor must follow

# Coding workflow preferences
- Focus on the areas of code relevant to the task
- Do not touch code that is unrelated to the task
- Avoid making major changes to the patterns and architecture of how a feature works, after it has shown to work well, unless explicitly instructed
- Always think about what other methods and areas of code might be affected by code changes 
- Keep code simple and readable
- Write thorough tests for all functionalities you wrote

Follow this sequential workflow:
1. Write or update existing code
2. Write the incremental unit-test to cover code logic you wrote
3. Test unit-test pass
4. Verify it passes all the tests by running `make test` command
5. Ensue your unit-test has good code coverage for the code you have written

---
> Source: [base/bridge-sdk](https://github.com/base/bridge-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
