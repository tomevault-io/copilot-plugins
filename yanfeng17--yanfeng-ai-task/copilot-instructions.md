## yanfeng-ai-task

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

本文件为 Claude Code (claude.ai/code) 提供项目指导。

## 项目概述

Yanfeng AI Task 是基于 ModelScope API 的 Home Assistant AI 集成，采用 **Subentry-Only 架构**，现已整合**三层意图识别能力**，实现快速设备控制和智能对话的完美结合。

### 核心特性
- 🤖 **对话代理** - 支持中文和多语言自然语言对话
- 📝 **AI 任务生成** - 生成文本、结构化 JSON 数据
- 🖼️ **图像生成/识别** - 使用 ModelScope 图像模型
- ⚡ **三层意图识别** - 智能设备控制 + AI 对话
- 🏗️ **Subentry-Only 架构** - 完全基于子配置项的模块化设计

---

## 🏗️ 架构特点：Subentry-Only 设计

### 与传统架构的区别

**传统架构（Main + Subentry）：**
```python
# 主配置项和子配置项都能创建实体
async def async_setup_entry(hass, entry):
    # 为主配置项创建实体
    async_add_entities([MainEntity(entry)])

    # 为子配置项创建实体
    for subentry in entry.subentries.values():
        async_add_entities([SubEntity(entry, subentry)])
```

**本项目架构（Subentry-Only）：**
```python
# 只为子配置项创建实体，主配置项仅用于初始化
async def async_setup_entry(hass, entry, async_add_entities):
    # 只遍历 subentries，不创建主配置项实体
    for subentry in entry.subentries.values():
        if subentry.subentry_type != "conversation":
            continue
        async_add_entities(
            [YanfengAIConversationEntity(entry, subentry)],
            config_subentry_id=subentry.subentry_id,
        )
```

### 设计优势
- ✅ **更模块化** - 每个功能都是独立的子配置项
- ✅ **更灵活** - 用户可以创建多个不同配置的代理
- ✅ **更清晰** - 配置层次结构明确
- ✅ **更易维护** - 避免主/子配置项混合的复杂性

### 实体初始化模式

```python
class YanfengAIConversationEntity(ConversationEntity, YanfengAILLMBaseEntity):
    def __init__(self, entry: ConfigEntry, subentry: ConfigSubentry) -> None:
        # 注意：subentry 是必需参数（不是 Optional）
        super().__init__(entry, subentry)

        # 所有配置都从 subentry.data 读取
        options = self.subentry.data
        llm_api_enabled = self.subentry.data.get(CONF_LLM_HASS_API, False)

        # unique_id 使用 subentry_id
        self._attr_unique_id = subentry.subentry_id
```

---

## 🎯 三层处理机制

借鉴智谱AI集成的设计理念，结合本项目的 Subentry-Only 架构：

### 第一层：意图识别层（Intent Recognition）
- **响应速度**：50-200ms
- **处理方式**：正则表达式 + Intent Handler
- **适用场景**：简单直接的设备控制命令
- **实现文件**：`intents.py` + `intents.yaml`
- **注册位置**：`__init__.py` 中全局注册，所有 subentry 共享

**支持的控制命令：**
```
✅ "打开卧室空调"
✅ "把空调调到26度"
✅ "空调设置制冷模式"
✅ "调高空调风速"
✅ "关闭所有窗帘"
✅ "打开客厅灯"
✅ "通知：明天开会"
```

### 第二层：AI意图理解层
- **响应速度**：500-1500ms
- **处理方式**：AI 解析 + 工具调用
- **适用场景**：复杂的多步骤任务
- **实现文件**：`conversation.py` 中的 AI 处理

### 第三层：AI对话层
- **响应速度**：1-3秒
- **处理方式**：完整的 LLM 对话
- **适用场景**：开放性问答、知识咨询
- **实现文件**：`entity.py` 中的 `_async_handle_chat_log`

---

## 📁 关键文件说明

### 核心模块

#### `__init__.py`
- 集成入口，负责初始化和全局服务注册
- **关键改动**：
  - 在 `async_setup_entry` 中全局注册意图处理器
  - 意图处理器只注册一次，被所有 subentry 共享
  - 没有 `add_update_listener`（与传统架构不同）

