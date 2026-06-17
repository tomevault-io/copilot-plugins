## reward-harness

> > **Harness Engineering 最佳实践**: 本文档是目录与入口地图，全部详细知识沉淀于 `docs/` 目录。

# Reward Harness Agent Context Map

> **Harness Engineering 最佳实践**: 本文档是目录与入口地图，全部详细知识沉淀于 `docs/` 目录。
> 
> **版本**: v1.1 | **最后更新**: 2026-04-16

---

## 快速导航

| 目标 | 文档 |
|------|------|
| 先看架构总览地图 | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) |
| 理解架构设计与契约演进 | [docs/DESIGN.md](docs/DESIGN.md) |
| 了解前端约束 | [docs/FRONTEND.md](docs/FRONTEND.md) |
| 查看执行计划 | [docs/PLANS.md](docs/PLANS.md) |
| 了解产品方向 | [docs/PRODUCT_SENSE.md](docs/PRODUCT_SENSE.md) |
| 查看路线图 | [docs/ROADMAP.md](docs/ROADMAP.md) |

---

## 项目结构

```
reward_harness/              # 核心包
autosr/                      # legacy 兼容 shim（从 reward_harness re-export）
├── cli.py                   # CLI入口
├── harness/                 # 搜索会话底座
│   ├── session.py           # SearchSession
│   ├── state.py             # Checkpoint schema
│   └── storage.py           # StateManager
├── search/                  # 搜索算法
├── rm/                      # RM Artifact (阶段B)
└── types.py                 # 统一枚举

docs/                        # 知识库
├── ARCHITECTURE.md          # 架构顶层地图（域与分层）
├── DESIGN.md                # 架构设计
├── FRONTEND.md              # 前端约束与规范
├── PLANS.md                 # 计划管理
├── PRODUCT_SENSE.md         # 产品感
├── ROADMAP.md               # 路线图
├── design-docs/             # 详细设计
├── exec-plans/              # 执行计划
│   ├── active/              # 活跃中
│   ├── completed/           # 已完成
│   └── tech-debt/           # 技术债
├── generated/               # 生成物文档
├── product-specs/           # 产品规格
└── references/              # 参考资料
```

---

## 常用命令

```bash
# 运行搜索（推荐入口）
uv run python -m reward_harness.cli \
  --dataset examples/demo_dataset.json \
  --mode evolutionary \
  --output artifacts/best_rubrics.json

# legacy 兼容入口
uv run python -m autosr.cli \
  --dataset examples/demo_dataset.json \
  --mode evolutionary \
  --output artifacts/best_rubrics.json

# 单元测试
./scripts/run_tests_unit.sh

# 集成测试
export LLM_API_KEY="..."
./scripts/run_tests_integration.sh

# 文档结构与链接校验
uv run python scripts/validate_docs.py
```

---

## 核心约束

1. **所有密钥必须走环境变量** (`LLM_API_KEY`)
2. **保持 legacy CLI 兼容**: `uv run python -m autosr.cli` 继续可用
3. **Harness底座不可回退**: 搜索会话化能力是上层RM/RL的依赖
4. **文档与代码同步**: 修改架构时必须更新对应设计文档

---

## 代码规范

- **命名**: `snake_case` 变量/函数, `PascalCase` 类
- **架构**: `use_cases.py` 编排, `strategies.py` 工具, `factory.py` 组装
- **导入**: 从 `reward_harness.data_models` 导入实体; `reward_harness.types` 导入枚举
- **提交**: 使用 Conventional Commits (`feat:`, `fix:`, `docs:`)

---

*详细内容请查阅 `docs/` 目录下的专项文档。*

---
> Source: [MarsPain/reward_harness](https://github.com/MarsPain/reward_harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
