## openclaw-hot-skills-zh

> from browser_use import Agent, ChatBrowserUse

# Agent 配置与行为

## 目录
- [基本用法](#基本用法)
- [所有参数](#所有参数)
- [输出格式](#输出格式)
- [结构化输出](#结构化输出)
- [提示指南](#提示指南)
- [生命周期钩子](#生命周期钩子)
- [超时环境变量](#超时环境变量)

---

## 基本用法

```python
from browser_use import Agent, ChatBrowserUse

agent = Agent(
    task="Search for latest news about AI",
    llm=ChatBrowserUse(),
)

async def main():
    history = await agent.run(max_steps=500)
```

- `task`：要自动化的任务
- `llm`：LLM 实例（参见 `models.md`）
- `max_steps`（默认：`500`）：代理最大步数

## 所有参数

### 核心设置
- `tools`：代理可调用的工具注册表
- `skills`（或 `skill_ids`）：要加载的技能 ID 列表（例如 `['skill-uuid']` 或 `['*']` 加载全部）。需要 `BROWSER_USE_API_KEY`
- `browser`：浏览器设置对象
- `output_model_schema`：用于结构化输出验证的 Pydantic 模型类

### 视觉与处理
- `use_vision`（默认：`True`）：`True` 始终包含截图，`"auto"` 包含截图工具但仅在请求时使用视觉，`False` 从不使用
- `vision_detail_level`（默认：`'auto'`）：`'low'`、`'high'` 或 `'auto'`
- `page_extraction_llm`：用于页面内容提取的独立 LLM（默认：与 `llm` 相同）

### 回退与容错
- `fallback_llm`：主 LLM 失败时的备用 LLM。主 LLM 会先耗尽其重试逻辑（5 次尝试，指数退避）。触发条件：429（速率限制）、401（认证）、402（付款）、500/502/503/504（服务器错误）。一旦切换，后续运行将使用备用 LLM。

### 动作与行为
- `initial_actions`：在主任务之前运行的操作，无需 LLM
- `max_actions_per_step`（默认：`5`）：每步最大动作数（例如一次填写 5 个表单字段）
- `max_failures`（默认：`5`）：出错步骤的最大重试次数
- `final_response_after_failure`（默认：`True`）：达到 max_failures 后强制进行最后一次模型调用
- `use_thinking`（默认：`True`）：启用显式推理步骤
- `flash_mode`（默认：`False`）：快速模式 — 跳过评估、下一步目标、思考；仅使用记忆。覆盖 `use_thinking`

### 系统消息
- `override_system_message`：完全替换默认系统提示
- `extend_system_message`：在默认系统提示中追加指令

### 文件与数据管理
- `save_conversation_path`：保存对话历史的路径
- `save_conversation_path_encoding`（默认：`'utf-8'`）
- `available_file_paths`：代理可访问的文件路径
- `sensitive_data`：敏感数据字典（模式参见 `examples.md`）

### 视觉输出
- `generate_gif`（默认：`False`）：生成操作 GIF。设为 `True` 或字符串路径
- `include_attributes`：在页面分析中包含的 HTML 属性

### 性能与限制
- `max_history_items`：在 LLM 记忆中保留的最大步数（`None` = 全部）
- `llm_timeout`（默认：按模型自动检测 — Groq: 30s, Gemini: 75s, Gemini 3 Pro: 90s, o3/Claude/DeepSeek: 90s, 其他: 75s）：LLM 调用的秒数
- `step_timeout`（默认：`180`）：每步的超时秒数
- `directly_open_url`（默认：`True`）：自动打开任务中检测到的 URL

### 高级
- `calculate_cost`（默认：`False`）：追踪 API 成本（通过 `history.usage` 访问）
- `display_files_in_done_text`（默认：`True`）

### 向后兼容
- `controller` → `tools` 的别名
- `browser_session` → `browser` 的别名

---

## 输出格式

`run()` 返回一个 `AgentHistoryList`：

```python
history = await agent.run()

# 基本访问
history.urls()                    # 访问过的 URL
history.screenshot_paths()        # 截图文件路径
history.screenshots()             # base64 格式的截图
history.action_names()            # 已执行的动作名称
history.extracted_content()       # 从所有动作中提取的内容
history.errors()                  # 错误（正常步骤为 None）
history.model_actions()           # 所有动作及其参数
history.model_outputs()           # 所有模型输出
history.last_action()             # 最后一个动作

# 分析
history.final_result()            # 最终提取内容（最后一步）
history.is_done()                 # 代理是否完成？
history.is_successful()           # 是否成功完成？（未完成时为 None）
history.has_errors()              # 是否有错误？
history.model_thoughts()          # 推理过程（AgentBrain 对象）
history.action_results()          # 所有 ActionResult 对象
history.action_history()          # 截断的动作历史
history.number_of_steps()         # 步数
history.total_duration_seconds()  # 总耗时

# 结构化输出
history.structured_output         # 解析后的结构化输出（如果设置了 output_model_schema）
```

## 结构化输出

使用 `output_model_schema` 配合 Pydantic 模型：

```python
from pydantic import BaseModel

class SearchResult(BaseModel):
    title: str
    url: str

agent = Agent(task="...", llm=llm, output_model_schema=SearchResult)
history = await agent.run()
result = history.structured_output  # SearchResult 实例
```

---

## 提示指南

### 要具体

```python
# 好
task = """
1. Go to https://quotes.toscrape.com/
2. Use extract action with the query "first 3 quotes with their authors"
3. Save results to quotes.csv using write_file action
"""

# 不好
task = "Go to web and make money"
```

### 直接命名动作

```python
task = """
1. Use search action to find "Python tutorials"
2. Use click to open first result in a new tab
3. Use scroll action to scroll down 2 pages
4. Use extract to extract the names of the first 5 items
"""
```

### 通过键盘处理交互问题

```python
task = """
If the submit button cannot be clicked:
1. Use send_keys action with "Tab Tab Enter"
2. Or use send_keys with "ArrowDown ArrowDown Enter"
"""
```

### 自定义动作集成

```python
@tools.action("Get 2FA code from authenticator app")
async def get_2fa_code():
    pass

task = """
Login with 2FA:
1. Enter username/password
2. When prompted for 2FA, use get_2fa_code action
3. NEVER try to extract 2FA codes from the page manually
"""
```

### 错误恢复

```python
task = """
1. Go to openai.com to find their CEO
2. If navigation fails due to anti-bot protection:
   - Use google search to find the CEO
3. If page times out, use go_back and try alternative approach
"""
```

---

## 生命周期钩子

通过 `agent.run()` 可用的两个钩子：

| 钩子 | 调用时机 |
|------|----------|
| `on_step_start` | 代理处理当前状态之前 |
| `on_step_end` | 代理执行完该步所有动作之后 |

```python
async def my_hook(agent: Agent):
    state = await agent.browser_session.get_browser_state_summary()
    print(f'Current URL: {state.url}')

await agent.run(on_step_start=my_hook, on_step_end=my_hook)
```

### 钩子中可用的数据

完整访问 Agent 实例：

- `agent.task` — 当前任务；`agent.add_new_task(...)` — 排队新任务
- `agent.tools` — Tools() 对象和注册表
  - `agent.tools.registry.execute_action('click', {'index': 123}, browser_session=agent.browser_session)`
- `agent.sensitive_data` — 敏感数据字典（可变）
- `agent.settings` — 所有配置选项
- `agent.llm` — 直接 LLM 访问
- `agent.state` — 内部状态（思考、输出、动作）
- `agent.history` — 执行历史：
  - `.model_thoughts()`、`.model_outputs()`、`.model_actions()`
  - `.extracted_content()`、`.urls()`
- `agent.browser_session` — BrowserSession + CDP：
  - `.agent_focus_target_id` — 当前目标 ID
  - `.get_or_create_cdp_session()` — CDP 会话
  - `.get_tabs()`、`.get_current_page_url()`、`.get_current_page_title()`
- `agent.pause()` / `agent.resume()` — 控制执行

### 钩子示例：CDP 访问

```python
async def my_hook(agent: Agent):
    cdp_session = await agent.browser_session.get_or_create_cdp_session()
    doc = await cdp_session.cdp_client.send.DOM.getDocument(session_id=cdp_session.session_id)
    html = await cdp_session.cdp_client.send.DOM.getOuterHTML(
        params={'nodeId': doc['root']['nodeId']}, session_id=cdp_session.session_id
    )
```

**提示：**
- 保持钩子高效（同一执行线程）
- 大多数用例更适合使用自定义工具
- 如果钩子耗时较长，请增加 `step_timeout`

---

## 超时环境变量

通过环境变量微调超时（单位为秒）：

### 浏览器动作
| 变量 | 默认值 |
|------|--------|
| `TIMEOUT_NavigateToUrlEvent` | 30.0 |
| `TIMEOUT_ClickElementEvent` | 15.0 |
| `TIMEOUT_ClickCoordinateEvent` | 15.0 |
| `TIMEOUT_TypeTextEvent` | 60.0 |
| `TIMEOUT_ScrollEvent` | 8.0 |
| `TIMEOUT_ScrollToTextEvent` | 15.0 |
| `TIMEOUT_SendKeysEvent` | 60.0 |
| `TIMEOUT_UploadFileEvent` | 30.0 |
| `TIMEOUT_GetDropdownOptionsEvent` | 15.0 |
| `TIMEOUT_SelectDropdownOptionEvent` | 8.0 |
| `TIMEOUT_GoBackEvent` | 15.0 |
| `TIMEOUT_GoForwardEvent` | 15.0 |
| `TIMEOUT_RefreshEvent` | 15.0 |
| `TIMEOUT_WaitEvent` | 60.0 |
| `TIMEOUT_ScreenshotEvent` | 15.0 |
| `TIMEOUT_BrowserStateRequestEvent` | 30.0 |

### 浏览器生命周期
| 变量 | 默认值 |
|------|--------|
| `TIMEOUT_BrowserStartEvent` | 30.0 |
| `TIMEOUT_BrowserStopEvent` | 45.0 |
| `TIMEOUT_BrowserLaunchEvent` | 30.0 |
| `TIMEOUT_BrowserKillEvent` | 30.0 |
| `TIMEOUT_BrowserConnectedEvent` | 30.0 |

### 标签页管理
| 变量 | 默认值 |
|------|--------|
| `TIMEOUT_SwitchTabEvent` | 10.0 |
| `TIMEOUT_CloseTabEvent` | 10.0 |
| `TIMEOUT_TabCreatedEvent` | 30.0 |
| `TIMEOUT_TabClosedEvent` | 10.0 |

### 存储与下载
| 变量 | 默认值 |
|------|--------|
| `TIMEOUT_SaveStorageStateEvent` | 45.0 |
| `TIMEOUT_LoadStorageStateEvent` | 45.0 |
| `TIMEOUT_FileDownloadedEvent` | 30.0 |

---
> Source: [L-LesterYu/OpenClaw-hot-skills-zh](https://github.com/L-LesterYu/OpenClaw-hot-skills-zh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