```python
async def async_setup_entry(hass: HomeAssistant, entry: YanfengAIConfigEntry) -> bool:
    # 创建 session 并测试连接
    session = aiohttp.ClientSession(...)
    entry.runtime_data = session

    # 全局注册意图处理器（Layer 1 of three-layer processing）
    LOGGER.info("注册意图处理器...")
    await async_setup_intents(hass)
    LOGGER.info("意图处理器注册完成")

    # 设置平台（只为 subentries 创建实体）
    await hass.config_entries.async_forward_entry_setups(entry, PLATFORMS)

    return True
```

#### `conversation.py`
- 对话实体，处理所有用户输入
- **Subentry-Only 适配**：
  - `__init__` 中 `subentry` 参数是必需的（不是 Optional）
  - 直接使用 `self.subentry.data`，无需 None 检查

**三层处理逻辑**：
```python
async def _async_handle_message(self, user_input, chat_log):
    # Layer 1: Intent Recognition
    intent_result = await self._try_intent_recognition(user_input)
    if intent_result:
        LOGGER.info("✅ 第一层成功: 意图识别匹配 - %s", intent_result.intent.intent_type)
        response_text = intent_result.speech.get("plain", {}).get("speech", "")
        if response_text:
            # 添加到对话日志
            assistant_content = conversation.AssistantContent(
                agent_id=self.entry.entry_id,
                content=response_text
            )
            chat_log.content.append(assistant_content)
            return conversation.async_get_result_from_chat_log(user_input, chat_log)

    LOGGER.debug("⚠️ 第一层未匹配，转到第二/三层: AI处理")

    # Layer 2 & 3: AI Processing
    # 从 subentry.data 获取配置（不是 entry.options）
    options = self.subentry.data

    await chat_log.async_provide_llm_data(
        user_input.as_llm_context(DOMAIN),
        options.get(CONF_LLM_HASS_API),
        options.get(CONF_PROMPT),
        user_input.extra_system_prompt,
    )

    await self._async_handle_chat_log(chat_log)
    return conversation.async_get_result_from_chat_log(user_input, chat_log)
```

#### `intents.py`
- 定义所有意图处理器（Intent Handlers）
- **主要意图**：
  - `ClimateSetTemperatureIntent` - 空调温度控制（带智能模式判断）
  - `ClimateSetModeIntent` - 空调模式设置
  - `ClimateSetFanModeIntent` - 风速控制
  - `CoverControlAllIntent` - 批量窗帘控制
  - `HassLightSetAllIntent` - 灯光控制
  - `HassNotifyIntent` - 通知创建

**设计模式**：
```python
class BaseIntent(intent.IntentHandler):
    """Base intent handler."""

    def __init__(self, hass: HomeAssistant) -> None:
        self.hass = hass

class ClimateSetTemperatureIntent(BaseIntent):
    """Handle climate set temperature intent."""

    intent_type = INTENT_CLIMATE_SET_TEMP
    slot_schema = {
        vol.Required("name"): str,
        vol.Required("temperature"): vol.Any(str, int, float)
    }

    async def async_handle(self, intent_obj: intent.Intent) -> intent.IntentResponse:
        # 提取参数
        name = intent_obj.slots["name"]["value"]
        temperature = float(intent_obj.slots["temperature"]["value"])

        # 查找设备
        entity = find_climate_entity(self.hass, name)

        # 智能判断制冷/制热模式
        current_temp = state.attributes.get('current_temperature')
        if current_temp > temperature:
            mode = "cool"  # 当前温度高，需要制冷
        elif current_temp < temperature:
            mode = "heat"  # 当前温度低，需要制热

        # 调用服务
        await self.hass.services.async_call(
            "climate",
            "set_temperature",
            {"entity_id": entity.entity_id, "temperature": temperature},
        )

        return intent_obj.create_response()
```

#### `intents.yaml`
- 意图配置文件，定义句式模板和扩展规则
- **扩展规则**：
  - `request_word` - 请求词（请、帮我、麻烦等）
  - `action_word` - 动作词（打开、设置、调节等）
  - `mode` - 空调模式（制冷、制热、自动等）

