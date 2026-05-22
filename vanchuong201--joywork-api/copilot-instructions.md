## testing

> JoyWork backend verification and testing expectations


# JoyWork Backend Testing

- This repo has `vitest`, lint, and type-check scripts; for non-trivial backend changes, run the most relevant checks when practical.
- At minimum, prefer `npm run lint` and `npm run type-check` for touched backend code, plus targeted tests if a related suite exists.
- For endpoint changes, verify status codes, auth behavior, success envelope, and error envelope, not just the happy path.
- For Prisma changes, confirm the migration is valid and the updated queries still match the selected fields and nullability assumptions.
- Remember the current lint and type-check flow is centered on `src`; if a task touches scripts or non-covered paths, say clearly what was and was not validated.
- Never report tests as passed unless they were actually run in this task.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
