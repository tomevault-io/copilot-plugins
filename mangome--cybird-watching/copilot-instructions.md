## cybird-watching

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个名为"CybirdWatching"的嵌入式项目，基于ESP32平台开发。该项目是一个智能显示设备，集成了多种传感器、显示模块、网络连接和用户界面功能。项目使用Arduino框架和PlatformIO进行开发。

## 开发环境

- **平台**: ESP32 (pico32)
- **框架**: Arduino
- **构建系统**: PlatformIO
- **串口速度**: 115200
- **上传端口**: COM3（在 platformio.ini 中配置）

## 构建和开发命令

### 常用PlatformIO命令
```bash
# 编译项目
platformio run

# 上传到设备
platformio run --target upload

# 监控串口输出
platformio device monitor

# 清理构建文件
platformio run --target clean

# 上传并监控（推荐）
platformio run --target upload && platformio device monitor
```

### Windows快捷脚本（在 scripts/ 目录）
```bash
# 编译项目
.\pio_run.bat

# 上传并监控
.\upload_and_monitor.bat

# CLI工具（交互式串口命令）
.\cybird_cli.bat
```

## 项目架构

### v3.0 双核FreeRTOS架构

项目采用双核分离设计，充分利用ESP32的双核优势：

```
┌─────────────────────────────────────────────────────┐
│                     ESP32                           │
├─────────────────────┬───────────────────────────────┤
│   Core 0 (UI)       │   Core 1 (System)             │
│   优先级: 2         │   优先级: 1                    │
│   栈: 8KB           │   栈: 8KB                      │
│   频率: 200Hz       │   频率: 100Hz                  │
├─────────────────────┼───────────────────────────────┤
│ • LVGL GUI          │ • IMU 传感器                   │
│ • Display 刷新      │ • 串口命令                     │
│ • 小鸟动画          │ • WiFi 通信                    │
│ • 图片解码          │ • SD 卡操作                    │
│                     │ • 业务逻辑                     │
└─────────────────────┴───────────────────────────────┘
           ↕                        ↕
    ┌──────────────────────────────────────┐
    │   消息队列 + LVGL 互斥锁              │
    └──────────────────────────────────────┘
```

**关键要点**:
- UI任务运行在Core 0，专注于渲染，保证界面流畅
- 系统任务运行在Core 1，处理传感器、IO和业务逻辑
- 通过LVGL互斥锁保证线程安全
- 通过消息队列实现任务间通信
- 详见: [双核架构文档](docs/DUAL_CORE_ARCHITECTURE.md)

### 目录结构
```
src/
├── main.cpp                           # 主程序入口
├── drivers/                           # 硬件驱动层
│   ├── display/                       # 显示驱动 (TFT_eSPI)
│   ├── sensors/                       # 传感器驱动
│   │   ├── imu/                       # IMU传感器 (MPU6050)
│   │   └── ambient/                   # 环境传感器
│   ├── communication/network/         # 网络通信 (WiFi)
│   ├── storage/sd_card/               # SD卡存储
│   └── io/rgb_led/                    # RGB LED控制 (FastLED)
├── system/                            # 系统服务层
│   ├── logging/                       # 日志管理系统
│   ├── commands/                      # 串口命令系统
│   └── lvgl/ports/                    # LVGL端口层
└── applications/                      # 应用层
    └── gui/                           # 图形用户界面
        ├── core/                      # 核心GUI组件
        └── screens/                   # GUI屏幕
```

### 核心组件

#### 1. 显示系统 (`src/drivers/display/`)
- 基于TFT_eSPI库的LCD显示控制
- LVGL图形库集成
- 支持背光PWM控制
- 负责屏幕初始化和日常刷新

#### 2. 传感器系统 (`src/drivers/sensors/`)
- **IMU**: MPU6050加速度计和陀螺仪，用作用户输入设备
- **环境传感器**: 环境光线、温度等传感器支持

#### 3. 网络通信 (`src/drivers/communication/network/`)
- WiFi网络连接管理
- 从SD卡读取WiFi配置 (`/wifi.txt`)
- 支持网络数据传输功能

#### 4. 存储系统 (`src/drivers/storage/sd_card/`)
- SD卡文件系统支持
- 日志文件存储
- 配置文件读取

#### 5. 用户界面 (`src/applications/gui/`)
- 基于LVGL的图形用户界面
- 包含主屏幕和场景屏幕
- 支持触摸和IMU输入

#### 6. 日志系统 (`src/system/logging/`)
- 单例模式的日志管理器
- 支持串口和SD卡输出
- 多级日志 (DEBUG, INFO, WARN, ERROR, FATAL)
- 日志文件自动轮转
- 专为嵌入式环境优化

