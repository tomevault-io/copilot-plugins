## cnequity

> 所有代码在 `src/cnequity/`：

# 开发约定

本文描述包结构、分层规则与提交约定。

---

## 包布局

所有代码在 `src/cnequity/`：

| 子包 | 职责 |
|------|------|
| `domain` | 契约（schema、数据集元数据） |
| `adapters` | 源 I/O，薄 |
| `steps` | 采集编排单元 |
| `orchestrator` | 引擎与 manifest |
| `storage` | 湖读写 |
| `derive` | 可重算派生 |
| `quality` | 审计 |
| `query` | 只读消费 |
| `config` / `cli` | 配置与入口 |

---

## Steps 按数据层分文件

| 文件 | 数据层 |
|------|--------|
| `reference.py` | L0 |
| `bars.py` | L1 |
| `events.py` | L2 |
| `fundamentals.py` | L3 |
| `capital.py` | L4 |
| `structure.py` | L5 |
| `macro_risk.py` | L6/L8 |
| `research.py` | L4/L7 |
| `finalize.py` | 收尾 |

新数据集 step 放入对应层文件；新层则新建文件并在 `steps/__init__.py` import。

---

## 数据契约优先

1. `domain/schemas.py` — 列类型 + PK
2. `domain/datasets.py` — `DatasetSpec`
3. `tests/unit/test_dataset_registry.py` 保持同步

写 staging 前必须 `validate_dataframe()`。

---

## Adapters 要薄

- 协议、分页、源字段映射 → `adapters/`
- 增量窗口、写 staging、manifest → `steps/`
- 不含 DuckDB / compact 逻辑

---

## 提交约定

提交信息用 [Conventional Commits](https://www.conventionalcommits.org/)：

```
<type>(<scope>): <简短描述>
```

常用 type：`feat` `fix` `docs` `refactor` `perf` `test` `chore` `ci`。
scope 用模块名（`ths_official`、`quality`、`symbols`、`cli`…）。描述用祈使句、不加句号。
正文说明**为什么**这样改，而不是复述 diff。破坏性变更加 `!`，并在正文写明迁移方式。

---

## CI 门禁

| CI job | 本地等价命令 |
| --- | --- |
| `quality` — lint | `ruff check . && ruff format --check . && shellcheck scripts/*.sh` |
| `quality` — 离线基准 | `python scripts/benchmark_offline.py --check --max-elapsed-seconds 10 --max-concurrency 2` |
| `quality` — 前端产物 | `cd frontend && npm ci && npm run check` |
| `test` | `pytest` |
| `Docs` | `mkdocs build --strict` |

`test` 在 Linux（Python 3.10/3.12/3.13/3.14）、Windows 与 macOS 上各跑一遍。

**Windows 是一等目标**：路径、文件锁和换行差异都算回归。往 TOML 里写路径务必用 `path_for_toml()`，
裸 `Path` 在 `C:\Users\…` 下会变成非法转义。

**测试不要读 gitignore 的文件**：`configs/cnequity.toml` 是本地配置，在干净检出中不存在。

本地预览文档站：

```bash
mkdocs serve      # http://127.0.0.1:8000
```

---

## 测试原则

- **单元测试离线**：mock/monkeypatch 网络
- **网络测试**：`@pytest.mark.network`，单独运行
- **集成测试**：`@pytest.mark.integration`，`tests/integration/`

```bash
ruff format .
ruff check .
pytest tests/unit
pytest tests/integration  # 可选
```

全局超时 120s（`pyproject.toml`）。

---

## 架构决策

非平凡设计选择写 ADR：`docs/adr/`，复制 `0000-template.md`。

---

## 代码风格

- Ruff：line-length 100，py310
- 类型注解：新代码推荐完整标注
- 注释：仅解释非显而易见的业务/协议细节

---

## 相关文档

- [新增数据集](adding-dataset.md)
- [测试](testing.md)
- [模块索引](../modules/README.md)

---
> Source: [rootSunc/CNEquity](https://github.com/rootSunc/CNEquity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
