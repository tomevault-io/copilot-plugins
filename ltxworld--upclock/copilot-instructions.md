## upclock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

upClock 是一款 macOS 久坐提醒工具，常驻状态栏运行，通过键鼠活动监测、窗口追踪和可选的摄像头姿态检测来评估用户状态，并在连续坐姿超过阈值时发送通知。

**核心特性**：
- 状态栏常驻 UI（基于 rumps）
- FastAPI 后端提供实时仪表盘（WebSocket + REST）
- 可选的摄像头姿态检测（MediaPipe/ONNX Runtime）
- 心流模式、延后提醒、静默时段等灵活控制
- 所有数据保存在内存中，关闭应用后重新统计

## 开发命令

### 环境设置

```bash
# 安装基础依赖（不含视觉模块）
uv sync --dev

# 安装完整版（含摄像头姿态检测）
uv sync --dev --extra vision
```

### 运行与测试

```bash
# 运行应用（会启动状态栏和 API 服务）
uv run python main.py

# 运行测试
uv run pytest

# 运行单个测试文件
uv run pytest tests/test_activity_engine.py

# 运行单个测试函数
uv run pytest tests/test_activity_engine.py::test_function_name -v
```

### 打包 macOS 应用

```bash
# 完整版（含视觉依赖，约 300MB）
uv sync --extra macos --extra vision
bash scripts/build_macos_app.sh
# 输出: dist/upClock.app

# 轻量版（仅键鼠监控，约 50MB）
uv sync --extra macos
bash scripts/build_macos_app_light.sh
# 输出: dist/upClock-light.app
```

**注意**：打包脚本使用不同的虚拟环境（`UV_PROJECT_ENVIRONMENT`），避免测试和打包依赖冲突。

## 架构概览

### 模块分层

```
src/upclock/
├── core/                          # 核心引擎
│   ├── activity_engine.py         # 活动评分引擎，状态机（ACTIVE/SHORT_BREAK/PROLONGED_SEATED）
│   └── signal_buffer.py           # 时间序列信号缓冲区
├── adapters/                      # 平台适配器
│   ├── macos/                     # macOS 键鼠监听和窗口追踪（Quartz/pyobjc）
│   └── vision/                    # 摄像头姿态检测（MediaPipe/ONNX/模拟数据）
├── notifiers/                     # 通知发送器（macOS NSUserNotificationCenter）
├── ui/                            # 用户界面
│   ├── status/                    # rumps 状态栏应用
│   ├── static/                    # 仪表盘前端（HTML/CSS/JS + Chart.js）
│   └── server.py                  # FastAPI 服务（REST + WebSocket）
├── config.py                      # Pydantic 配置模型
├── config_store.py                # 用户设置持久化（~/.upclock/config.json）
└── service.py                     # 后台线程协调器（SharedState 桥接）
```

### 关键数据流

```
macOS 事件 (键鼠/窗口)
        ↓
  SignalBuffer (滚动聚合)
        ↓
  ActivityEngine (评分 + 状态判断)
        ↓
   SharedState (线程间通信)
        ↓
    ┌──────┴───────┐
    ↓              ↓
状态栏 UI      FastAPI 仪表盘
(rumps)        (WebSocket)
```

### 双虚拟环境策略

项目针对开发/测试和打包使用不同的虚拟环境：

- **开发环境**：默认 `.venv`（包含 pytest、所有 extra 依赖）
- **打包环境**：`.venv-full` 或 `.venv-light`（通过 `UV_PROJECT_ENVIRONMENT` 切换，仅保留运行时依赖）

这避免了测试框架被打包进 macOS App，减小体积。

## 配置系统

### AppConfig（应用启动配置）

定义在 `src/upclock/config.py`，可通过 `config.local.py` 覆盖默认值：

```python
# config.local.py 示例
from upclock.config import AppConfig

def load_config() -> AppConfig:
    return AppConfig(
        prolonged_seated_minutes=50,        # 久坐阈值
        vision_enabled=True,                # 启用摄像头
        vision_pose_backend="onnx",         # 使用 ONNX 姿态模型
        quiet_hours=[["22:00", "07:00"]],   # 静默时段
    )
```

### UserSettings（用户可调参数）

保存在 `~/.upclock/config.json`，通过状态栏"提醒设置…"面板修改：

