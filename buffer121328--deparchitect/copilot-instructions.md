## deparchitect

> This project is **DepArchitect**, a Python dependency architecture review and governance tool.

# DepArchitect Agent Rules

## Role

This project is **DepArchitect**, a Python dependency architecture review and governance tool.

You should work from project docs/specs, follow the current phase, keep scope controlled, and turn acceptance criteria into tested implementation.

## Project Context

- Project: **DepArchitect**
- CLI: `deparch`
- Python package namespace: `deparch`
- Tools: OpenCode, omo-slim / oh-my-opencode-slim, OpenSpec
- Methodology: SDD + ATDD
- Development style: **phase-by-phase**, not one-shot generation
- **Virtual environment**: Always activate before running commands: `source .venv/bin/activate`

## Hard Rules

1. **No AI-first logic** — scanning, parsing, graph building, scoring, and rule matching must be deterministic code.
2. **AI is explanation + decision support only** — not core analysis.
3. **Phase-aware** — identify the current phase before implementing; do not skip phases.
4. **Spec-driven** — use OpenSpec / docs as the source of truth before implementation.
5. **ATDD-oriented** — acceptance criteria and tests guide implementation.
6. **Stable CLI** — once introduced, keep commands backward compatible.
7. **Testable always** — analysis logic must work without network/LLM calls.
8. **Deterministic outputs** — same input should produce the same findings, scores, and reports.
9. **Evidence-grounded AI** — AI explanations must be based only on collected findings; never invent risks or facts.

## Project-Level Priority

Prefer project-local configuration and skills before generic behavior:

1. `AGENTS.md`
2. `.opencode/skills/*`
3. `.opencode/commands/*`
4. `openspec/*`
5. `docs/*`

If project-level instructions conflict with generic assumptions, follow the project-level instructions unless the user says otherwise.

## Dynamic Documentation Policy

This project has many documents. Do **not** load every document into context by default.

Instead:

- Start from the task request.
- Identify which phase/spec/doc is relevant.
- Read only the relevant documents or sections.
- Use search/glob when the right document is unknown.
- Prefer targeted reads over dumping entire long docs.
- If requirements are unclear, ask or use exploration before implementing.

## Documentation Lookup

| Situation / Need | Check |
|---|---|
| Starting implementation | `docs/project-plan.md` + relevant `docs/phases/phase-<N>-<name>.md` |
| Working from OpenSpec | Relevant `openspec/changes/<change-id>/` files |
| Understanding product intent | `README.md` |
| Understanding architecture/background | Relevant section of `docs/project-brief.md` |
| Choosing next work | `docs/project-plan.md` |
| Checking acceptance criteria | Relevant phase doc or OpenSpec tasks/spec |
| Project commands/skills | `.opencode/commands/*`, `.opencode/skills/*` |

## OpenSpec Specs 规则

`openspec/specs/` 存放**项目级 capability 规格**，按 `docs/phases/proposals.md` 中定义的 capability 列表组织：

```
openspec/specs/
  <capability-name>/spec.md    # 每个 capability 一个目录
```

**关键区分：**
- `openspec/specs/` = 项目级 capability 规格（来自 `proposals.md`）
- `openspec/changes/<id>/specs/` = 某次 change 的 delta specs（change 归档时同步到主 specs）

**归档时必须同步 delta specs 到主 specs：**
1. 检查 `openspec/changes/<id>/specs/` 是否有 delta specs
2. 将每个 delta spec 同步到对应的 `openspec/specs/<capability>/spec.md`
3. 然后再移动 change 目录到 archive

**不要：**
- 把 Phase 基础设施实现细节（如 cli-skeleton、project-scaffold）当作 capability spec
- 在 `openspec/specs/` 下创建不属于 `proposals.md` capability 列表的目录

## Phase Gates

- Check each phase doc's `前置条件` before starting.
- Treat `不包含内容` as hard scope boundaries.
- Do not declare a phase complete until its `验收标准` and `完成标志` are satisfied.
- During MVP phases, defer P2 items unless the user explicitly asks.
- If a request belongs to a later phase, state the scope mismatch before implementing.


## Execution Checklist

For implementation tasks:

1. Locate the current phase or OpenSpec change.
2. Load only the relevant docs/spec sections.
3. Check phase gates and acceptance criteria.
4. Write/update tests first when practical.
   - ATDD tests: only import from the current phase's new modules. Never re-import previous-phase modules to test their behavior indirectly.
5. Implement within the approved scope.
6. Verify functional, engineering, testing, and documentation expectations where applicable.
7. Report only changes made and verification performed.

## Simplified Architecture Boundaries

Keep responsibilities separated. Do not mix CLI, project loading, dependency parsing, import analysis, scoring, AI explanation, and report rendering in the same module.

## Test Directory Convention

```
tests/
├── acceptance/           # ATDD 验收测试（每个 AC 一个文件）
│   └── test_ac_*.py      # 命名: test_ac_<capability>.py
├── unit/                 # 单元测试（按模块分子目录）
│   ├── scanners/
│   ├── graph/
│   ├── reporters/
│   └── ...
├── integration/          # 集成测试（跨模块）
├── conftest.py           # 共享 fixtures
└── test_*.py             # Phase 0 遗留（不再新增）
```

**规则：**
- **ATDD 验收测试** → `tests/acceptance/test_ac_<name>.py`
- **单元测试** → `tests/unit/<module>/test_<name>.py`
- **集成测试** → `tests/integration/test_<name>.py`
- 不再在 `tests/` 根目录新增测试文件

## Product Boundaries

- MVP is Python-only.
- Support `requirements.txt` and PEP 621 `pyproject.toml` first.
- Prioritize Pydantic v1 → v2 for upgrade review.
- Keep the core engine deterministic; optional AI/agent features belong in a separate copilot layer.
- For current CLI scope, prefer lightweight provider SDK + `asyncio` + Pydantic structured output over heavy agent frameworks unless workflow complexity clearly justifies them.
- Do not build a full web dashboard early.
- Do not implement automatic large-scale refactoring early.
- Do not automatically submit PRs.
- Do not make core analysis depend on LLM/network availability.

---
> Source: [buffer121328/DepArchitect](https://github.com/buffer121328/DepArchitect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
