## webnovel-writer-opencode

> - **Python 3.10+**（不是3.9）

# AGENTS.md - Webnovel Writer 开发指南

## 项目要求
- **Python 3.10+**（不是3.9）
- OpenCode 运行环境

## 快速命令
```bash
# 安装
python install.py

# 核心命令
python .opencode/scripts/webnovel.py <command>
python .opencode/scripts/webnovel.py where              # 当前项目根
python .opencode/scripts/webnovel.py checkers list    # 审查器列表

# 测试
python .opencode/scripts/run_new_tests.py            # 新增模块测试
python .opencode/scripts/run_all_tests.py            # 全量测试
```

## 项目结构
```
项目目录/
├── .webnovel/          # state.json, index.db
├── 正文/               # 章节 Markdown
├── 设定集/             # 角色/势力设定
├── 大纲/               # 卷纲/章纲
└── .env               # API配置
```

## 架构要点

### 1. 异常基础层
- 统一异常体系：`WebnovelError`, `StateManagerError`, `IndexManagerError`, `APIClientError`, `ConfigError`
- 渐进式接入，不破坏现有功能

### 2. 性能埋点
- 观测数据写入：`{project_root}/.webnovel/observability/data_agent_timing.jsonl`
- 覆盖：state_manager.save_state, context_manager.build_context, api_client.embed, checkers_manager.run_layered_checkers

### 3. RAG 后端抽象
```python
from data_modules.rag_backend import BackendFactory, VectorSearchBackend
backend = BackendFactory.create("vector")
```

### 4. 分层审查
- **Code Checkers**：`world-consistency`, `DebtTracker` - 确定性检查
- **LLM Agents**：通过 `llm_invoker.py` 调用
- 分层执行：`run_layered_checkers(run_llm=False)` → Code层；`run_llm=True` → 完整

### 5. 债务系统
```python
from data_modules.debt_tracker import DebtTracker, DebtPriority
tracker = DebtTracker()
tracker.create_debt('explicit', '神秘玉佩', chapter=1, priority=DebtPriority.HIGH)
```

### 6. Graph-RAG 持久化
```python
from data_modules.temporal_graph import TemporalGraphIndex
graph = TemporalGraphIndex()
graph.load_from_db("index.db")
graph.save_to_db("index.db")
```

### 7. 题材预设
```python
config.world_preset = "xianxia"  # xianxia/urban/fantasy/scifi
```

## 新增模块
| 模块 | 路径 | 作用 |
|------|------|------|
| `exceptions.py` | `data_modules/` | 统一异常体系 |
| `rag_backend.py` | `data_modules/` | RAG后端抽象 |
| `config_defaults.py` | `data_modules/` | 配置默认值 |
| `config_presets.py` | `data_modules/` | 世界观预设 |
| `observability.py` | `data_modules/` | 性能埋点 |

## 测试覆盖
- 28个新增模块测试：test_exceptions.py, test_rag_backend.py
- 306个核心功能测试通过 (95.3%)

## Key Files
| 文件 | 作用 |
|------|------|
| `webnovel.py` | CLI入口 + COMMAND_REGISTRY |
| `config.py` | 配置+题材预设 |
| `debt_tracker.py` | 债务追踪 |
| `llm_invoker.py` | LLM调用 |
| `image_generator.py` | ModelScope 图片生成 |
| `checkers/registry.yaml` | 审查器配置 |

## 新手引导
详见 `docs/新手引导.md`

---

## 功能完成状态
| 模块 | 状态 |
|------|------|
| 基础架构 | ✅ |
| 一致性保障 | ✅ |
| 通用化 | ✅ |
| LLM集成 | ✅ |
| 图片生成 | ✅ |
| 后端优化 | ✅ |
| 性能观测 | ✅ |

---
> Source: [lujih/webnovel-writer-opencode](https://github.com/lujih/webnovel-writer-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