#### 7. 命令系统 (`src/system/commands/`)
- 串口命令解析和执行
- 支持系统控制和调试命令
- 非阻塞式命令处理

### 外部依赖库

#### 核心库 (`lib/`)
- **FastLED**: 高性能LED控制库，用于RGB LED
- **MPU6050**: IMU传感器驱动库
- **TFT_eSPI**: 高性能TFT LCD驱动库
- **LVGL**: 轻量级图形用户界面库

### 系统初始化流程

v3.0双核架构的初始化顺序（详见 src/main.cpp:30）：

1. 看门狗配置 (10秒超时，防止图像加载时触发)
2. 串口初始化 (115200 baud)
3. 日志系统初始化 (先串口输出)
4. 串口命令系统初始化
5. **SD卡初始化** (必须在显示屏之前，避免SPI冲突)
6. 日志系统切换为SD卡输出（保持CLI输出清洁）
7. 显示屏初始化和背光设置 (0.2亮度)
8. LVGL文件系统初始化
9. LVGL输入设备初始化
10. IMU (MPU6050) 初始化
11. RGB LED初始化
12. GUI界面创建 (setup_ui)
13. **Task Manager初始化** (创建LVGL互斥锁)
14. **双核任务启动** (Core 0: UI任务 200Hz, Core 1: 系统任务 100Hz)
15. Logo显示 (lv_init_gui)
16. **Bird Watching系统初始化** (扫描小鸟资源，耗时操作)
17. Logo关闭，切换到小鸟界面

**⚠️ 关键时序要求**:
- SD卡必须在显示屏之前初始化（避免SPI总线冲突）
- Task Manager必须在双核任务启动前初始化（提供LVGL互斥锁）
- 双核任务必须在logo显示前启动（UI任务负责渲染）
- 小鸟资源扫描在logo显示期间进行（提升用户体验）

### 主循环逻辑（v3.0双核架构）

在v3.0双核架构中，主loop已经不再承担核心功能：

```cpp
void loop() {
    // 所有核心功能已经在FreeRTOS任务中运行：
    // - Core 0: UI Task (200Hz - LVGL + Display + Animation)
    // - Core 1: System Task (100Hz - Sensors + Commands + Business Logic)

    // 可选：每60秒打印一次任务统计信息
    if (taskManager) {
        taskManager->printTaskStats();
    }

    delay(1000);  // 让出CPU给FreeRTOS调度器
}
```

**实际任务分配**:
- **UI任务** (Core 0, 200Hz): `screen.routine()` + `lv_timer_handler()` + 动画播放
- **系统任务** (Core 1, 100Hz): `mpu.update(200)` + `SerialCommands::handleInput()` + BirdManager逻辑

### 配置文件

- **WiFi配置**: `/wifi.txt` (第1行SSID, 第2行密码)
- **小鸟配置**: `/configs/bird_config.csv` - 小鸟ID、名称、权重配置
- **项目配置**: `platformio.ini` - 包含构建标志和库依赖
- **包含路径**: 所有模块的包含路径已在platformio.ini中配置

### 常用串口命令参考

使用 `cybird_cli.bat` 或串口工具连接后，常用命令：

```bash
# 系统命令
help                    # 显示所有命令
status                  # 显示系统状态
clear                   # 清空终端

# 小鸟系统
bird trigger [id]       # 触发小鸟动画（可选指定ID）
bird list               # 列出所有小鸟
bird stats              # 查看观鸟统计
bird reset              # 重置统计数据

# 任务监控
task stats              # 查看任务统计（栈使用、CPU占用）
task info               # 查看详细系统信息

# 日志管理
log                     # 查看最后20行日志
log lines <N>           # 查看最后N行日志
log cat                 # 查看完整日志
log clear               # 清空日志
log level <level>       # 设置日志级别

# 文件管理
tree [path] [levels]    # 显示目录树
file info <path>        # 查看文件信息
file delete <path>      # 删除文件
```

详细命令说明见 README.md 或使用 `help` 命令。

### 特殊功能

#### 串口命令系统
- 支持实时系统控制
- 日志查看和管理命令
- 调试和状态查询命令
- 不干扰GUI显示的SD卡专用日志记录

#### 日志管理
- 智能日志轮转防止SD卡写满
- 串口和SD卡双输出模式
- 可配置的日志级别
- 内置时间戳和标签系统

## 开发注意事项

### 必须遵循的规则

