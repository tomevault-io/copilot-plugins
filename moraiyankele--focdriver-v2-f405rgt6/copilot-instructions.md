## focdriver-v2-f405rgt6

> STM32F405RGT6 无刷电机 FOC（磁场定向控制）驱动器，用于机器人关节控制。

# CLAUDE.md - FOCDriver V2 F405RGT6

## 项目概述

STM32F405RGT6 无刷电机 FOC（磁场定向控制）驱动器，用于机器人关节控制。
支持 CAN 总线 MIT 协议，多种控制模式（力矩/速度/位置/MIT/棘轮/惯性）。

- **MCU**: STM32F405RGT6, ARM Cortex-M4 @ 168 MHz, 硬件 FPU
- **OS**: FreeRTOS
- **开发环境**: VSCode + EIDE 插件, ARM GCC 编译器
- **烧录调试**: J-Link
- **CubeMX**: `FOCDriver_V2_F405RGT6.ioc`

## 核心约定

### 代码修改规则

1. **用户代码全部在 `User/` 目录**，`Core/`、`Drivers/`、`Middlewares/` 由 CubeMX 生成，**禁止随意修改**
2. **先审查再修改**：对代码的任何改动，必须先阅读现有代码并说明修改意图，经确认后再执行
3. **不做多余改动**：不要添加不相关的注释、重构、格式化或"顺手改进"
4. **中文交流**：所有沟通使用中文

### 语言混用规则

- **C++**：BSP 层 (`User/Bsp/`)、App 层 (`User/App/`)、部分 Lib（Foc.cpp, Log.cpp, ws2812.cpp）
- **纯 C**：算法模块 — `pid.c`, `MT6701.c`, `LowPassFilter.c`, `VOFA.c`, `dwt.c`
- C++ 文件中引用 C 头文件需用 `extern "C" {}` 包裹

### 命名规范

| 类别 | 风格 | 示例 |
|------|------|------|
| 类名 | `PascalCase` | `Foc`, `BspUart`, `Can` |
| 成员变量 | `camelCase` | `motorAngle`, `targetSpeed` |
| 宏/常量 | `UPPER_SNAKE_CASE` | `PWM_MAX_VALUE`, `POLE_PAIRS` |
| C++ 方法 | `PascalCase` | `UpdateCurrent()`, `MotorControlTask()` |
| C 函数 | `snake_case` | `mt6701_read()`, `pid_update()` |

### ISR 约束（极其重要）

- `MotorTask()` 在 ADC 注入完成 ISR 中直接执行（20kHz），**严格控制执行时间**
- ISR 中**禁止**调用任何阻塞函数（FreeRTOS API 必须用 `FromISR` 变体）
- ISR 中**禁止**进行 SPI 阻塞读取等耗时操作（MT6701 已改为 DMA 预取模式）

## 目录结构

```
User/                        # ★ 所有用户代码
├── App/                     # 应用层
│   ├── Inc/
│   │   ├── common_inc.h     # 全局宏、电机型号选择
│   │   ├── CmdTask.h        # CAN 指令任务
│   │   ├── MotorTask.h      # 电机控制任务
│   │   └── Callback.h       # HAL 中断路由
│   └── Src/
│       ├── user_main.cpp    # 入口，任务创建
│       ├── CmdTask.cpp      # CAN 命令解析、MIT 协议
│       ├── MotorTask.cpp    # 20kHz 实时控制循环
│       └── Callback.cpp     # HAL 回调路由
├── Bsp/                     # 板级支持包（C++ 封装 HAL）
│   ├── Inc/                 # BspUart.h, BspCan.h, BspPwm.h, BspTimer.h, BspDevice.h
│   └── Src/                 # 对应实现
└── Lib/                     # 算法与工具库
    ├── Inc/                 # Foc.h, pid.h, MT6701.h, LowPassFilter.h, Log.h, VOFA.h
    └── Src/                 # 对应实现
```

## 构建

- **工具链**: EIDE（VSCode 插件）+ ARM GCC
- **编译**: 在 VSCode 中使用 EIDE 工具栏的 Build 按钮，或 EIDE 命令面板
- **输出**: `build/` 目录，生成 `.hex` / `.bin` / `.elf`
- **烧录**: J-Link，通过 EIDE 的 Flash 功能烧录
- **关键编译宏**: `USE_HAL_DRIVER`, `STM32F405xx`

## 实时控制流

```
TIM1_CC4 触发 ADC 注入转换 (20kHz)
  → ADC 完成 ISR → Callback.cpp 路由
    → MotorTask()
      ├─ UpdateCurrent()     # ADC → Clarke → Park → LPF
      ├─ UpdateMotorAngle()  # MT6701 SPI DMA 预取
      ├─ UpdateVelocity()    # 每 20 次 (1kHz)
      └─ MotorControlTask()  # 电流环(20kHz) + 速度/位置环(1kHz) → SVPWM → PWM
```

## 电机参数切换

在 `User/App/Inc/common_inc.h` 中通过宏选择电机型号：
```c
#define MOTOR_4621  1   // 默认：11极对，0.14 Nm/A
#define MOTOR_2804  0   // 备选：7极对，0.06 Nm/A
```

PID 参数和限幅值在 `User/Lib/Inc/Foc.h` 中按电机型号条件编译。

## CAN 通信

- CAN1, 1 Mbps, 标准 11 位帧
- MIT 协议：8 字节打包（位置/速度/KP/KD/力矩）
- 电机 ID: `0x100`(M1), `0x200`(M2), `0x300`(M3), `0x400`(M4)
- 解析在 `CmdTask.cpp`，通过 `vTaskNotifyGiveFromISR()` 唤醒

## 调试方式

- **VOFA+**：通过 USART1 DMA 输出 JustFloat 格式数据，实时查看波形
- VOFA+ 同时支持**实时修改变量**（如 PID 参数、控制目标等），用于在线调参
- 相关函数：`VOFA_SendJustFloat()`, `VOFA_RegisterData_float()`
- Log 系统（`Log.h`）用于文本调试输出，宏接口：`LOG_INFO()`, `LOG_WARN()`, `LOG_ERROR()`

## 参考文档

- `PROJECT_ARCHITECTURE.md` — 完整架构文档
- `BSP_Driver_Doc.md` — BSP 层设计文档
- `Timer_Usage.md` — 定时器配置文档

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MoraiYanKele) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
