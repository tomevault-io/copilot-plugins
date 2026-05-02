## noise-info-toolkit-gcc

> > 本文档供AI编码助手阅读，包含项目背景、架构和开发规范。

# AGENTS.md - AI编码助手项目指南

> 本文档供AI编码助手阅读，包含项目背景、架构和开发规范。

## 项目概述

**noise_info_toolkit_gcc** 是一个噪声剂量计算工具包的C++实现，从Python版本移植而来。用于职业噪声暴露评估，支持多种国际标准的噪声剂量计算。

### 核心功能

- **噪声剂量计算**: 支持 NIOSH、OSHA_PEL、OSHA_HCA、EU_ISO 四种标准
- **声学指标计算**: Leq、LAeq、LCeq、Peak、TWA、LEX,8h 等
- **事件检测**: 脉冲噪声事件检测（支持LEQ/Peak/Slope三种触发模式）
- **时序处理**: 每秒时间历史数据处理和存储
- **音频处理**: WAV文件读写、1/3倍频程分析、A/C计权滤波
- **数据存储**: SQLite数据库存储（与Python版本兼容的表结构）

## 技术栈

- **语言**: C++17
- **构建系统**: CMake (最低版本 3.14)
- **数据库**: SQLite3
- **线程**: pthread (std::thread)
- **依赖库**:
  - SQLite3 (系统库)
  - Threads (pthread)

## 项目结构

```
noise_info_toolkit_gcc/
├── CMakeLists.txt           # CMake主配置
├── README.md                # 项目说明（中文）
├── AGENTS.md                # 本文件
│
├── include/                 # 头文件目录
│   ├── noise_toolkit.hpp    # 主入口头文件
│   ├── dose_calculator.hpp  # 剂量计算器
│   ├── audio_processor.hpp  # 音频处理器
│   ├── signal_utils.hpp     # 信号处理工具
│   ├── time_history_processor.hpp  # 时序处理器
│   ├── event_detector.hpp   # 事件检测器
│   ├── event_processor.hpp  # 事件处理器
│   ├── ring_buffer.hpp      # 环形缓冲区
│   ├── database.hpp         # 数据库接口
│   ├── wav_reader.hpp       # WAV文件读写
│   ├── tdms_converter.hpp   # TDMS转换器
│   └── iir_filter.hpp       # IIR滤波器
│
├── src/                     # 源文件目录
│   ├── dose_calculator.cpp
│   ├── audio_processor.cpp
│   ├── signal_utils.cpp
│   ├── time_history_processor.cpp
│   ├── event_detector.cpp
│   ├── event_processor.cpp
│   ├── ring_buffer.cpp
│   ├── database.cpp
│   ├── wav_reader.cpp
│   ├── tdms_converter.cpp
│   └── iir_filter.cpp
│
├── examples/                # 示例代码
│   └── main.cpp             # 完整功能示例
│
├── validation/              # 验证脚本（Python）
│   ├── validate_dose_calculator.py  # 剂量计算对比验证
│   └── process_tdms_python.py       # Python版TDMS处理
│
├── doc/                     # 文档
│   └── validation_report.md # 验证报告
│
├── build_test/              # 构建目录（CMake输出）
│
├── dose_validator.cpp       # 独立验证程序（单文件）
└── process_tdms_cpp.cpp     # TDMS处理验证程序（单文件）
```

## 构建指令

```bash
# 创建构建目录
cd build_test

# 配置（Release模式）
cmake .. -DCMAKE_BUILD_TYPE=Release

# 编译
make -j$(nproc)
```

### 构建选项

| 选项 | 默认值 | 说明 |
|------|--------|------|
| `BUILD_TESTS` | ON | 是否构建测试 |
| `BUILD_SHARED_LIBS` | ON | 构建动态库(.so)或静态库(.a) |

## 标准参数

| 标准 | 准则级 (dBA) | 交换率 (dB) | 参考时长 (h) | 说明 |
|------|-------------|-------------|-------------|------|
| NIOSH | 85 | 3 | 8 | 美国NIOSH标准 |
| OSHA_PEL | 90 | 5 | 8 | OSHA允许暴露限值 |
| OSHA_HCA | 85 | 5 | 8 | OSHA听力保护修正案 |
| EU_ISO | 85 | 3 | 8 | 欧盟/ISO标准 |

### 核心计算公式

- **允许暴露时间**: `T = Tref / 2^((L - Lc) / ER)`
- **剂量计算**: `Dose% = 100 × (dt/Tref) × 2^((L-Lc)/ER)`
- **TWA**: 
  - NIOSH/ISO: `TWA = 10 × log10(Dose%/100) + Lc`
  - OSHA: `TWA = 16.61 × log10(Dose%/100) + Lc`
