## convex-doctor

> Run convex-doctor after backend changes to maintain code health score


# convex-doctor

After making changes to files in `convex/`, run `npx convex-doctor --score` to verify the health score stays at or above 90.

If the score drops below 90:
1. Run `npx convex-doctor -v` to see detailed findings.
2. Fix errors in order: correctness, security, performance.
3. Only suppress rules in `convex-doctor.toml` for documented false positives.

The project configuration lives in `convex-doctor.toml` at the project root. Current suppressions have inline comments explaining why each rule is disabled.

---
> Source: [waynesutton/better-todo](https://github.com/waynesutton/better-todo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
