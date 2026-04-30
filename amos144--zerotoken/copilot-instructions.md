## zerotoken

> ZeroToken 是一个面向 AI Agent 的浏览器自动化 MCP 引擎，核心目标是：

# ZeroToken 项目开发指南

## 项目概述

ZeroToken 是一个面向 AI Agent 的浏览器自动化 MCP 引擎，核心目标是：
- **降低 Token 消耗** - 通过轨迹记录与 AI 提示导出，供 Skills 等模块分析
- **详细执行上下文** - 每次操作记录完整状态、截图、模糊点
- **模糊点标记** - 显式标记需 AI/人判断的步骤

## 核心架构

### 设计思路

```
AI Agent (ReAct 模式)
       ↓
   MCP 工具调用 (原子能力)
       ↓
   返回结构化 OperationRecord (含 fuzzy_point)
       ↓
   记录完整 Trajectory
       ↓
   trajectory.to_ai_prompt_format() 导出含模糊点的 AI 提示
       ↓
   Skills 或外部模块进一步处理
```

### 模块职责

#### 1. BrowserController (`controller.py`)
- **职责**: 浏览器控制，返回详细的 OperationRecord
- **关键特性**:
  - 全局单例模式
  - 每个操作返回完整上下文（页面状态、截图、结果）
  - 自动等待机制
  - 操作历史追踪

```python
# 核心方法
- open(url) → OperationRecord
- click(selector) → OperationRecord
- input(selector, text) → OperationRecord
- get_text(selector) → OperationRecord
- extract_data(schema) → OperationRecord  # AI 节点能力
```

#### 2. TrajectoryRecorder (`trajectory.py`)
- **职责**: 记录完整的操作轨迹，导出 AI 提示
- **关键特性**:
  - 绑定 BrowserController 自动记录
  - 导出 AI 友好的提示格式（含模糊点 `[需判断: {reason}]`）
  - 轨迹持久化存储

```python
# 核心方法
- start_trajectory(task_id, goal)
- record_operation(record)
- complete_trajectory() → Trajectory
- trajectory.to_ai_prompt_format() → str  # AI 提示格式
```

## 数据结构

### OperationRecord
```python
{
    "step": int,
    "action": str,
    "params": dict,
    "result": {
        "success": bool,
        "navigated": bool,  # click 专属
        "value": any,  # extract 专属
        ...
    },
    "page_state": {
        "url": str,
        "title": str,
        "timestamp": str
    },
    "screenshot": str,  # base64
    "error": str,
    "fuzzy_point": {  # 可选，需 AI/人判断时存在
        "requires_judgment": bool,
        "reason": str,
        "hint": str
    },
    "timestamp": str
}
```

### Trajectory
```python
{
    "task_id": str,
    "goal": str,
    "start_time": str,
    "end_time": str,
    "operations": List[OperationRecord],
    "metadata": {
        "total_steps": int,
        "successful_steps": int,
        "failed_steps": int
    }
}
```

## 开发规范

### 代码风格
- 使用 async/await 异步编程
- 所有浏览器操作必须等待元素就绪
- 错误处理要返回统一的响应格式
- 操作记录必须包含完整上下文

### 命名约定
- 模块名：小写 + 下划线
- 类名：大驼峰
- 函数名：小写 + 下划线
- 常量：大写 + 下划线

### 模糊点 (fuzzy_point) 设计

需 AI/人判断的步骤标记为模糊点：

- **extract_data** 默认自动设置 `fuzzy_point`
- **其他方法** 可通过 `fuzzy_reason`、`fuzzy_hint` 传入
- 导出 AI 提示时追加 `[需判断: {reason}]`

## MCP 工具列表

