## htn-planner

> - Store logs from manual or agent-run builds, tests and diagnostics in

# Workspace housekeeping

- Store logs from manual or agent-run builds, tests and diagnostics in
  `build/logs/`, creating the directory before redirecting output.
- Use descriptive filenames within that directory, such as
  `build/logs/recursion-debug-tests.log`. Resolve the path relative to the
  repository root even when running a command from a project subdirectory.
- Keep the existing SDK validation scripts' logs in their dedicated build
  directories. Those scripts manage their own output locations.
- When documenting a local validation run, reference its actual log path.

---
> Source: [urosidoki/htn_planner](https://github.com/urosidoki/htn_planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
