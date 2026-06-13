## ftp1-policy

> - Core Python package: `src/openpi/` (models, policies, training, shared utilities).

# Repository Guidelines

## Project Structure & Module Organization
- Core Python package: `src/openpi/` (models, policies, training, shared utilities).
- Workspace package: `packages/openpi-client/` (client library code and tests).
- Training/inference entrypoints: `scripts/` (for example `train.py`, `serve_policy.py`, `compute_norm_stats.py`).
- Dataset and embodiment tooling: `data_processing/` and `UniVTAC/`.
- Usage examples and platform-specific guides: `examples/` and `docs/`.
- Tests are colocated with code and scripts (see `src/**/_test.py`, `packages/**/_test.py`, `scripts/test_*.py`).

## Build, Test, and Development Commands
- Install deps and dev tools:
  ```bash
  GIT_LFS_SKIP_SMUDGE=1 uv sync --all-extras --dev
  GIT_LFS_SKIP_SMUDGE=1 uv pip install -e .
  ```
- Run full test suite (CI-aligned):
  ```bash
  uv run pytest --strict-markers -m "not manual"
  ```
- Run lint and format:
  ```bash
  uv run ruff check .
  uv run ruff format .
  pre-commit run -a
  ```
- Typical training workflow:
  ```bash
  uv run scripts/compute_norm_stats.py --config-name <config>
  uv run scripts/train.py <config> --exp-name=<run_name> --overwrite
  ```

## Coding Style & Naming Conventions
- Python 3.11+ (`pyproject.toml`).
- Ruff is the source of truth for lint/format; max line length is 120.
- Follow existing import style (single-line imports and sorted sections as enforced by Ruff/isort settings).
- Prefer descriptive module names; tests should follow existing patterns: `<module>_test.py` or `test_<feature>.py`.

## Testing Guidelines
- Framework: `pytest`; configured `testpaths` are `src`, `scripts`, and `packages`.
- Use `@pytest.mark.manual` only for tests requiring manual/hardware execution.
- Keep automated tests deterministic and runnable via `uv run pytest --strict-markers -m "not manual"`.

## Commit & Pull Request Guidelines
- Recent history favors short, imperative subjects (for example `fix ...`, `feat: ...`).
- Keep commits focused (one logical change per commit) and include context in the body when needed.
- PRs should include:
  - clear title and description,
  - linked issue/discussion when relevant,
  - passing `pre-commit`, Ruff, and pytest checks,
  - reproduction details/screenshots/logs for behavior or UI changes.



## 文档管理原则

**核心原则**: Skills 是本项目的核心文档库，负责知识管理和传承。

### Skill 文档管理

1. **知识归档到 Skills**
   - 任何有价值的信息（工作流程、问题解决方案、架构设计、调试经验）都应追加到最相关的 skill 中
   - 如果没有合适的 skill，使用 `skill-creator` 创建新 skill
   - Skill 文档应持续更新，反映项目最新状态

2. **任务完成后更新 Skills**
   - 完成功能开发：更新相关 skill 的工作流程和使用说明
   - 修复 bug：在 skill 的 "Common Issues" 或 "Known Issues" 部分记录问题和解决方案
   - 性能优化：更新 skill 中的最佳实践和配置参数
   - 架构变更：更新 skill 中的架构图和核心概念说明

3. **Skills 作为协作基础**
   - 其他开发者通过阅读 skill 快速了解子系统
   - Skill 提供足够的上下文用于修改相关代码
   - Skill 记录设计决策和历史演进

4. **最小化其他文档**
   - **不要创建**（除非必要）：
     - `docs/` 下的独立文档
     - 子目录的 `README.md`
     - 任务完成报告（临时性文档）
   - **原因**：分散的文档难以维护，容易过时，skill 集中管理更高效

5. **文档层次**
   - **CLAUDE.md**（本文件）：项目总览、环境配置、快速开始
   - **Skills**（`.claude/skills/`）：子系统详细文档、工作流程、问题排查
   - **代码注释**：实现细节、算法说明

### 示例：正确的文档更新流程

```
任务：实现 delay1 回测功能
1. 开发功能 → 测试验证
2. 更新 rolling-train skill:
   - 添加 delay1 预测生成说明
   - 更新回测工作流程
   - 简要记录发现的 bug 和 workaround
3. 清理临时测试脚本和代码
4. ✅ 完成（无需创建独立报告）
```

---
> Source: [michaelyuancb/ftp1-policy](https://github.com/michaelyuancb/ftp1-policy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