```yaml
language: "zh"
intents:
  ClimateSetTemperature:
    data:
      - sentences:
          - "{request_word} {action_word} {name} {temperature}度"
          - "{request_word} {name} {action_word} {temperature}度"
    speech:
      text: "正在设置{name}温度为{temperature}度"

expansion_rules:
  request_word:
    values:
      - "请"
      - "帮我"
      - "麻烦"
      - ""  # 可选
```

#### `entity.py`
- 基础实体类，封装 ModelScope API 调用
- **Subentry-Only 适配**：
  - `YanfengAILLMBaseEntity` 的 `__init__` 需要 `subentry: ConfigSubentry`（必需）
  - 使用 `subentry.subentry_id` 作为 `unique_id`

```python
class YanfengAILLMBaseEntity(Entity):
    def __init__(self, entry: ConfigEntry, subentry: ConfigSubentry) -> None:
        # subentry 是必需参数
        self.entry = entry
        self.subentry = subentry

        # 使用 subentry_id 作为 unique_id
        self._attr_unique_id = subentry.subentry_id
        self._attr_name = subentry.data.get("name") or "AI Assistant"

    async def _async_handle_chat_log(self, chat_log):
        # 从 subentry.data 读取模型配置
        model = self.subentry.data.get("model", "Qwen/Qwen2.5-72B-Instruct")
        # 调用 ModelScope API
        # ...
```

#### `const.py`
- 常量定义，包括模型列表、默认值等

#### `ai_task.py`
- AI Task 平台实现，处理数据生成和图像生成任务
- **Subentry-Only 适配**：
  - 只为 `subentry_type == "ai_task_data"` 的子配置项创建实体
  - 继承 `YanfengAILLMBaseEntity` 复用 API 调用逻辑

```python
async def async_setup_entry(hass, config_entry, async_add_entities):
    # 只为 ai_task_data 子配置项创建实体
    for subentry in config_entry.subentries.values():
        if subentry.subentry_type != "ai_task_data":
            continue
        async_add_entities(
            [YanfengAITaskEntity(hass, config_entry, subentry)],
            config_subentry_id=subentry.subentry_id,
        )
```

**支持的功能**：
- `GENERATE_DATA` - 生成文本或结构化 JSON 数据
- `GENERATE_IMAGE` - 生成图像（使用 ModelScope 图像模型）
- `SUPPORT_ATTACHMENTS` - 支持图像输入（视觉模型）

#### `helpers.py`
- `ModelScopeAPIClient` - 封装 ModelScope API 调用
- 提供统一的文本生成、图像生成接口
- 处理 API 错误和重试逻辑

#### `config_flow.py`
- 配置流程，包括主配置项和子配置项的配置
- **重要**：版本号为 `VERSION = 2, MINOR_VERSION = 1`
- 支持两种子配置项类型：
  - `conversation` - 对话代理
  - `ai_task_data` - AI Task 数据生成

---

## 🔧 开发命令

### 项目结构

```
custom_components/yanfeng_ai_task/
├── __init__.py           # 集成入口，注册意图处理器
├── manifest.json         # 集成元数据，版本 2.0.0
├── config_flow.py        # 配置流程（主配置项 + 子配置项）
├── const.py              # 常量定义
├── entity.py             # 基础实体类（YanfengAILLMBaseEntity）
├── conversation.py       # 对话平台（三层处理逻辑）
├── ai_task.py            # AI Task 平台
├── intents.py            # 意图处理器（Layer 1）
├── intents.yaml          # 意图配置（句式模板）
├── helpers.py            # API 客户端
├── services.yaml         # 服务定义
├── strings.json          # 配置界面文本
├── translations/         # 多语言翻译
│   ├── en.json
│   └── zh-Hans.json
└── icons.json            # 自定义图标
```

### 安装测试

```bash
# 1. 复制到 Home Assistant 配置目录
# Windows (推荐使用符号链接便于开发)
mklink /D "C:\path\to\homeassistant\config\custom_components\yanfeng_ai_task" "C:\AI Coding\000\yanfeng_ai_task-main\custom_components\yanfeng_ai_task"

# Linux/macOS
ln -s /path/to/repo/custom_components/yanfeng_ai_task /path/to/homeassistant/config/custom_components/yanfeng_ai_task

# 或直接复制（每次修改后需重新复制）
cp -r custom_components/yanfeng_ai_task /path/to/homeassistant/config/custom_components/

# 2. 重启 Home Assistant
# 修改 Python 代码后必须重启才能生效
# 修改 YAML/JSON 文件后通常也需要重启
```