```json
{
  "prolonged_seated_minutes": 45,
  "notification_cooldown_minutes": 30,
  "quiet_hours": [["22:00", "07:00"], ["12:30", "13:30"]]
}
```

运行时 `SharedState` 会优先使用 `UserSettings`，并在用户保存后动态更新。

## 视觉模块（可选）

摄像头姿态检测采用自适应采集策略：

- 键鼠静默 >60 秒或久坐达 95% 时短暂启用 3 秒
- 支持三种后端：
  - `mediapipe`（默认，BlazePose Lite）
  - `onnx`（MoveNet SinglePose，需提供 `.onnx` 模型）
  - `simulated`（差分模拟数据，摄像头不可用时自动回退）

**权限要求**：首次使用需在"系统设置 → 隐私与安全"授予摄像头访问权限。

## API 接口

FastAPI 默认监听 `http://127.0.0.1:8000`：

- `GET /`：仪表盘首页（静态 HTML）
- `GET /metrics`：当前活动状态（JSON）
- `WebSocket /ws`：实时指标推送（5 秒间隔）

### /metrics 响应示例

```json
{
  "activity_sum": 128,
  "normalized_activity": 0.42,
  "seated_minutes": 23.5,
  "break_minutes": 0.0,
  "presence_confidence": 0.85,
  "posture_score": 0.73,
  "posture_state": "upright",
  "score": 0.68,
  "state": "ACTIVE",
  "daily_prolonged_seated_minutes": 15,
  "daily_break_count": 2,
  "daily_max_seated_minutes": 42,
  "flow_mode_active": false,
  "snooze_active": false,
  "quiet_active": true,
  "quiet_remaining_minutes": 127.5
}
```

## 状态栏图标映射

- `👨🏻‍💻`：活跃状态（ACTIVE）
- `☕`：短暂休息（SHORT_BREAK）
- `💥`：久坐超阈值（PROLONGED_SEATED）

## 测试策略

- **单元测试**：`tests/` 目录，覆盖 `ActivityEngine`、信号缓冲、评分逻辑
- **模拟数据**：视觉模块提供 `SimulatedVisionAdapter`，无需真实摄像头
- **集成测试**：通过 `SharedState` 注入事件流，验证端到端流程

运行测试时确保未开启真实视觉采集（`vision_enabled=False`），避免测试依赖硬件。

## 常见开发任务

### 添加新的窗口分类规则

编辑 `config.local.py`：

```python
AppConfig(
    window_categories=[
        WindowCategory(name="work", weight=1.2, patterns=["Xcode", "VSCode"]),
        WindowCategory(name="leisure", weight=0.5, patterns=["YouTube", "Netflix"]),
    ]
)
```

### 调整专注指数权重

修改 `src/upclock/core/activity_engine.py:ActivityEngine._compute_score()`：

```python
# 当前公式：activity_factor + time_penalty + posture_bonus
# 可调整各因子权重或新增信号
```

### 禁用摄像头回退到键鼠模式

```python
AppConfig(vision_enabled=False)
```

或在 `~/.upclock/config.json` 中设置环境变量/启动参数。

## 权限与隐私

- **辅助功能**：macOS 需授予终端/Python 辅助功能权限以监听键鼠
- **摄像头**：仅在 `vision_enabled=True` 时请求，原始帧不保存
- **数据存储**：聚合指标保存在内存，用户设置保存在 `~/.upclock/config.json`

## 技术债务与改进方向

参考 `AGENT.md` 的迭代路线图：

- 迭代 2：SQLite 持久化历史数据（可选）
- 迭代 3：插件化传感器架构
- 迭代 4：Svelte 重构仪表盘前端

## 故障排查

- **状态栏图标不显示**：检查 rumps 是否安装（`uv sync`）
- **通知不弹出**：确认"系统设置 → 通知"允许 Python/终端发送通知
- **键鼠活动为 0**：检查"辅助功能"权限是否授予
- **摄像头无法启动**：查看日志，自动回退到 `SimulatedVisionAdapter`
- **打包失败**：确保使用正确的虚拟环境（`UV_PROJECT_ENVIRONMENT`）

---
> Source: [LTXWorld/upClock](https://github.com/LTXWorld/upClock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
