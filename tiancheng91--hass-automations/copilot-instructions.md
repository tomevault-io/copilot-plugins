## hass-automations

> 本项目是一个模块化的 Home Assistant 配置项目，通过 YAML 文件组织各种自动化场景。

# Home Assistant 自动化配置项目规则

## 项目结构

本项目是一个模块化的 Home Assistant 配置项目，通过 YAML 文件组织各种自动化场景。

### 目录结构
- `automations/` - 自动化配置，每个文件包含一个或多个自动化
- `inputs/` - input_number 配置（如模式状态、参数等）
- `templates/` - 模板传感器配置（binary_sensor, sensor 等）
- `scripts/` - 脚本配置（可复用的自动化逻辑）
- `rest_commands/` - REST 命令配置（如 Server酱推送）
- `secrets.yaml` - 敏感信息（API token 等，已加入 .gitignore）

### 配置文件引用
在 `configuration.yaml` 中需要包含：
```yaml
input_number: !include_dir_merge_named inputs/
automation: !include_dir_list automations/
script: !include_dir_list scripts/
rest_command: !include_dir_merge_named rest_commands/
template: !include_dir_merge_list templates/
```

## 命名规范

### 文件命名
- 使用小写字母和下划线：`light_kitchen.yaml`、`home_state_control.yaml`
- 文件名应反映自动化功能或区域：`light_*`（灯光）、`temp_*`（温度）、`fan_*`（风扇）、`alert_*`（提醒）

### 自动化 ID 命名
- 使用小写字母和下划线：`light_kitchen_auto_off`、`bedroom_fan_level`
- 格式：`{区域/设备}_{功能}_{动作}`，如 `light_wash_motion_detected`、`bedroom_climate_on`
- 必须唯一，建议包含区域和设备信息

### 自动化别名（alias）
- 使用中文描述：`厨房 - 玄关无人30分钟且灯开启25分钟关灯`
- 格式：`{区域} - {功能描述}`
- 应清晰描述自动化功能

## 自动化配置必需字段

每个自动化必须包含以下字段：

### 必需字段
1. **alias** (string) - 中文别名，描述自动化功能
2. **id** (string) - 唯一标识符，使用下划线命名
3. **description** (string) - 详细描述，说明自动化触发条件和执行动作
4. **trigger** (list) - 触发条件列表
5. **action** (list) - 执行动作列表

### 推荐字段
6. **mode** (string) - 执行模式：
   - `single` - 单次执行（默认），适合大多数场景
   - `restart` - 重启模式，触发时重新开始计时
   - `queued` - 队列模式，按顺序执行
   - `parallel` - 并行模式，同时执行多个实例
7. **initial_state** (boolean) - 初始状态：
   - `true` - 启动时启用（默认）
   - `false` - 启动时禁用，需要手动启用
8. **condition** (list) - 额外条件，用于进一步过滤触发

## 自动化配置模板

### 基础模板
```yaml
- alias: {区域} - {功能描述}
  id: {区域}_{功能}_{动作}
  mode: single
  description: {详细描述触发条件和执行动作}
  trigger:
    - platform: {触发平台}
      entity_id: {实体ID}
      # ... 其他触发条件
  condition:
    # 只在有人在家时触发（大多数自动化需要此条件）
    - condition: state
      entity_id: binary_sensor.home_state
      state: "on"
  action:
    - service: {服务名}
      target:
        entity_id: {实体ID}
```

### 灯光自动化模板
```yaml
- alias: {区域} - {功能描述}
  id: {区域}_light_{功能}
  mode: single  # 或 restart（需要重新计时时）
  description: {详细描述}
  trigger:
    - platform: state
      entity_id: {传感器实体ID}
      # ... 触发条件
  condition:
    - condition: state
      entity_id: binary_sensor.home_state
      state: "on"
    - condition: state
      entity_id: {灯光实体ID}
      state: 'on'  # 确保灯光当前是开启状态（如需要）
  action:
    - service: light.turn_on  # 或 switch.turn_on
      target:
        entity_id: {灯光实体ID}
      data:
        brightness_pct: {亮度百分比}
        transition: {过渡时间（秒）}
```

### 温度/环境自动化模板
```yaml
- alias: {区域} - {功能描述}
  id: {区域}_{设备}_{动作}
  initial_state: true
  mode: single
  description: {详细描述}
  trigger:
    - platform: numeric_state
      entity_id: {温度实体ID}
      attribute: current_temperature
      above: {温度阈值}
      for:
        minutes: {持续时间}
  condition:
    - condition: state
      entity_id: binary_sensor.home_state
      state: "on"
    - condition: time
      after: '{开始时间}'
      before: '{结束时间}'
  action:
    - service: {服务名}
      target:
        entity_id: {设备实体ID}
```

## 常见模式和最佳实践

### 1. 在家状态检查
大多数自动化应只在有人在家时触发：
```yaml
condition:
  - condition: state
    entity_id: binary_sensor.home_state
    state: "on"
```

