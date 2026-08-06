## march-cli

> 和用户讨论机制设计时，先用简单结构图讲清楚“现在系统是什么样”和“改完以后会变成什么样”，再给原则和边界。不要先展开代码文件、函数名、测试清单或长实现方案。

## 设计讨论沟通规则

和用户讨论机制设计时，先用简单结构图讲清楚“现在系统是什么样”和“改完以后会变成什么样”，再给原则和边界。不要先展开代码文件、函数名、测试清单或长实现方案。

### 讨论结构

- 先抓一个主轴，不同时展开多个方向。
- 优先画数据/职责流向图，而不是列文件路径。
- 文件路径尽量少，只在定位现有实现或准备动代码时出现。
- 先说判断和原则，再说落地；实现细节必须服务于前面的原则。

### 数据流向图风格

- 用单个盒状图表达主链路，优先展示“谁调用谁、数据往哪流、哪些部分保持不变”。
- 图里可以标注 `现有`、`新`、`不变`，但不要塞实现细节；实现细节放在图后说明。
- 横向用 `→` 表示主调用链，纵向用 `↓` 表示继续流转，分支用 `├──` / `└──` 表示可选 provider、模块或责任分叉。
- 对新增能力，画在现有链路旁边，明确它接入哪里、复用哪里、不会影响哪里。
- 示例：

```text
│ 用户 → Agent → Tools (现有)                         │
│                 │                                   │
│                 ├── 新: image_generate tool         │
│                 │       ↓                           │
│                 │   ImageGen Provider Registry      │
│                 │       ├── openai-codex ← OAuth    │
│                 │       ├── openai       ← API Key  │
│                 │       └── fal          ← FAL Key  │
│                 │                                   │
│                 ↓                                   │
│            Model Provider (现有, 不变)              │
│                 ↓                                   │
│            文本 completion                          │
```

### 文档表达

- 表达和写文档都遵守“少即是多”：只写最重要的判断、原则和边界，其他内容适当补充，不自由发挥。
- 按信息金字塔组织内容：先给总判断，再按同一分类维度拆分；同层内容必须同类，不把不同性质的规则混在一个列表里。

### 工程实现

- 核心机制优先追求“优雅架构”，而不是“最小的局部补丁”：先判断责任边界，再做最小改动。优先级是正确职责边界、代码设计优雅，能够合理得调整系统行为并保持系统的自洽
- 保持功能内聚：代码超过 300 行时，按内聚原则把同一块逻辑拆出；如果新增逻辑在原代码中只有较少接入点，设计时就应拆出，再接回原代码。
- 保持目录边界清晰：单个源码目录直属文件超过 10 个时，按职责边界拆出子目录，不用杂项目录或机械搬运凑数。
- 暂时不要修改 WebUI；WebUI 还没开启开发，除非用户明确要求 WebUI 相关改动。

### 发布形态

- 发布形态保持源码发布：npm 包带 `bin/` 和 `src/`，资源文件随源码目录一起发布；不优先追求单文件 bundle。
- 每次发布版本前，必须至少新增或实质修改一篇 GitHub/VitePress 文档；不能只是发布日志，优先补充 March 内部机制剖析或使用指南，例如 provider 配置、provider share/reuse、runtime/context/tool/memory/browser/office/gateway 边界。


### 编码原则
- 每完成一个功能或修改一个功能都需要提交commit
- 日常开发验证优先使用 `:fast` 测试脚本；只有发布前、CI 或涉及高风险改动时再跑全量 smoke。



## 术语表

| 术语 | 中文名 | 定义 |
|---|---|---|
| Agent Run | 运行轮次 | 一次用户输入触发的完整智能体执行过程。一个 Agent Run 可以包含一个或多个 Model Call、零个或多个 Tool Call；当模型不再请求工具并产出最终回复时，该 Agent Run 结束。 |
| Model Call | 模型调用 | pi-agent 基于当前 transcript / provider payload 向模型发起的一次请求，并接收一次模型输出。一个 Agent Run 内可能发生多次 Model Call。 |
| Dialog Entry | 对话条目 | 单次 Model Call 的 payload 中 `messages` 数组里的单个元素，包含 `role` 和 `content`。全称 Dialog Entry，简称 Entry。一个 Model Call 可以包含多个 Dialog Entry，其中 `role=tool` 的 Entry 即工具结果条目。 |
| Tool Call | 工具调用 | 模型请求 March 执行一个工具的动作。Tool Call 不是 Agent Run，也不是 Model Call；它通常发生在一次 Model Call 的输出之后。 |
| Context Assembly | 上下文组装 | March 在 Agent Run 开始时，从各 context layer 读取当前事实并生成初始上下文的过程。当前实现不是每次 Model Call 都重新组装 March context。 |
| Agent-run-start Context Assembly | 运行轮次开始上下文组装 | Agent Run 开始时执行的 Context Assembly；当前会生成初始 system prompt / user context，并注入 user recall。 |
| Pi Transcript Append | Pi 对话追加 | Agent Run 过程中，pi-agent 将 assistant 输出、Tool Call、工具结果、hidden steer message 等追加到当前 transcript，用于后续 Model Call。 |
| Provider Payload Assembly | Provider 请求组装 | 每次 Model Call 前，pi/provider 将当前 transcript、工具定义和 provider 选项转换成最终 provider request payload 的过程。March 可在 `before_provider_request` 做有限 payload 调整。 |
| Layer | 上下文层 | 组成初始上下文的独立上下文模块。每个 layer 以 `[name]` 格式的 header 开头，后接该层对应的结构化文本。当前 layers 包括 `system_core`、`injections`、`session_identity`、`project_context`、`recent_chat` 五个。Agent-run-start Context Assembly 按固定顺序将这些 layers 组装为 Agent Run 的初始上下文。 |
| Diagnostic | 诊断信息 | 编译器、类型检查器、linter 或语言服务器对当前工作区产生的错误、警告和提示，等价于 VS Code Problems 面板里的结构化问题列表。 |
| Model-specific system prompt | 模型专属系统提示 | `system_core` 中针对具体 `modelId` 选择的 prompt 文件；用于适配不同模型的行为差异，不按 provider 区分。provider 只负责请求路由、鉴权和 transport，不参与 prompt 选择。 |

---
> Source: [liuzhengdongfortest/march-cli](https://github.com/liuzhengdongfortest/march-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
