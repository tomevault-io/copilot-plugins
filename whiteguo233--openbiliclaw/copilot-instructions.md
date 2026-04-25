## openbiliclaw

> 主代码位于 `src/openbiliclaw/`：`agent/` 负责编排，`bilibili/` 负责站点接入，`memory/`、`soul/`、`discovery/`、`recommendation/` 分别承载理解、发现与推荐链路。测试位于 `tests/`，命名采用 `test_*.py`。设计和路线文档集中在 `docs/`，其中 `docs/v0.1-todolist.md` 是当前 v0.1 的开发主线。浏览器插件代码单独放在 `extension/`，其中 `extension/src/` 为脚本源码，`extension/popup/` 为弹窗页面。

# Repository Guidelines

## 项目结构与模块组织
主代码位于 `src/openbiliclaw/`：`agent/` 负责编排，`bilibili/` 负责站点接入，`memory/`、`soul/`、`discovery/`、`recommendation/` 分别承载理解、发现与推荐链路。测试位于 `tests/`，命名采用 `test_*.py`。设计和路线文档集中在 `docs/`，其中 `docs/v0.1-todolist.md` 是当前 v0.1 的开发主线。浏览器插件代码单独放在 `extension/`，其中 `extension/src/` 为脚本源码，`extension/popup/` 为弹窗页面。

## 构建、测试与开发命令
先创建虚拟环境并安装开发依赖：`pip install -e ".[dev]"`。常用检查命令如下：

```bash
ruff format src/ tests/
ruff check src/ tests/
mypy src/
pytest
pytest --cov=openbiliclaw
```

本地体验 CLI 可使用 `openbiliclaw start`、`openbiliclaw profile`、`openbiliclaw recommend`。如修改配置相关逻辑，请同步验证 `openbiliclaw config-show`。`extension/` 当前未声明独立包管理脚本；若修改插件，请在 PR 中写明手动验证步骤。

## 开发顺序与配置约定
v0.1 开发建议以 `docs/v0.1-todolist.md` 为准，按“连接 -> 理解 -> 发现 -> 推荐 -> 学习 -> 插件 -> 稳定交付”的里程碑顺序推进，避免跳过底层依赖直接做上层体验。配置样例使用 `config.example.toml`；本地调试时基于它生成 `config.toml`，并仅在本机保存 API Key、Cookie 等敏感信息。

## 编码风格与命名约定
Python 统一使用 4 空格缩进、类型注解和清晰的模块边界；公开 API 与核心数据结构应补充简洁 docstring。格式化与 lint 由 Ruff 管理，静态类型检查使用 MyPy 严格模式。模块文件名使用小写下划线风格，如 `openai_provider.py`；测试函数采用 `test_<behavior>` 命名。

## 测试要求
新增功能默认同时补充单元测试；涉及真实 B 站或模型服务的流程，优先拆成可 mock 的单元测试，并将真实调用保留为手动或集成测试。v0.1 目标覆盖率参考 `docs/v0.1-todolist.md`，保持在 70% 以上。提交前至少运行 `pytest`，改动接口、配置或类型定义时同时运行 `mypy src/` 和 `ruff check src/ tests/`。

## 提交与 Pull Request 要求
提交信息遵循 Conventional Commits，例如 `feat: add bilibili auth status command`、`fix: validate missing api key`。PR 说明应包含：变更摘要、测试命令与结果、关联任务或文档入口；如改动 CLI 输出或插件页面，请附终端输出或截图。不要提交真实 `config.toml`、Cookie、API Key 或其他本地敏感数据。

## 文档更新要求（强制）
每完成一个 `docs/v0.1-todolist.md` 中的任务并准备合回 main 时，**必须同步更新以下文档**。这不是可选的——缺少文档更新的分支不应合入。

### 必须更新的文档
1. **模块文档** `docs/modules/<模块>.md`：更新"已实现功能"表格和"公开 API"部分，确保新增的类、方法、异常都有记录。模块文档位于 `docs/modules/` 目录下，包括 `llm.md`、`bilibili.md`、`memory.md`、`soul.md`、`cli.md`、`config.md`。
2. **变更日志** `docs/changelog.md`：在对应里程碑下追加一条变更记录，简述本次交付的核心内容。
3. **CLI 命令参考** `docs/modules/cli.md`：如新增或修改了 CLI 命令，更新命令一览表和详细说明。
4. **配置参考** `docs/modules/config.md`：如新增了 `config.toml` 配置项，更新对应段落。

### 按需更新的文档
5. **架构设计** `docs/architecture.md`：如改动涉及跨模块交互、新增模块或数据流变化，更新模块职责和关系描述。
6. **文档导航** `docs/index.md`：如新增了模块文档或模块状态发生变化，更新导航表格。

### 文档模板
所有 `docs/modules/*.md` 遵循统一结构：概述 → 已实现功能（表格）→ 公开 API（代码示例）→ 配置项 → 设计决策。详见现有模块文档作为参考。

---
> Source: [whiteguo233/OpenBiliClaw](https://github.com/whiteguo233/OpenBiliClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
