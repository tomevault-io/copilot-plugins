## y-x4-album

> 项目：Xteink X4 E-Ink 电子相册固件

# Y-X4-Album 开发指南

项目：Xteink X4 E-Ink 电子相册固件
使命：在 ESP32-C3 受限硬件上提供流畅的图片浏览体验，支持 TF 卡图片读取、高质量渲染和直觉式交互。

## AI Agent 认知规则
* 角色：高级嵌入式系统工程师（ESP-IDF/Arduino-ESP32 专精）
* 首要约束：380KB RAM 是硬上限。稳定性不可妥协。
* 基于证据推理：修改前必须引用具体文件路径和行号。
* 反幻觉：不要假设库或 ESP-IDF 函数存在。不确定时先检查 open-x4-sdk。
* 资源论证：每一次新的堆分配（new, malloc, std::vector）都必须论证为何不用栈/静态替代。

---

## 硬件规格（Xteink X4）

* MCU: ESP32-C3（单核 RISC-V @ 160MHz）
* RAM: ~380KB 可用（**无 PSRAM**，极其有限）
  * 单缓冲模式：仅一个 48KB framebuffer
* Flash: 16MB
* 显示屏: 800×480 E-Ink（单色+灰度抖动，刷新 1-2s）
  * Framebuffer: 48,000 字节 (800 × 480 ÷ 8)
* 存储: TF/SD 卡（图片存储和缓存）
* 输入: 物理按钮（电源键、上/下侧键、前置按钮组）
* 电池: 内置锂电池

---

## 项目架构

### 与 CrossPoint Reader 的关系
本项目复用 crosspoint-reader 的以下组件：
- **open-x4-sdk**：底层硬件驱动（EInkDisplay, InputManager, BatteryMonitor, SDCardManager）
- **HAL 层**：HalDisplay, HalGPIO, HalStorage, HalPowerManager, HalSystem
- **GfxRenderer**：图形渲染引擎（绘图、文字、位图、灰度支持）
- **PngToBmpConverter / JpegToBmpConverter**：图片格式转换
- **EpdFont**：字体系统
- **Logging**：日志系统
- **I18n**：国际化框架

### 本项目自有代码（src/）
- `src/main.cpp` - 入口点，初始化和主循环
- `src/activities/` - UI 界面（Activity 生命周期模式）
- `src/components/` - UI 组件和主题
- `src/album/` - 相册核心逻辑（图片扫描、缩略图缓存、幻灯片）
- `src/settings/` - 相册设置管理

### 构建系统: PlatformIO
- 标准: C++20 (`-std=gnu++2a`)，无异常，无 RTTI
- 日志: 使用 `LOG_INF`, `LOG_DBG`, `LOG_ERR`（来自 Logging.h）

---

## 编码规范

### 与 CrossPoint Reader 一致的规范
- 命名：PascalCase（类），camelCase（方法/变量），UPPER_SNAKE_CASE（常量）
- 头文件保护：`#pragma once`
- 智能指针：优先 `std::unique_ptr`，禁止 `std::shared_ptr`
- 字符串：热路径禁止 `std::string`/`String`，用 `string_view` + `snprintf`
- 栈安全：局部变量 < 256 字节
- 堆分配：必须检查 nullptr，使用后立即释放
- 错误处理：LOG_ERR + return false，无异常，无 abort()

### 资源管理协议
1. Activity 的 `onEnter()` 中分配的资源必须在 `onExit()` 中释放
2. FreeRTOS 任务必须在 `onExit()` 中 `vTaskDelete()`
3. 文件句柄必须在 `onExit()` 中关闭
4. 大缓冲区（>256B）用 malloc，必须检查返回值

### 图片处理特殊规范
- 图片解码必须分块进行，不能一次性加载整张图片到内存
- 缩略图缓存到 SD 卡，避免重复解码
- 灰度渲染时需使用 `renderer.storeBwBuffer()` / `renderer.restoreBwBuffer()`
- RISC-V 对齐：不允许将 `uint8_t*` 强转为宽类型指针，用 `memcpy`

---

## 按钮映射
使用 MappedInputManager 的逻辑按钮，不要直接使用 HalGPIO 物理按钮索引。
- `Button::Up/Down` - 上下翻页/导航
- `Button::Left/Right` - 左右切换图片
- `Button::Confirm` - 确认/进入
- `Button::Back` - 返回
- `Button::PageForward/PageBack` - 快速翻页

---

## 测试流程

### AI 可验证
1. ✅ 构建: `pio run -t clean && pio run`（0 错误/警告）
2. ✅ 静态分析: `pio check`
3. ✅ 代码格式: clang-format

### 需要硬件测试
- 🔲 设备上测试所有功能
- 🔲 4 个方向的屏幕旋转
- 🔲 堆内存: `ESP.getFreeHeap()` > 50KB
- 🔲 TF 卡兼容性（FAT32/exFAT）
- 🔲 各种尺寸和格式的图片

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yixi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
