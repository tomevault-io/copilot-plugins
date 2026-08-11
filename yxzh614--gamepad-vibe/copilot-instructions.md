## gamepad-vibe

> PS DualSense 手柄 → macOS 键盘映射工具。通过 IOKit HID 直接读取手柄原始报告（蓝牙或 USB 有线），解析按键/摇杆状态，模拟键盘事件发送给系统。

# AGENTS.md - Gamepad Vibe 项目指南

## 项目概述

PS DualSense 手柄 → macOS 键盘映射工具。通过 IOKit HID 直接读取手柄原始报告（蓝牙或 USB 有线），解析按键/摇杆状态，模拟键盘事件发送给系统。

## 架构

```
gamepad_reader.swift  ← 主程序，HID 读取 + 按键映射
gamepad.json          ← 用户按键映射配置（覆盖默认值）
gamepad_reader        ← 编译后的二进制
ps_controller_terminal.py ← 废弃，早期 Python 方案（无法从 CLI 读取 HID 输入）
```

## 编译运行

```bash
swiftc -O gamepad_reader.swift -o gamepad_reader -framework IOKit -framework CoreGraphics
./gamepad_reader
```

运行时命令：
- `d` — 切换 RAW 模式（打印原始 hex，关闭按键映射）
- `q` — 退出

**依赖：** macOS 辅助功能权限（系统设置 → 隐私与安全 → 辅助功能 → Terminal/iTerm2）

## DualSense Report 格式

### BT Report 0x31（蓝牙，已验证）

78 字节报告，reportID=0x31：

| 字节 | 含义 | 备注 |
|------|------|------|
| 0 | 0x31 | Report ID |
| 1 | 序列号 | 每帧递增 |
| 2 | LX | 左摇杆 X (0x00=左, 0x80=中, 0xFF=右) |
| 3 | LY | 左摇杆 Y (0x00=上, 0x80=中, 0xFF=下) |
| 4 | RX | 右摇杆 X |
| 5 | RY | 右摇杆 Y |
| 6 | L2 axis | L2 模拟行程 (0x00~0xFF) |
| 7 | R2 axis | R2 模拟行程 |
| 8 | 状态 | 0x01=空闲值 |
| 9 | D-pad + 面部按键 | 低4位=D-pad(0=上,8=无), 高4位=□×○△ |
| 10 | 肩键 | L1/R1/L2d/R2d/Share/Options/L3/R3 |
| 11 | Touch/Mic | bit1=触摸板按下, bit2=Mic键 |
| 34 | 触摸板触点 | 非零=手指在板上 |
| 35-37 | 触摸坐标 | X/Y 12-bit 编码 |

### USB Report 0x01（有线）

64 字节报告，reportID=0x01：

| 字节 | 含义 | 备注 |
|------|------|------|
| 0 | 0x01 | Report ID |
| 1 | 序列号 | 每帧递增 |
| 2 | LX | 左摇杆 X |
| 3 | LY | 左摇杆 Y |
| 4 | RX | 右摇杆 X |
| 5 | RY | 右摇杆 Y |
| 6 | L2 axis | L2 模拟行程 |
| 7 | R2 axis | R2 模拟行程 |
| 8 | D-pad + 面部按键 | 低4位=D-pad, 高4位=□×○△ |
| 9 | 肩键 | L1/R1/L2d/R2d/Share/Options/L3/R3 |
| 10 | PS/Touch/Mic | bit0=PS, bit1=触摸板, bit2=Mic |

### BT 与 USB 的区别

摇杆和扳机（bytes 2-7）偏移相同。按钮数据 USB 比 BT **提前 1 字节**（USB 从 byte[8] 开始，BT 从 byte[9] 开始）。

**注意：** 早期使用的 BT report 0x01 格式不适用于此设备，实际蓝牙 reportID 是 0x31。USB 有线连接使用 reportID 0x01。

## 按键映射

### 默认映射（Agent 开发场景）

| 手柄按键 | 键值 | 实现方式 |
|----------|------|----------|
| D-Pad ↑↓ | 方向键 | CGEvent keycode |
| D-Pad ← | / (斜杠) | AppleScript |
| D-Pad → | Shift+Tab | AppleScript |
| × Cross | Enter | CGEvent keycode |
| ○ Circle | Esc | CGEvent keycode |
| □ Square | Tab | CGEvent keycode |
| △ Triangle | Ctrl+K (删至行尾) | AppleScript |
| L1 | Ctrl+A (行首) | AppleScript |
| R1 | Ctrl+E (行尾) | AppleScript |
| L2 | Cmd+[ (上一标签页) | AppleScript |
| R2 | Cmd+] (下一标签页) | AppleScript |
| L3 | y | AppleScript |
| R3 | Ctrl+C | AppleScript |
| Share | Ctrl+R (搜索历史) | AppleScript |
| Options | Ctrl+L (清屏) | AppleScript |
| Touchpad | Space | AppleScript |
| Mic | PageDown | CGEvent keycode (用户配置覆盖) |
| 左摇杆 | 方向键 | CGEvent, 持续推自动重复 |

### 实现方式说明

- **keycode 类型** → CGEvent 直接发送（快速，全局热键应用可捕获，如 Typeless）
- **keystroke 类型** → AppleScript via osascript（支持修饰键组合）
- **none 类型** → 忽略该按键

### 配置文件

`./gamepad.json`，格式：

```json
{
  "repeat_interval_ms": 120,
  "stick_deadzone": 40,
  "mappings": {
    "cross":    {"type": "keycode", "code": 36},
    "triangle": {"type": "keystroke", "key": "k", "mods": ["control"]},
    "mic":      {"type": "keycode", "code": 121},
    "l3":       {"type": "keystroke", "key": "y"},
    "r3":       {"type": "none"}
  }
}
```

支持的字段：
- `type`: `"keycode"` | `"keystroke"` | `"none"`
- `code`: macOS key code（keycode 类型）
- `key`: 要输入的字符（keystroke 类型）
- `mods`: 修饰键数组 `["control", "command", "shift", "option"]`

## 已知问题

- PS 按钮位置未确认（byte 8 bit0 疑似状态标志，非按钮）
- 触摸板只支持按下检测，不支持滑动/手势
- 右摇杆未映射按键

---
> Source: [yxzh614/gamepad-vibe](https://github.com/yxzh614/gamepad-vibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
