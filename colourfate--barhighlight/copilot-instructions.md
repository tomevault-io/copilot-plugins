## barhighlight

> Windows 任务栏高亮工具，为任务栏图标添加彩色高亮标记。

# BarHighLight

Windows 任务栏高亮工具，为任务栏图标添加彩色高亮标记。

## 项目概述

通过 PyQt6 透明覆盖层和 Windows UI Automation API，检测任务栏图标位置并绘制高亮效果。支持横线模式和半透明覆盖模式，通过系统托盘管理。

## 技术栈

- Python 3
- PyQt6 (覆盖层窗口 + 配置编辑器)
- uiautomation (任务栏图标检测)
- pystray + Pillow (系统托盘)
- comtypes (Windows API 交互)

## 运行命令

```bash
# 安装依赖
pip install -r requirements.txt

# 启动程序
python main.py

# 运行覆盖层演示
python demo_pyqt_overlay.py          # 交互式
python demo_pyqt_overlay.py auto     # 自动模式
python demo_pyqt_overlay.py coords   # 打印坐标信息
```

## 项目结构

```
main.py              - 应用入口，App 主类，QTimer 主循环
overlay_window.py    - PyQt6 透明覆盖层，负责绘制高亮
taskbar_monitor.py   - Windows UIA 任务栏图标检测
config_manager.py    - 配置管理 (config.json 读写)
config_editor.py     - PyQt6 配置编辑器 GUI
tray_app.py          - 系统托盘图标和菜单
logging_config.py    - 日志配置 (控制台 + 文件)
demo_pyqt_overlay.py - 覆盖层功能演示
requirements.txt     - Python 依赖
```

## 架构要点

- **主循环**: `App.run()` 使用 QTimer 定时轮询任务栏图标状态
- **线程模型**: 主线程运行 Qt 事件循环；托盘图标和配置编辑器在独立守护线程运行
- **跨线程通信**: 使用 `pyqtSignal` 将回调从托盘线程转发到主线程
- **覆盖层**: 无边框、置顶、透明背景、点击穿透 (WS_EX_TRANSPARENT)
- **DPI 适配**: 通过 `GetSystemMetrics(SM_CXVIRTUALSCREEN)` 获取物理像素，计算 DPI 缩放比
- **配置热更新**: `ConfigManager` 支持回调监听，配置变更后自动刷新覆盖层和定时器间隔

## 配置文件

`config.json` (git 忽略)，包含:

```json
{
    "enabled": true,
    "mode": "line",
    "opacity": 200,
    "line_height": 4,
    "refresh_interval": 800,
    "auto_start": false,
    "debug": false,
    "highlights": {
        "chrome.exe": "#FF5722",
        "code.exe": "#2196F3"
    }
}
```

- `mode`: `"line"` (横线) 或 `"overlay"` (半透明覆盖)
- `highlights`: 进程名 -> 颜色映射

## 代码约定

- 日志模块名: `BarHighLight.<子模块>` (如 `BarHighLight.overlay`)
- 所有文件使用 UTF-8 编码
- Windows 专有 API 通过 `ctypes.windll` 直接调用
- 配置变更统一走 `ConfigManager` 的 save/notify 流程
- 任务栏窗口层级: `Shell_TrayWnd > ReBarWindow32 > MSTaskSwWClass > MSTaskListWClass`

---
> Source: [colourfate/BarHighLight](https://github.com/colourfate/BarHighLight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