### 本地开发工作流

1. **修改代码** - 在项目目录中编辑文件
2. **重启 HA** - 重启 Home Assistant 加载新代码
3. **查看日志** - 检查日志中的错误和调试信息
4. **测试功能** - 在 HA 界面中测试对话、设备控制等

**提示**：
- 使用符号链接（symlink）可以避免每次都复制文件
- YAML 语法错误会导致集成加载失败，务必验证格式
- 意图处理器的正则表达式需要在 `conversation.py` 和 `intents.yaml` 中同步

### 验证配置文件

```bash
# 检查 YAML 语法
python -c "import yaml; yaml.safe_load(open('custom_components/yanfeng_ai_task/intents.yaml'))"

# 检查 JSON 语法
python -c "import json; json.load(open('custom_components/yanfeng_ai_task/manifest.json'))"
```

### 调试日志配置

```yaml
logger:
  default: info
  logs:
    custom_components.yanfeng_ai_task: debug
    custom_components.yanfeng_ai_task.intents: debug
    custom_components.yanfeng_ai_task.conversation: debug
    homeassistant.components.conversation: debug
```

### 查看日志

```bash
tail -f /config/home-assistant.log | grep yanfeng
```

### 测试意图识别

在 HA 的对话界面输入：
- "打开卧室空调"
- "把空调调到26度"

查看日志中的：
- "✅ 第一层成功：意图识别匹配" - Layer 1 工作正常
- "⚠️ 第一层未匹配，转到第二/三层" - 转到 AI 处理

---

## 💡 如何添加新的意图

### 步骤 1：定义 Intent Handler

在 `intents.py` 中创建新的处理器类：

```python
# 1. 定义意图类型常量
INTENT_MY_NEW = "MyNewIntent"

# 2. 创建处理器类
class MyNewIntent(BaseIntent):
    """Handle my new intent."""

    intent_type = INTENT_MY_NEW
    slot_schema = {
        vol.Required("param1"): str,
        vol.Optional("param2"): str
    }

    async def async_handle(self, intent_obj: intent.Intent) -> intent.IntentResponse:
        # 提取参数
        param1 = intent_obj.slots["param1"]["value"]
        param2 = intent_obj.slots.get("param2", {}).get("value")

        # 实现你的逻辑
        # ...

        # 返回响应
        response = intent_obj.create_response()
        response.response_type = intent.IntentResponseType.ACTION_DONE
        response.speech = {
            "plain": {"speech": f"正在处理 {param1}"}
        }
        return response
```

### 步骤 2：注册 Intent

在 `intents.py` 的 `async_setup_intents()` 中添加：

```python
async def async_setup_intents(hass: HomeAssistant) -> None:
    """Register intent handlers."""
    # ... 其他意图 ...

    intent.async_register(hass, MyNewIntent(hass))
    LOGGER.info("✅ 注册意图: MyNewIntent")
```

### 步骤 3：配置句式模板

在 `intents.yaml` 中添加：

```yaml
MyNewIntent:
  data:
    - sentences:
        - "{request_word} {action_word} {param1}"
        - "{param1} {action_word}"
  speech:
    text: "正在处理{param1}"
```

### 步骤 4：添加模式匹配

在 `conversation.py` 的 `_try_intent_recognition()` 中添加：

```python
intent_patterns = {
    # ... 其他模式 ...

    "MyNewIntent": [
        r"(?:请|帮我)?做某事(.+)",
        r"处理(.+)",
    ],
}
```

并在 `_extract_slots_from_match()` 中添加参数提取逻辑：

```python
def _extract_slots_from_match(self, intent_type: str, match: re.Match) -> dict:
    slots = {}

    if intent_type == "MyNewIntent":
        if len(match.groups()) >= 1:
            slots["param1"] = {"value": match.group(1).strip()}

    # ... 其他意图 ...

    return slots
```

