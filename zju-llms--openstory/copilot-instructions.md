## openstory

> This file provides guidance to Claude Code when working in `examples/WorldKernel`.

# CLAUDE.md — WorldKernel

This file provides guidance to Claude Code when working in `examples/WorldKernel`.

---

## Project Purpose

WorldKernel is a **text-to-interactive-world generation system** built on top of Agent-Kernel.

用户输入一句自然语言（如"创建最后一部中的霍格沃茨魔法学院"），系统经过多阶段 pipeline 生成完整的世界内容，最终适配 Agent-Kernel 启动可交互的多智能体仿真。

---

## Scope Constraint

**Only modify files under `examples/WorldKernel/`.** Sibling examples、`packages/`、repo root 均为只读依赖。

Runtime target: **`agentkernel_distributed`**.

---

## Pipeline Overview

```
User NL input
    │
    ▼
Stage 1  — 理解与模版准备                    ← IMPLEMENTED
    │       intent_parser → world_type_classifier → generation_planner → ontology_selector
    │       Output: templates/<session_id>/ (configs, models, generated/)
    ▼
Stage 2  — 世界内容语义生成                  ← PARTIALLY IMPLEMENTED
    │       InitDAGRunner 按拓扑分波执行 generation tools
    │       LocationGenerationTool: generate → review → retry → validate (完整)
    │       CharacterGenerationTool / PathGraphTool / RelationGraphTool: 接口已声明, run() 未实现
    ▼
Stage 3  — 校验、修复与适配                  ← NOT YET
    │       Patch Validation → AK Adapter
    ▼
Stage 4  — Agent-Kernel 仿真                ← NOT YET
            Tick-based multi-agent simulation
```

---

## Directory Structure

```
examples/WorldKernel/
├── CLAUDE.md
├── pyproject.toml
├── .env.example                    ← WORLDKERNEL_API_KEY
│
├── src/worldkernel/
│   ├── server.py                   ← FastAPI: API routes + static frontend (mount at /)
│   ├── constraints.py              ← GenerationConstraints (max_locations/max_characters)
│   │
│   ├── stage1/                     ← Stage 1 pipeline
│   │   ├── pipeline.py             ← run_stage1(): orchestrates modules, saves files + codegen
│   │   ├── intent_parser.py        ← parse_intent(): 深度意图解析
│   │   ├── world_type_classifier.py← build_world_template(): 世界模版构建
│   │   ├── generation_planner.py   ← plan_generation(): 生成计划 + 约束截断
│   │   ├── ontology_selector.py    ← generate_templates(): 6类实体模版 (并行LLM)
│   │   ├── world_spec.py           ← SessionInfo model
│   │   ├── types.py                ← IntentResult, WorldTemplate, GenerationPlan, EntityTemplate...
│   │   └── prompts/                ← Stage1 prompt templates (.md)
│   │
│   ├── llm/                        ← LLM call layer (all stages share)
│   │   ├── client.py               ← init(), chat(), chat_json() + JSON extract/repair
│   │   └── config_loader.py
│   │
│   ├── architect/                  ← Stage 2 semantic generation
│   │   ├── __init__.py             ← Public API re-exports
│   │   ├── init/                   ← Stage1→Stage2 桥接: 加载 artifacts, 编译 context
│   │   │   ├── loader.py           ← InitInputLoader.from_session_root()
│   │   │   ├── compilers.py        ← ContractCompiler, ExecutionDAGCompiler, SeedResolver
│   │   │   ├── models.py           ← InitBuildContext, ExecutionDAG, ResolvedSeed, Stage1ArtifactBundle
│   │   │   └── pipeline.py         ← compile_stage1_init_context()
│   │   ├── registry/               ← Schema + Tool 注册表
│   │   │   ├── core.py             ← SchemaRegistry, ToolRegistry
│   │   │   └── schema_loader.py    ← 从 session models/ 动态加载 Pydantic 模型
│   │   ├── semantic/               ← 生成执行核心
│   │   │   ├── runner.py           ← InitDAGRunner (拓扑分波, asyncio.gather)
│   │   │   ├── state.py            ← SemanticGenerationState (result_store)
│   │   │   ├── storage.py          ← save_semantic_artifacts()
│   │   │   └── bundle.py / repository.py / models.py
│   │   └── tools/                  ← Stage2 生成工具
│   │       ├── base.py             ← BaseStage2Tool, Stage2ToolRequest/Result/Context
│   │       ├── generation.py       ← CharacterGenerationTool, PathGraphTool, RelationGraphTool (stubs)
│   │       ├── identity_allocator.py ← IdentityAllocator + IdentityRegistry (确定性ID预分配)
│   │       └── generators/
│   │           ├── base_generator.py ← 共享工具函数 (prompt构建, schema内省, 校验)
│   │           ├── location_generator.py ← LocationGenerationTool (generate→review→retry 完整实现)
│   │           └── prompts/        ← Stage2 生成/评审/重试 prompt templates
│   │
│   └── models/                     ← 共享 Pydantic models
│       └── agent_schema.py
│
├── configs/
│   ├── models.yaml                 ← LLM config (OpenAI-compatible)
│   ├── architect.yaml              ← generation_constraints (max_locations: 7, max_characters: 10)
│   ├── simulation.yaml
│   └── storage.yaml
│
├── templates/                      ← Output: <session_id>/ (Stage1+Stage2 产出)
│   └── <session_id>/
│       ├── generated/
│       │   ├── artifact_manifest.json   ← Stage2 入口索引
│       │   ├── world_template.json
│       │   ├── plan/                    ← ontology_hints, instance_seed_catalog, execution_plan, world_background
│       │   ├── templates/<entity>/      ← 各实体各维度原始 JSON
│       │   └── artifacts/               ← Stage2 生成结果持久化
│       ├── configs/<entity>/            ← YAML 维度定义 (agent, location, path, relation)
│       └── models/                      ← 自动生成的 Pydantic 模型 + schema_manifest.json
│
└── frontend/                       ← Static HTML/CSS/JS (Vite dev optional), served by FastAPI at /
    ├── index.html
    ├── app.js
    ├── style.css
    ├── vite.config.ts
    └── package.json
```

