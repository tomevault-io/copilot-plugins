## tasks

> Rules for editing Task implementations.


# Task Implementation Rules

- Inherit from `BaseTask`. Implement `setup` and `run`. `teardown` is optional.
- Register via `@register_task("snake_case_name")`. Registered name must match
  the file basename.
- `run()` returns a `TaskResult(metrics=..., artifacts=...)`. Status defaults to SUCCESS.
- Use `self.ctx.tracker.log(...)` for metrics during training, not `print`.
- Use `self.ctx.checkpoints_dir` / `self.ctx.artifacts_dir` / `self.ctx.results_dir`
  for output paths. Don't construct paths from cwd.
- For HF Trainer-based tasks, configure with `report_to="none"` and `seed=self.ctx.seed`,
  and let the Trainer own the loop inside `run()`. No "dual mode" needed.
- For mixins (federated/agent), inherit them additionally:
  `class MyTask(BaseTask, FederatedMixin): ...`

---
> Source: [qoham/experiment_code_template](https://github.com/qoham/experiment_code_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