1. **LVGL线程安全** ⚠️ 最重要：
   ```cpp
   // 从系统任务访问LVGL对象时，必须获取互斥锁
   TaskManager* taskMgr = TaskManager::getInstance();
   if (taskMgr->takeLVGLMutex(100)) {
       lv_obj_set_pos(obj, x, y);
       lv_label_set_text(label, "Hello");
       taskMgr->giveLVGLMutex();
   }
   ```
   - UI任务（Core 0）不需要加锁，因为它独占LVGL
   - 系统任务（Core 1）访问LVGL对象必须加锁
   - 超时建议: 100ms（普通操作）, 1000ms（耗时操作）

2. **头文件包含**: 所有必要的包含路径已在`platformio.ini`中配置（第24-41行）

3. **日志系统**:
   ```cpp
   #include "system/logging/log_manager.h"
   LOG_INFO("TAG", "Message: %d", value);
   LOG_ERROR("TAG", "Error occurred");
   ```
   - 使用`LOG_*`宏而非`Serial.print`
   - v3.0默认只输出到SD卡，保持CLI清洁
   - 使用串口命令 `log cat` 查看完整日志

4. **串口命令系统**:
   - 添加新命令: 编辑 `src/system/commands/serial_commands.cpp`
   - 在`initialize()`中注册，在`processCommand()`中处理
   - 使用`cybird_cli.bat`进行交互式调试

5. **内存管理**:
   - 注意ESP32内存限制（SRAM约320KB）
   - 避免创建大型静态对象
   - 使用`task stats`命令监控栈使用
   - UI任务和系统任务各8KB栈，可在`task_manager.cpp`调整

6. **任务设计**:
   - UI相关代码应在UI任务中执行（或加锁）
   - 传感器读取、文件IO在系统任务中执行
   - 避免在任务中使用`delay()`，使用`vTaskDelay()`
   - 看门狗超时10秒，耗时操作需注意

7. **小鸟资源**:
   - 图片格式: RGB565 .bin文件
   - 尺寸: 120x120像素
   - 位置: SD卡 `/birds/<id>/x.bin`
   - 配置: `/configs/bird_config.csv`
   - 详见: [添加小鸟指引](docs/GUIDE_TO_ADD_NEW_BIRDS.md)

8. **调试技巧**:
   ```bash
   task stats          # 查看任务统计
   task info           # 查看系统信息
   log lines 50        # 查看最近50行日志
   bird trigger 1001   # 手动触发小鸟测试
   tree / 2            # 查看SD卡目录
   ```

## 快速参考

### 关键文件位置

| 功能 | 文件路径 |
|------|---------|
| 主程序入口 | `src/main.cpp` |
| 任务管理器 | `src/system/tasks/task_manager.cpp` |
| 串口命令系统 | `src/system/commands/serial_commands.cpp` |
| 日志管理器 | `src/system/logging/log_manager.cpp` |
| 小鸟管理器 | `src/applications/modules/bird_watching/core/bird_manager.cpp` |
| 小鸟动画系统 | `src/applications/modules/bird_watching/core/bird_animation.cpp` |
| GUI主逻辑 | `src/applications/gui/core/lv_cubic_gui.cpp` |
| SD卡小鸟配置 | `resources/configs/bird_config.csv` |
| 字体文件 | `src/applications/modules/resources/fonts/` |
| PlatformIO配置 | `platformio.ini` |

### 修改上传端口

编辑 `platformio.ini`:
```ini
upload_port = COM5      # 改为你的端口
```

### 调整任务栈大小

如果遇到栈溢出，编辑 `src/system/tasks/task_manager.cpp`:
```cpp
#define UI_TASK_STACK_SIZE      10240  // 默认8192
#define SYSTEM_TASK_STACK_SIZE  10240  // 默认8192
```

### 性能优化建议

1. **提升UI流畅度**: 增加UI任务栈 或 降低系统任务频率
2. **减少内存占用**: 优化小鸟资源尺寸、减少字体大小
3. **提升响应速度**: 提高系统任务优先级（不推荐，可能影响UI）
4. **监控性能**: 定期运行 `task stats` 查看栈使用和CPU占用

### 相关文档链接

- **添加小鸟**: [docs/GUIDE_TO_ADD_NEW_BIRDS.md](docs/GUIDE_TO_ADD_NEW_BIRDS.md)
- **双核架构**: [docs/DUAL_CORE_ARCHITECTURE.md](docs/DUAL_CORE_ARCHITECTURE.md)
- **修改字体**: [docs/CHANGE_FONT_SIZE.md](docs/CHANGE_FONT_SIZE.md)
- **CLI工具**: [scripts/README_CLI_TOOLS.md](scripts/README_CLI_TOOLS.md)
- **更新日志**: [CHANGELOG.md](CHANGELOG.md)

---
> Source: [Mangome/cybird-watching](https://github.com/Mangome/cybird-watching) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