---

## Setup and Running

```bash
pip install -e "examples/WorldKernel"
cp examples/WorldKernel/.env.example examples/WorldKernel/.env  # fill API key

python -m worldkernel.server          # http://localhost:8100/

```

---

## LLM Integration

- Config: `configs/models.yaml` (OpenAI-compatible format, key from `WORLDKERNEL_API_KEY` env var)
- Single entry point: `llm/client.py` — `chat()` and `chat_json()` (auto-strips markdown fences + extracts JSON + repairs common LLM errors)
- All modules import `from worldkernel.llm.client import chat_json`, never instantiate SDK directly
- Server lifespan calls `llm_client.init()` at startup

---

## API Endpoints

| Method | Path | Description |
|---|---|---|
| POST | `/api/stage1/parse` | `{"input": "..."}` → runs Stage1 pipeline → returns SessionInfo |
| GET | `/api/stage1/session/{session_id}` | Returns file list |
| GET | `/api/stage1/session/{session_id}/{path}` | Returns file content (JSON/YAML) |
| POST | `/api/stage2/generate/{session_id}` | Runs Stage2 DAG generation → returns completed_steps + locations stats |

---

## Key Design Patterns

- **约束系统**: `constraints.py` 定义 `GenerationConstraints`，在 Stage1 (prompt注入 + 后处理截断) 和 Stage2 (SeedResolver) 双重生效
- **确定性 ID**: `IdentityAllocator` 在任何 LLM 调用前为所有种子预分配 entity ID，LLM 必须使用预分配的 ID
- **DAG 执行**: `InitDAGRunner` 按拓扑分波 (`_topological_waves`) 并行执行同波步骤
- **质量保证**: LocationGenerationTool 三阶段循环: generate → review (打分+纠正) → 低于阈值则 retry
- **动态 Schema**: Stage1 自动生成 Pydantic 模型代码，Stage2 通过 `SchemaRegistry` 动态加载，使得生成逻辑与世界结构解耦
- **模型代码生成**: `pipeline.py` 中 `_generate_pydantic_models()` 从 configs YAML → Python Pydantic 模型

---

## Conventions

- Pydantic v2 models for all data structures
- All pipeline modules are pure async functions with typed signatures
- Prompt templates: `src/worldkernel/stage1/prompts/*.md` (Stage1), `src/worldkernel/architect/tools/generators/prompts/` (Stage2)
- `templates/` contains session outputs (gitignored except committed example sessions)
- Frontend: static HTML/JS/CSS, no build step required, served at `/`

---

## Reference (read-only)

| Path | What to look at |
|---|---|
| `examples/story_of_the_stone/` | Server setup, registry, config structure |
| `packages/agentkernel-distributed/` | Builder API, available components |

---
> Source: [ZJU-LLMs/OpenStory](https://github.com/ZJU-LLMs/OpenStory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
