## copool

> - When the user says `提交 TestFlight 审核`, `提交外部测试审核`, or similar wording, always treat it as `submit the build for TestFlight external beta review`, not App Store review.

# Copool Repository Rules

## Release Terminology

- When the user says `提交 TestFlight 审核`, `提交外部测试审核`, or similar wording, always treat it as `submit the build for TestFlight external beta review`, not App Store review.
- Do not start App Store submission work unless the user explicitly says `App Store 审核`, `提交 App Store`, `提交苹果正式审核`, or equivalent.
- For release tasks that mention both TestFlight and review in the same request, default to:
  1. upload to TestFlight
  2. add tester groups if needed
  3. submit TestFlight external beta review
  Only start App Store submission if the user separately and explicitly asks for it.

---
> Source: [AlickH/Copool](https://github.com/AlickH/Copool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