### Browser Tools
- `browser_init` - 初始化浏览器；可选 **stealth**（默认 false）：为 true 时启用反检测，包含：启动参数（--disable-blink-features=AutomationControlled 等）、navigator 指纹伪装（webdriver、plugins、languages、platform、hardwareConcurrency、deviceMemory、maxTouchPoints）、Sec-CH-UA 等 HTTP 头、WebGL 指纹伪装（UNMASKED_VENDOR/RENDERER），适合易被云盾/反爬拦截的场景（如 B 站、小红书）
- `browser_close` - 关闭浏览器
- `browser_open` / `browser_click` / `browser_input` / `browser_get_text` / `browser_get_html` / `browser_wait_for` / `browser_extract_data` - 原子操作，返回 OperationRecord
- 上述返回 record 的工具支持可选参数 **include_screenshot**（默认 true）；设为 false 时响应中不包含截图，可减少 token
- **自适应元素定位**：`browser_click`、`browser_input`、`browser_get_text`、`browser_get_html` 支持可选参数 **auto_save**（默认 false）、**adaptive**（默认 false）、**identifier**（可选）。首次选择器命中时传 `auto_save: true` 可保存元素指纹；改版后选择器失效时传 `adaptive: true` 可按指纹相似度自动重定位，无需改代码。
- `browser_screenshot` - 截图

### Trajectory Tools
- `trajectory_start` - 开始轨迹记录
- `trajectory_complete` - 完成轨迹并导出（含 fuzzy_point 的 AI 提示）
- `trajectory_get` - 获取当前未结束轨迹（format: json | ai_prompt）
- `trajectory_list` - 列出已保存轨迹（可选 limit、since），返回 id、task_id、goal、created_at
- `trajectory_load` - 按 task_id 加载已保存轨迹（format: ai_prompt | json），供 Skill 生成脚本等使用
- `trajectory_delete` - 按 task_id 删除已保存轨迹，防止记录过多；建议先 `trajectory_list` 再对不需要的 task_id 调用

## 使用示例

### AI Agent 集成流程

```python
# 1. 初始化
await mcp_call("browser_init", {"headless": True})

# 2. 开始记录
await mcp_call("trajectory_start", {
    "task_id": "login_demo",
    "goal": "登录到系统"
})

# 3. AI 分步推理执行
await mcp_call("browser_open", {"url": "https://example.com/login"})
await mcp_call("browser_input", {"selector": "#user", "text": "test"})
await mcp_call("browser_input", {"selector": "#pass", "text": "secret"})
await mcp_call("browser_click", {"selector": "#submit"})

# 4. 完成记录获取 AI 提示（含模糊点标记）
result = await mcp_call("trajectory_complete", {"export_for_ai": True})
ai_prompt = result["ai_prompt"]

# 事后按需获取已保存轨迹（供 Skill 等使用）
list_result = await mcp_call("trajectory_list", {"limit": 20})
# list_result["trajectories"]: [{ id, task_id, goal, created_at }, ...]
load_result = await mcp_call("trajectory_load", {"task_id": "login_demo", "format": "ai_prompt"})
# load_result["ai_prompt"] 或 load_result["trajectory"]（format=json 时）
```

## 错误响应格式

工具失败时统一返回 JSON：`success: false`、`error`（必填）、可选 `code`（机器可读，如 INVALID_PARAMS、TRAJECTORY_NOT_FOUND、NO_ACTIVE_TRAJECTORY、TIMEOUT、UNKNOWN_TOOL）、可选 `retryable`（bool，是否建议重试）。模型可根据 `code` 与 `retryable` 决定是否重试或提示用户。

## 注意事项

- 不要使用 emoji（编码问题）
- 所有回答使用中文
- BrowserController 必须保持单例
- 截图数据较大时，可传 `include_screenshot: false` 减少响应体积；轨迹仍会完整记录
- 建议通过 `trajectory_list` 查看已保存轨迹，对不需要的 `task_id` 调用 `trajectory_delete`，避免本地记录过多
- 自适应定位：对关键元素首次操作时传 `auto_save: true`，网站改版后同一选择器失效时传 `adaptive: true` 可自动按指纹重定位；指纹存于主库 `zerotoken.db` 的 fingerprints 表

## 扩展方向

1. **智能选择器** - 自动选择最稳定的选择器
2. **视觉定位** - 基于截图的视觉元素定位
3. **自愈脚本** - 检测 UI 变化自动修复脚本
4. **并行执行** - 多浏览器实例并行执行任务
5. **结果分析** - 执行结果统计和报告生成

## 稳定性增强模块

### 1. SmartSelector (`selector.py`)

智能选择器生成器，为元素生成多个备选选择器。

