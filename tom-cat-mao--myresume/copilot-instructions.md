## myresume

> **触发条件**：当需要执行以下操作时，自动激活本模块：


  # Grok Search Enhance 系统提示词（详细版）

  ## 0. Module Activation
  **触发条件**：当需要执行以下操作时，自动激活本模块：
  - 网络搜索 / 信息检索 / 事实核查
  - 获取网页内容 / URL 解析 / 文档抓取
  - 查询最新信息 / 突破知识截止限制

  ## 1. Tool Routing Policy

  ### 强制替换规则
  | 需求场景 | ❌ 禁用 (Built-in) | ✅ 强制使用 (GrokSearch) |
  | :--- | :--- | :--- |
  | 网络搜索 | `WebSearch` | `mcp__grok-search__web_search` |
  | 网页抓取 | `WebFetch` | `mcp__grok-search__web_fetch` |
  | 配置诊断 | N/A | `mcp__grok-search__get_config_info` |

  ### 工具能力矩阵

| Tool | Parameters | Output | Use Case |
|------|------------|--------|----------|
| `web_search` | `query`(必填), `platform`/`min_results`/`max_results`(可选) | `[{title,url,content}]` | 多源聚合/事实核查/最新资讯 |
| `web_fetch` | `url`(必填) | Structured Markdown | 完整内容获取/深度分析 |
| `get_config_info` | 无 | `{api_url,status,test}` | 连接诊断 |
| `switch_model` | `model`(必填) | `{status,previous_model,current_model}` | 切换Grok模型/性能优化 |
| `toggle_builtin_tools` | `action`(可选: on/off/status) | `{blocked,deny_list,file}` | 禁用/启用官方工具 |


  ## 2. Search Workflow

  ### Phase 1: 查询构建 (Query Construction)
  1.  **意图识别**：分析用户需求，确定搜索类型：
      - **广度搜索**：多源信息聚合 → 使用 `web_search`
      - **深度获取**：单一 URL 完整内容 → 使用 `web_fetch`
  2.  **参数优化**：
      - 若需聚焦特定平台，设置 `platform` 参数
      - 根据需求复杂度调整 `min_results` / `max_results`

  ### Phase 2: 搜索执行 (Search Execution)
  1.  **首选策略**：优先使用 `web_search` 获取结构化摘要
  2.  **深度补充**：若摘要不足以回答问题，对关键 URL 调用 `web_fetch` 获取完整内容
  3.  **迭代检索**：若首轮结果不满足需求，**调整查询词**后重新搜索（禁止直接放弃）

  ### Phase 3: 结果整合 (Result Synthesis)
  1.  **信息验证**：交叉比对多源结果，识别矛盾信息
  2.  **时效标注**：对时间敏感信息，**必须**标注信息来源与时间
  3.  **引用规范**：输出中**强制包含**来源 URL，格式：`[标题](URL)`

  ## 3. Error Handling

  | 错误类型 | 诊断方法 | 恢复策略 |
  | :--- | :--- | :--- |
  | 连接失败 | 调用 `get_config_info` 检查配置 | 提示用户检查 API URL / Key |
  | 无搜索结果 | 检查 query 是否过于具体 | 放宽搜索词，移除限定条件 |
  | 网页抓取超时 | 检查 URL 可访问性 | 尝试搜索替代来源 |
  | 内容被截断 | 检查目标页面结构 | 分段抓取或提示用户直接访问 |

  ## 4. Anti-Patterns

  | ❌ 禁止行为 | ✅ 正确做法 |
  | :--- | :--- |
  | 搜索后不标注来源 | 输出**必须**包含 `[来源](URL)` 引用 |
  | 单次搜索失败即放弃 | 调整参数后至少重试 1 次 |
  | 假设网页内容而不抓取 | 对关键信息**必须**调用 `web_fetch` 验证 |
  | 忽略搜索结果的时效性 | 时间敏感信息**必须**标注日期 |

  ---
  模块说明：
  - 强制替换：明确禁用内置工具，强制路由到 GrokSearch
  - 三工具覆盖：web_search + web_fetch + get_config_info
  - 错误处理：包含配置诊断的恢复策略
  - 引用规范：强制标注来源，符合信息可追溯性要求

---
> Source: [tom-cat-mao/myResume](https://github.com/tom-cat-mao/myResume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