### 2. 时间段判断
使用模板进行时间段判断：
```yaml
action:
  - variables:
      current_hour: "{{ now().hour }}"
      is_daytime: "{{ current_hour >= 8 and current_hour < 17 }}"
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ is_daytime }}"
        sequence:
          # 白天逻辑
      - conditions:
          - condition: template
            value_template: "{{ not is_daytime }}"
        sequence:
          # 夜间逻辑
```

### 3. 设备状态检查
在执行动作前检查设备状态：
```yaml
condition:
  - condition: state
    entity_id: {设备实体ID}
    state: 'on'  # 或 'off'
```

### 4. 渐进式亮度调整
使用延迟和多次调用来实现渐进式调整：
```yaml
action:
  - service: light.turn_on
    target:
      entity_id: {灯光实体ID}
    data:
      brightness_pct: {初始亮度}
  - delay:
      seconds: {延迟时间}
  - service: light.turn_on
    target:
      entity_id: {灯光实体ID}
    data:
      brightness_pct: {目标亮度}
```

### 5. 设备可用性等待
等待设备可用后再执行操作：
```yaml
action:
  - wait_template: "{{ states('{实体ID}') not in ['unavailable', 'unknown'] }}"
    timeout: "00:00:30"
    continue_on_timeout: true
  - delay:
      seconds: 3
  - service: {服务名}
    target:
      entity_id: {实体ID}
```

### 6. 消息推送
使用 Server酱推送消息：
```yaml
action:
  - service: script.serverchan_push
    data:
      title: "{消息标题}"
      content: "{消息内容，支持模板语法}"
```

### 7. 日志记录
记录自动化执行日志：
```yaml
action:
  - service: logbook.log
    data:
      name: {日志名称}
      message: "{日志消息}"
```

## 创建新自动化时的检查清单

当用户要求创建新自动化时，必须确认以下信息：

### 必需信息
1. **区域/位置** - 自动化作用的区域（如：厨房、卫生间、主卧）
2. **功能描述** - 自动化要实现的功能（如：自动关灯、温度控制）
3. **触发条件** - 什么情况下触发（传感器、时间、状态变化等）
4. **执行动作** - 触发后执行什么操作（开关设备、调整参数等）
5. **实体ID** - 涉及的设备/传感器实体ID

### 可选但重要的信息
6. **时间段限制** - 是否只在特定时间段生效（如：夜间、白天）
7. **在家状态** - 是否只在有人在家时触发（大多数自动化需要）
8. **执行模式** - 使用 `single`、`restart`、`queued` 还是 `parallel`
9. **初始状态** - 启动时是否启用（`initial_state: true/false`）
10. **延迟时间** - 触发后是否需要延迟执行
11. **持续时间** - 条件需要持续多久才触发（`for` 参数）

### 引导流程
1. 询问用户要创建什么类型的自动化（灯光、温度、风扇等）
2. 确认区域和功能描述
3. 询问触发条件（传感器、时间、状态等）
4. 确认涉及的实体ID（如果用户未提供，需要查找或询问）
5. 询问是否需要时间段限制
6. 确认是否需要在家状态检查（默认需要）
7. 询问执行模式和初始状态
8. 创建完整的自动化配置
9. 确认文件保存位置（`automations/` 目录下）

## 实体ID命名模式

### 传感器
- 人体传感器事件：`event.linp_cn_*_motion_detected_e_*`
- 无移动时长：`sensor.linp_cn_*_no_motion_duration_p_*`
- 温度传感器：`climate.*_current_temperature` 或 `sensor.*_temperature`

### 灯光设备
- 开关类型：`switch.*_on_p_*`
- 灯光类型：`light.*_light`
- 亮度控制：使用 `brightness_pct` 参数（0-100）

### 状态传感器
- 在家状态：`binary_sensor.home_state`
- 门状态：`binary_sensor.door_state`

## 注意事项

1. **实体ID验证** - 创建自动化前，确认实体ID存在且正确
2. **状态检查** - 大多数自动化应检查 `binary_sensor.home_state` 状态
3. **错误处理** - 使用 `not_from`/`not_to` 过滤 unavailable/unknown 状态
4. **模板语法** - 使用 Jinja2 模板语法进行复杂逻辑判断
5. **延迟和等待** - 设备响应可能需要时间，适当使用延迟和等待
6. **模式选择** - 根据需求选择合适的执行模式
7. **文件组织** - 相关自动化放在同一文件中，按功能分类
8. **注释说明** - 在文件中添加注释说明自动化逻辑

## 代码风格

1. 使用 2 空格缩进
2. 字符串使用单引号（除非包含模板语法）
3. 注释使用中文，清晰说明逻辑
4. 每个自动化之间用空行分隔
5. 复杂逻辑使用模板变量和 choose 条件分支
6. 时间格式使用 `"HH:MM:SS"` 或 `minutes: N` 格式

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tiancheng91) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
