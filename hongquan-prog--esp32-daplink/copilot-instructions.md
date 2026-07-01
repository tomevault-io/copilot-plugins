## esp32-daplink

> > **文档语言声明**: 本项目的所有文档（包括但不限于 CLAUDE.md、README）均采用中文编写，代码注释使用英语。

# CLAUDE.md

> **文档语言声明**: 本项目的所有文档（包括但不限于 CLAUDE.md、README）均采用中文编写，代码注释使用英语。

本文件为 Claude Code (claude.ai/code) 在此代码库中工作时提供指导。

## Commit 消息规范

### 基本格式

```
[area]: [summary]

[详细描述（必须非空）]

Signed-off-by: [Your Name] <[your.email@example.com]>
```

### 标题行

标题行格式为 `[area]: [summary]`，必须满足：
- 单行
- 少于 72 字符
- 后面必须跟一个空行

**Area 标签示例**：
- `main:` - main 目录相关更改
- `usbipd:` - USBIP 服务器相关
- `debug_probe:` - DAP 调试相关
- `Program:` - 离线编程相关
- `build:` - 构建系统、CMake、Kconfig
- `doc:` - 文档更改
- `util:` - 工具类更改
- `test:` - 测试相关

**Summary 示例**：
- `main: fix USB CDC callback race condition`
- `usbipd: add URB timeout handling`
- `doc: update build instructions`

### 正文

**正文必须非空**，即使是微小的更改也需要描述。

正文应包含：
- **what** - 更改做了什么
- **why** - 为什么需要这个更改
- **how** - 如何实现，做了哪些假设
- **验证** - 如何测试验证

每行通常 75 字符或更少，超长用换行处理（URL、邮箱地址除外）。

### Signed-off-by

提交时需添加 `Signed-off-by` 行：
```
Signed-off-by: Your Name <your.email@example.com>
```

**必须从全局 Git 配置读取**：
- Name: `git config --global user.name`
- Email: `git config --global user.email`

提交时必须使用上述配置中的值，不允许硬编码或使用占位符。
可使用 `git commit -s` 自动添加。

### AI 辅助规范

- **不要添加 AI 签名**（如 `Co-Authored-By: Claude ...`）
- AI 生成的代码由人工审查后提交
- 人工提交者负责代码质量和 DCO 认证

### 示例

```
usbipd: fix URB queue deadlock

The URB processor thread could deadlock when the queue was full
and the producer thread was blocked. This fix adds a timeout
to the queue push operation and returns error when timeout occurs.

Testing: Verified with multiple concurrent USBIP connections.

Signed-off-by: Developer Name <developer@example.com>
```

### 链接 Issue

如果 Commit 关联 GitHub Issue，在正文或 PR 描述中添加：
```
Fixes #[issue number]
```

## 项目概述

ESP32-DAPLink 是基于 ESP32-S3 的硬件调试器，实现了 CMSIS-DAP 协议。它提供：
- DAPLink 在线调试（HID/WinUSB 模式）
- CDC 串口通信（USB 和 Web Serial）
- 支持 Keil FLM 算法的离线固件编程
- 通过 Web 界面进行远程编程
- 通过 USBIP 协议进行无线调试

## 构建命令

这是一个 ESP-IDF 项目。确保已设置 `IDF_PATH` 环境变量。

```bash

# 构建项目
idf.py build

# 烧录到设备
idf.py flash

# 构建并烧录
idf.py build flash

# 监控串口输出
idf.py monitor

# 构建、烧录并监控
idf.py build flash monitor

# 清理构建
idf.py fullclean && idf.py build

# 配置 (menuconfig)
idf.py menuconfig
```

## 架构

### 组件结构

