## 020-task-protocol

> - Start from docs/issue/TASKS.md and pick a task with status PENDING.


# Task protocol (docs/issue/TASKS.md)

- Start from docs/issue/TASKS.md and pick a task with status PENDING.
- Before coding, produce:
  1) goal in 2-4 sentences
  2) acceptance criteria
  3) explicit scope: 1-3 target files
- Do not implement "TASK-xxx" without an explicit file scope.
- After completion:
  - mark COMPLETED
  - add a short summary of changes
  - list commands/tests executed

## Feature Flags Cleanup

- **If task used developer flags (`category: 'experiments'`):** After feature deployment, **mandatory cleanup:**
  - Remove flag definition from `config/pennant.php`
  - Remove flag class from `app/Features/`
  - Remove conditional code (`Feature::active()` checks)
  - Update tests (remove flag-related test cases)

---
> Source: [lukaszzychal/moviemind-api-public](https://github.com/lukaszzychal/moviemind-api-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