- **LEX,8h**: `LEX = 10 × log10(Dose%/100) + Lc`

## 代码风格规范

### 文件头格式

```cpp
/**
 * @file filename.hpp
 * @brief 简短描述
 * 
 * 详细描述（可选）
 */
```

### 命名规范

| 类型 | 命名方式 | 示例 |
|------|----------|------|
| 类名 | PascalCase | `DoseCalculator`, `AudioProcessor` |
| 函数 | snake_case | `calculate_dose()`, `process_signal()` |
| 变量 | snake_case | `criterion_level`, `sample_rate` |
| 成员变量 | snake_case + 下划线后缀 | `reference_pressure_`, `profiles_` |
| 结构体 | PascalCase | `DoseProfile`, `NoiseMetrics` |
| 枚举 | 类内 enum class | `DoseStandard::NIOSH` |
| 常量 | UPPER_SNAKE_CASE | `REFERENCE_PRESSURE`, `DEFAULT_LEQ_THRESHOLD` |
| 宏 | 避免使用 | - |

### 命名空间

所有代码位于 `noise_toolkit` 命名空间下。

```cpp
namespace noise_toolkit {
    // 代码
} // namespace noise_toolkit
```

### 注释规范

- 使用 Doxygen 风格注释
- 类和方法必须添加文档注释
- 复杂算法需要添加实现注释

```cpp
/**
 * @brief 计算剂量增量
 * @param laeq A计权等效声级 (dBA)
 * @param duration_s 持续时间 (秒)
 * @param profile 剂量标准配置
 * @return 剂量百分比 (%)
 */
double calculate_dose_increment(double laeq, double duration_s, 
                                const DoseProfile& profile) const;
```

## 验证与测试

### 独立验证程序

项目包含两个独立的验证程序（单文件，不依赖库）：

```bash
# 理论值验证
./dose_validator

# TDMS处理对比
./process_tdms_cpp
```

### Python验证脚本

```bash
# 对比Python和C++计算结果
python3 validation/validate_dose_calculator.py
```

### 示例程序测试

```bash
# 运行所有测试
./noise_toolkit_example

# 运行特定测试
./noise_toolkit_example dose      # 剂量计算器
./noise_toolkit_example audio     # 音频处理器
./noise_toolkit_example time      # 时序处理器
./noise_toolkit_example buffer    # 环形缓冲区
./noise_toolkit_example database  # 数据库
```

## 关键设计说明

### 1. 剂量计算器 (DoseCalculator)

- 预定义四种标准配置（NIOSH/OSHA_PEL/OSHA_HCA/EU_ISO）
- 支持自定义标准配置
- 提供单次计算和批量计算接口

### 2. 音频处理器 (AudioProcessor)

- 处理原始音频数据或WAV文件
- 使用IIR滤波器实现A/C计权（符合IEC 61672-1）
- 计算1/3倍频程分析和总峭度

### 3. 事件检测器 (EventDetector)

- 三种触发模式：LEQ(90dB)、Peak(130dB)、Slope(10dB/50ms)
- 滑动窗口计算LZeq_125
- 支持防抖和事件回调

### 4. 环形缓冲区 (RingBuffer)

- 满足白皮书要求：≥12秒缓冲
- 预触发2秒，后触发8秒
- 用于保存事件触发前后的音频

### 5. 数据库 (Database)

- SQLite3实现
- 表结构与Python版本兼容
- 支持TimeHistory、EventLog、SessionSummary等表

## 注意事项

1. **编译依赖**: 需要安装 SQLite3 开发库 (`libsqlite3-dev`)
2. **数学库**: 使用 `<cmath>` 而非 `<math.h>`
3. **线程安全**: 核心类不是线程安全的，多线程需要外部同步
4. **异常处理**: 使用异常处理错误，关键接口检查参数有效性
5. **浮点精度**: 与Python版本对比时，差异应在1e-6以内

## 与Python版本的关系

- 本C++版本从 Python `noise_info_toolkit` 移植
- 计算结果应与Python版本完全一致
- 数据库表结构与Python版本兼容
- 验证脚本 `validation/validate_dose_calculator.py` 用于确保一致性

## 文件模板

### 新增类头文件模板

```cpp
/**
 * @file new_class.hpp
 * @brief 类功能描述
 */

#pragma once

#include "noise_toolkit.hpp"

namespace noise_toolkit {

/**
 * @brief 类描述
 */
class NewClass {
public:
    /**
     * @brief 构造函数
     */
    explicit NewClass();
    
    /**
     * @brief 方法描述
     */
    void some_method();

private:
    int member_variable_;
};

} // namespace noise_toolkit
```

---

*最后更新: 2026-03-18*

---
> Source: [LewisBase/noise_info_toolkit_gcc](https://github.com/LewisBase/noise_info_toolkit_gcc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
