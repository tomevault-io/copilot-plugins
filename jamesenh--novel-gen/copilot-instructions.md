## novel-gen

> 1. 模块化设计：每一步小说生成流程都必须是一个独立 chain/LangGraph 节点。


## 项目哲学（必须遵守）

1. 模块化设计：每一步小说生成流程都必须是一个独立 chain/LangGraph 节点。
2. 结构化输出优先：所有链的输出必须对应一个 Pydantic 模型。
3. 可迭代可修改：每个 chain/节点必须可以独立运行，不依赖 UI。
4. 状态管理：使用 LangGraph 管理工作流状态，链之间通过 LangGraph 状态传递信息，外部持久化仍使用 JSON 文件 (projects/*)。
5. LangChain + LangGraph 架构：LangChain 负责具体业务逻辑，LangGraph 负责工作流编排。
6. 数据结构（世界观/角色/大纲/LangGraph 状态）必须放在 models.py
7. LangChain/LangGraph 不能内嵌业务结构，避免耦合
8. Prompt 要高度结构化：避免使用自由输出，要尽量使用 JSON schema 约束。
9. 每个节点的逻辑是：
    LangGraph State → 提取输入 → PromptTemplate → LLM → OutputParser → Python对象 → 更新 LangGraph State
10. 工作流定义：小说生成流程使用 LangGraph StateGraph 定义，包含节点和边的流转逻辑

## 项目结构要求

所有代码必须写在以下模块结构中：

```
novelgen/
  models.py           # 所有数据结构(Pydantic)，包括LangGraph状态
  config.py           # 配置管理
  llm.py              # LLM初始化
  chains/             # LangChain处理链，作为LangGraph节点
    world_chain.py
    theme_conflict_chain.py
    characters_chain.py
    outline_chain.py
    chapters_plan_chain.py
    scene_text_chain.py
  runtime/            # 运行时与工具
    orchestrator.py   # 当前编排器（将逐步迁移）
    workflow.py       # LangGraph工作流定义
    summary.py        # 章节/全书摘要
    revision.py       # 修订机制
```

## models.py 内必须定义

- Settings
- WorldSetting
- ThemeConflict
- Character
- CharactersConfig
- Outline
- ChapterPlan
- ScenePlan
- GeneratedScene
均使用 Pydantic BaseModel。

## LangChain + LangGraph 代码规范

### LangChain 链规范
1. 所有链必须使用：
    - ChatPromptTemplate
    - LLMChain 或 RunnableSequence
    - PydanticOutputParser 或 with_structured_output
2. 所有提示必须包含：
    - 「你的任务」
    - 「输入说明」
    - 「输出格式（JSON schema）」
    - 「注意事项」
3. 输出严格使用 JSON，不允许 Markdown 包裹 JSON。
4. 使用 langchain 1.0+ 的版本的语法

### LangGraph 工作流规范
1. 使用 StateGraph 定义工作流，包含节点和边的流转逻辑
2. LangGraph 状态必须通过 Pydantic 模型定义
3. 每个节点函数必须接收状态并返回更新后的状态
4. 边的定义必须清晰，支持条件分支
5. 必须添加工作流的可视化支持（使用 .draw() 方法）
6. 使用 langgraph 1.0+ 的版本的语法

## 项目运行

1. 使用uv,不要直接使用系统默认的python

## git操作

1. commit msg使用中文描述

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamesenh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