---

## 🎨 中文支持优化

### 控制词扩展

项目支持丰富的中文表达：

**请求词：**
```
请、帮我、请帮我、麻烦、能否、可以、希望、我想、让
```

**动作词：**
```
打开、开启、启动、关闭、调节、设置、调整
```

**设备模式：**
- 空调模式：制冷、制热、自动、除湿、送风
- 风速：低速、中速、高速、一档、二档、三档
- 摆风：开启、关闭、水平、垂直

### 智能判断

**空调温度自动判断制冷/制热：**
```python
current_temp = state.attributes.get('current_temperature')
if current_temp > temperature:
    # 当前28度，设置26度 → 自动制冷
    mode = "cool"
elif current_temp < temperature:
    # 当前24度，设置26度 → 自动制热
    mode = "heat"
```

---

## 📊 性能优化

### 选择合适的模型
- **快速响应**: Qwen/Qwen2.5-7B-Instruct
- **平衡**: Qwen/Qwen2.5-32B-Instruct
- **最佳质量**: Qwen/Qwen2.5-72B-Instruct（推荐）
- **视觉任务**: Qwen/Qwen3-VL-235B-A22B-Instruct

### 调整参数
```yaml
temperature: 0.7   # 标准
top_p: 0.9         # 标准
max_tokens: 2048   # 根据需求调整
```

---

## 🔍 故障排除

### 问题1：意图识别不工作
**症状**：设备控制命令没有反应

**解决方案**：
1. 查看日志，确认意图是否注册
   ```bash
   grep "注册意图" /config/home-assistant.log
   ```
2. 检查 `intents.yaml` 语法（YAML 格式是否正确）
3. 确认正则表达式是否匹配用户输入
4. 验证 `__init__.py` 中 `async_setup_intents` 是否被调用

### 问题2：响应速度慢
**症状**：所有命令都需要1-3秒

**解决方案**：
1. 检查第一层是否正常工作
2. 查看日志中的 "✅ 第一层成功" 信息
3. 如果都是 "⚠️ 第一层未匹配"，优化正则表达式：
   - 增加更多匹配模式
   - 简化复杂的表达式
   - 检查中文字符是否正确匹配

### 问题3：设备找不到
**症状**：返回 "找不到XX设备"

**解决方案**：
1. 检查设备的 `friendly_name` 和 `entity_id`：
   ```python
   # 在开发者工具 - 状态 中查看设备名称
   ```
2. 在 `intents.py` 的 `find_climate_entity` 中调整匹配逻辑：
   ```python
   def find_climate_entity(hass, name):
       # 添加更多匹配方式
       for state in hass.states.async_all("climate"):
           if name in state.attributes.get("friendly_name", ""):
               return state
           # 也可以尝试匹配 entity_id 的一部分
           if name in state.entity_id:
               return state
   ```

### 问题4：Subentry 配置问题
**症状**：实体无法创建或配置读取失败

**解决方案**：
1. 确认配置项类型：
   ```python
   # 检查 subentry_type 是否为 "conversation" 或 "ai_task_data"
   if subentry.subentry_type not in ["conversation", "ai_task_data"]:
       continue
   ```
2. 验证配置数据结构：
   ```python
   # subentry.data 应该包含所需的配置
   options = self.subentry.data
   LOGGER.debug("Subentry data: %s", options)
   ```
3. 检查是否误用了 `entry.options` 而不是 `subentry.data`

### 问题5：意图处理器重复注册
**症状**：日志显示 "Intent already registered"

**解决方案**：
- 意图处理器在 `__init__.py` 中全局注册，只会注册一次
- 如果多次加载集成，需要先检查是否已注册：
  ```python
  # 在 intents.py 中添加检查
  if not intent.async_is_registered(hass, INTENT_CLIMATE_SET_TEMP):
      intent.async_register(hass, ClimateSetTemperatureIntent(hass))
  ```

### 问题6：配置流程版本升级
**症状**：修改配置流程后用户数据丢失

**解决方案**：
- 当前版本：`VERSION = 2, MINOR_VERSION = 1`
- 如需修改配置结构，需实现迁移函数：
  ```python
  @staticmethod
  async def async_migrate_entry(hass, config_entry):
      # 从旧版本迁移数据
      if config_entry.version == 1:
          # 迁移逻辑
          config_entry.version = 2
      return True
  ```