```python
from zerotoken import SmartSelectorGenerator

generator = SmartSelectorGenerator()
element = await page.wait_for_selector("#target")
smart_selector = await generator.generate(element)

# 获取最佳选择器
best = smart_selector.best_selector()
print(f"类型：{best.type.value}")
print(f"值：{best.value}")
print(f"稳定性评分：{best.stability_score}")

# 获取所有选择器（按稳定性排序）
for selector in smart_selector.all_selectors():
    print(f"  {selector.type.value}: {selector.value}")
```

**选择器优先级**:
1. `data-testid` (评分 0.95)
2. `id` (评分 0.90)
3. `aria-label` 等 ARIA 属性 (评分 0.85)
4. `role` 选择器 (评分 0.80)
5. 稳定 CSS 选择器 (评分 0.70)
6. 文本选择器 (评分 0.65)
7. XPath (评分 0.50)

**不稳定模式检测**:
- `el-*` (Element UI)
- `ant-*` (Ant Design)
- `Mui-*` (Material UI)
- `css-*` (CSS Modules)
- `sc-*` (Styled Components)

### 2. SmartWait (`wait_strategy.py`)

智能等待策略，解决时序问题。

```python
from zerotoken import SmartWait, WaitCondition, WaitChain

smart_wait = SmartWait(page)

# 等待元素可见
result = await smart_wait.wait_for(
    WaitCondition.VISIBLE,
    "#button",
    description="等待按钮显示"
)

# 级联等待
result = await (
    WaitChain(page)
    .wait_for_selector("#button")
    .wait_for_visible()
    .wait_for_network_idle()
    .execute()
)
```

**等待条件类型**:
- `SELECTOR` - 等待元素出现
- `VISIBLE` - 等待元素可见
- `HIDDEN` - 等待元素隐藏
- `NAVIGATION` - 等待导航完成
- `NETWORK_IDLE` - 等待网络空闲
- `LOAD_STATE` - 等待加载状态
- `TEXT` - 等待文本出现
- `FUNCTION` - 等待函数返回 true

### 3. ErrorRecovery (`recovery.py`)

错误恢复机制，自动处理常见错误。

```python
from zerotoken import ErrorRecovery, RetryWrapper

recovery = ErrorRecovery(page, controller)

# 错误恢复
try:
    await page.click("#button")
except Exception as e:
    result = await recovery.handle_error(
        e,
        selector="#button",
        action="click"
    )
    if result.recovered:
        print(f"已恢复：{result.action_taken}")

# 重试包装器
retry = RetryWrapper(max_retries=3, base_delay=1.0)

async def flaky_operation():
    # 可能失败的操作
    pass

result = await retry.execute(flaky_operation, description="执行操作")
```

**错误类型检测**:
- `SELECTOR_NOT_FOUND` - 选择器未找到
- `ELEMENT_NOT_VISIBLE` - 元素不可见
- `ELEMENT_NOT_INTERCEPTABLE` - 元素不可点击
- `NAVIGATION_TIMEOUT` - 导航超时
- `NETWORK_ERROR` - 网络错误
- `POPUP_BLOCKED` - 弹窗被阻止

**恢复策略**:
- 选择器变体尝试
- iframe 内查找
- 滚动到元素
- JavaScript 点击
- 指数退避重试

### 4. BrowserController 集成

```python
controller = BrowserController()

# 启用稳定性增强
controller.set_config(
    enable_stability=True,
    max_retries=3,
    retry_delay=1.0,
    timeout=30000
)

# 所有操作自动使用稳定性增强
await controller.click("#button")  # 自动重试 + 错误恢复
```

## 测试建议

```python
# 单元测试重点
- BrowserController 各方法返回正确的 OperationRecord（含 fuzzy_point）
- TrajectoryRecorder 正确记录和导出轨迹
- trajectory.to_ai_prompt_format() 含模糊点标记

# 集成测试
- 完整流程：记录 → 导出 AI 提示
- MCP 工具调用和响应格式
```
- 严格按照superpower的流程，严格执行TDD

---
> Source: [AMOS144/ZeroToken](https://github.com/AMOS144/ZeroToken) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