```
components/
├── debug_probe/   # CMSIS-DAP 实现 (DAP.c, SW_DP.c, JTAG_DP.c, swd.c)
├── Program/       # 离线编程（flash 算法、HEX/BIN 解析、SWD 主机）
├── usbipd/        # USBIP 服务器用于无线调试
│   ├── include/   # 公共头文件
│   │   ├── usbip_server.h      # 服务器主接口
│   │   ├── usbip_protocol.h    # USBIP 协议定义
│   │   ├── usbip_devmgr.h      # 设备驱动接口
│   │   ├── usbip_control.h     # 控制传输框架
│   │   ├── usbip_hid.h         # HID 设备基类
│   │   ├── usbip_common.h      # 公共定义
│   │   └── hal/                # 硬件抽象层
│   │       ├── usbip_osal.h    # OS 抽象（互斥锁、线程等）
│   │       ├── usbip_transport.h # 传输抽象
│   │       └── usbip_log.h     # 日志系统
│   └── src/
│       ├── server/             # 服务器核心
│       │   ├── usbip_server.c  # 连接管理
│       │   ├── usbip_urb.c     # URB 处理（生产者-消费者）
│       │   ├── usbip_devmgr.c  # 设备管理
│       │   └── usbip_protocol.c # 协议工具
│       ├── device/             # 设备基类
│       │   ├── usbip_hid.c     # HID 设备实现
│       │   └── usbip_bulk.c    # Bulk 设备实现
│       ├── hal/                # HAL 实现
│       │   ├── usbip_osal.c    # OSAL 封装函数
│       │   ├── usbip_mempool.c # 静态内存池
│       │   └── usbip_transport.c # 传输接口
│       ├── platform/espidf/    # ESP-IDF 特定
│       │   ├── osal_espidf.c   # FreeRTOS 实现
│       │   └── transport_espidf.c # lwIP TCP 传输
│       └── device_drivers/     # DAP 设备驱动
│           ├── hid_dap.c       # HID CMSIS-DAP (busid: 2-1)
│           └── bulk_dap.c      # Bulk CMSIS-DAP (busid: 2-2)
└── util/          # 工具（LED 控制）

main/              # 应用入口和处理器
├── main.cpp       # 入口点、USB 初始化、WiFi、任务创建
├── cdc_uart.c     # UART 桥接双输出（USB/Web）
├── usb_cdc_handler.c  # USB CDC 回调
├── web_handler.cpp    # WebSocket 串口处理器
├── web_server.c       # HTTP/WebSocket 服务器
├── SerialManager.cpp  # 串口状态管理（USB vs Web）
└── programmer.cpp     # 离线编程状态机
```

### 关键数据流

1. **DAP 调试 (USB)**: 主机 (Keil/OpenOCD) → USB (HID/Bulk) → DAP_ProcessCommand() → SWD/JTAG → 目标
2. **DAP 调试 (USBIP)**: 主机 → USBIP 协议 → usbip_server → hid_dap/bulk_dap → DAP_ProcessCommand() → SWD/JTAG → 目标
3. **USB 串口**: 主机 CDC → tud_cdc_rx_cb → usb_cdc_send_to_uart() → UART → 目标
4. **Web 串口**: 浏览器 WebSocket → web_handler → SerialManager → UART → 目标
5. **串口输出**: UART RX → cdc_uart → (USB CDC || WebSocket 广播)

### USBIP 架构

USBIP 实现采用模块化、多线程架构：

- **服务器线程**: 接受 TCP 连接，处理 OP_REQ_DEVLIST/OP_REQ_IMPORT
- **连接线程**: 每客户端连接处理
- **URB 处理线程**: URB 处理的生产者-消费者队列
- **设备驱动**: 通过构造函数属性自动注册

关键文件：
- `usbip_server.c`: 主服务器循环、连接管理
- `usbip_urb.c`: URB 队列和处理器线程
- `hid_dap.c`: HID 设备驱动，带 MS OS 2.0 描述符
- `bulk_dap.c`: WinUSB 的 Bulk 设备驱动

### 串口管理

`SerialManager` (main/SerialManager.cpp) 管理状态转换：
- `STATE_IDLE`: 无客户端连接，UART 数据丢弃
- `STATE_USB`: USB CDC 已连接，UART 数据 → USB
- `STATE_WEB`: Web 客户端已连接，UART 数据 → WebSocket

优先级：USB > Web > 空闲

### 编程架构

`Program` 组件实现了流式编程模式：
- `StreamProgrammer`: 主协调器
- `TargetFlash`: Flash 操作，带扇区管理
- `FlashAccessor`: 底层 Flash 访问
- `SWDIface` / `TargetSWD`: SWD 接口抽象
- `AlgoExtractor`: 从 FLM 文件提取 Flash 算法

## 代码风格

完整风格指南见 `docs/style/code_style.md`。要点：

- **命名**: 函数/变量使用 snake_case，类使用 PascalCase，私有成员使用 `_prefix` 前缀
- **花括号**: Allman 风格（左花括号在新行）
- **缩进**: 4 空格，无制表符
- **头文件**: 使用 `#pragma once`
- **C/C++**: C 头文件使用 `extern "C"` 保护
- **无全局变量**: 使用依赖注入

## 依赖项

通过 ESP-IDF 组件管理器管理 (`dependencies.lock`):
- `espressif/tinyusb`: TinyUSB 栈
- `espressif/esp_tinyusb`: ESP-IDF TinyUSB 集成
- `components/usbipd`: USBIP 服务器实现

## 重要文件

- `components/debug_probe/DAP/Config/DAP_config.h`: DAP 引脚分配
- `components/usbipd/Kconfig`: USBIP 服务器配置选项
- `algorithm/`: 离线编程的 FLM Flash 算法文件

---
> Source: [hongquan-prog/ESP32-DAPLink](https://github.com/hongquan-prog/ESP32-DAPLink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