### 启用调试日志

```yaml
logger:
  default: info
  logs:
    custom_components.yanfeng_ai_task: debug
    custom_components.yanfeng_ai_task.intents: debug
    custom_components.yanfeng_ai_task.conversation: debug
```

重启后查看日志：
```bash
tail -f /config/home-assistant.log | grep yanfeng
```

---

## 🔑 关键设计要点

### Subentry-Only 架构的注意事项

1. **实体初始化**：所有实体类的 `__init__` 必须接受 `subentry: ConfigSubentry` 参数（不是 Optional）
2. **配置读取**：始终使用 `self.subentry.data`，不要使用 `entry.options`
3. **实体 ID**：使用 `subentry.subentry_id` 作为 `unique_id`
4. **平台设置**：在 `async_setup_entry` 中遍历 `entry.subentries.values()`，根据 `subentry_type` 筛选

### 三层处理机制的协作

1. **Layer 1 (Intent Recognition)** 在 `conversation.py` 的 `_try_intent_recognition()` 中执行
2. **成功匹配后**：直接返回 `IntentResponse`，不再调用 AI
3. **未匹配时**：透明地转到 Layer 2/3，用户无感知
4. **意图注册**：全局注册一次，所有 subentry 共享

### ModelScope API 使用

1. **API Base URL**：`https://api-inference.modelscope.cn/`
2. **认证**：`Authorization: Bearer {api_key}`
3. **模型选择**：
   - 对话：`Qwen/Qwen2.5-72B-Instruct`（推荐）
   - 视觉：`Qwen/Qwen3-VL-235B-A22B-Instruct`
   - 图像生成：`Qwen/Qwen-Image`
4. **超时设置**：30 秒（`TIMEOUT_SECONDS = 30`）

### 常见陷阱

❌ **错误**：在主配置项的 `async_setup_entry` 中创建实体
```python
# 不要这样做
async def async_setup_entry(hass, entry, async_add_entities):
    async_add_entities([MainEntity(entry)])  # ❌
```

✅ **正确**：只为子配置项创建实体
```python
async def async_setup_entry(hass, entry, async_add_entities):
    for subentry in entry.subentries.values():
        if subentry.subentry_type == "conversation":
            async_add_entities(
                [ConversationEntity(entry, subentry)],
                config_subentry_id=subentry.subentry_id,
            )
```

---

## 🆕 版本更新

### v2.0.0 (2025-01-XX)
- ✨ **新增**：三层意图识别机制
- ✨ **新增**：设备控制意图处理器
- ✨ **新增**：空调智能模式判断
- ✨ **新增**：批量窗帘控制
- ✨ **优化**：中文表达支持
- ✨ **优化**：响应速度（50-200ms for Layer 1）
- 📦 **依赖**：添加 pyyaml>=6.0

### v1.0.7 及更早版本
- 基础对话代理功能
- AI Task 数据生成
- 图像生成/识别
- Subentry-Only 架构设计
- aiofiles 异步文件操作支持

---

## 📚 相关资源

- [Home Assistant 官方文档](https://www.home-assistant.io/)
- [ModelScope 平台](https://modelscope.cn/)
- [Qwen 模型文档](https://github.com/QwenLM)
- [智谱AI集成参考](https://github.com/knoop7/zhipuai) - 三层架构设计灵感来源
- [Home Assistant Intent 开发文档](https://developers.home-assistant.io/docs/intent_builtin/)

---

## 🙏 致谢

- [Home Assistant](https://www.home-assistant.io/) - 智能家居平台
- [ModelScope](https://modelscope.cn/) - AI 模型服务
- [Qwen Team](https://github.com/QwenLM) - 强大的语言模型
- [智谱AI集成](https://github.com/knoop7/zhipuai) - 三层架构设计灵感

---

Made with ❤️ by Yanfeng | Powered by ModelScope | Enhanced with Three-Layer Intent Recognition | Subentry-Only Architecture

---
> Source: [yanfeng17/yanfeng_ai_task](https://github.com/yanfeng17/yanfeng_ai_task) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
