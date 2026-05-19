## project-overview

> ElegooSlicer 项目架构、技术栈和开发概览


# ElegooSlicer 项目概览

## 项目简介

ElegooSlicer 是一款开源的 3D 切片软件，基于 OrcaSlicer 开发。OrcaSlicer 衍生自 Bambu Studio，而 Bambu Studio 又是基于 PrusaSlicer 进行开发的，PrusaSlicer 则起源于 Slic3r。项目使用 C++ 编写，采用 wxWidgets 构建 GUI，并使用 CMake 作为构建系统。该项目采用模块化架构，核心切片功能、GUI 组件和平台特定代码分别使用独立的库。

## 技术栈概览

- 语言与标准：C++17（按需使用 C++20 特性）、部分 Objective-C++（macOS）、Python/Batch/Shell（脚本）
- GUI 框架：wxWidgets 3.2+，结合 ImGui 渲染 3D 视图
- 渲染：OpenGL（3D 预览、模型交互）
- 构建系统：CMake（最小 3.13，Windows 建议 ≤3.31.x）
- 并行与任务调度：Intel TBB、std::thread
- 脚本/工具链：Python 3.x、PowerShell/Batch、Shell、gettext
- 依赖管理：`deps/` 目录单独构建并缓存，主工程通过 CMake `CMAKE_PREFIX_PATH` 引入

## 核心架构

libslic3r/ - 核心切片引擎和算法（平台无关）
- 主要切片逻辑、几何处理、G-code 生成
- 核心类：Print, PrintObject, Layer, GCode, Config
- 模块化设计，专业子目录：
  - `GCode/` - G-code 生成、冷却、压力均衡、缩略图
  - `Fill/` - 填充图案实现（gyroid、蜂窝、闪电填充等）
  - `Support/` - 树形支撑和传统支撑生成
  - `Geometry/` - 高级几何操作、Voronoi 图、中轴
  - `Format/` - 文件 I/O（3MF、AMF、STL、OBJ、STEP）
  - `SLA/` - SLA 特定打印处理和支撑生成
  - `Arachne/` - 使用骨架梯形化的高级壁生成

src/slic3r/ - 主应用框架和 GUI
- 基于 wxWidgets 构建的 GUI 应用
- libslic3r 核心与用户界面的集成
- 位于 `src/slic3r/GUI/`

## 关键算法组件

- Arachne 壁生成：使用骨架梯形化的可变宽度外围生成
- 树形支撑：有机支撑生成算法
- 闪电填充：内部结构的稀疏填充优化
- 自适应切片：基于几何的可变层高
- 多材料：多挤出机和可溶性支撑处理
- G-code 后处理：冷却、风扇控制、压力提前、冲突检查

## 文件格式支持

- 3MF/BBS_3MF：原生格式，支持多材料和元数据扩展
- STL：3D 模型的标准镶嵌语言
- AMF：支持颜色/材料的增材制造格式
- OBJ：带材质定义的 Wavefront OBJ
- STEP：支持精确几何的 CAD 格式
- G-code：具有广泛后处理功能的输出格式

## 外部依赖

- Clipper2：高级 2D 多边形裁剪和偏移
- libigl：网格操作的计算几何库
- TBB：Intel 线程构建块，用于并行化
- wxWidgets：跨平台 GUI 框架
- OpenGL：3D 图形渲染和可视化
- CGAL：计算几何算法库（选择性使用）
- OpenVDB：高级操作的体积数据结构
- Eigen：数学操作的线性代数库
- nlohmann/json：JSON 解析
- curl：网络操作
- OpenSSL：安全通信
- imgui：3D 视口的即时模式 GUI
- boost：实用工具库

## 文件组织

资源和配置
- `resources/web/` - 用于应用内嵌入式网页界面的 Web 资源（首页、指南、打印机管理等）
- `resources/profiles/` - 按制造商组织的打印机和材料配置文件
- `resources/printers/` - 打印机特定配置和 G-code 模板
- `resources/images/` - UI 图标、徽标、校准图像
- `resources/calib/` - 校准测试图案和数据
- `resources/handy_models/` - 内置测试模型（benchy、校准立方体）

国际化和本地化
- `localization/i18n/` - 源翻译文件（.pot、.po）
- `resources/i18n/` - 运行时语言资源
- 通过 `scripts/run_gettext.sh` / `scripts/run_gettext.bat` 管理翻译

平台特定代码
- `src/libslic3r/Platform.cpp` - 平台抽象和实用工具
- `src/libslic3r/MacUtils.mm` - macOS 特定实用工具（Objective-C++）
- Windows 特定构建脚本和配置
- `scripts/linux.d/` 中的 Linux 发行版支持脚本

构建和开发工具
- `cmake/modules/` - 自定义 CMake 查找模块和实用工具
- `scripts/` - 用于配置文件生成和验证的 Python 实用工具
- `tools/` - Windows 构建工具（gettext 实用工具）
- `deps/` - 外部依赖构建配置

## 构建系统

- 使用 CMake（最低版本 3.13，Windows 上最高 3.31.x）
- 主构建目录：`build/`
- 依赖项构建在：`deps/build/`
- 构建过程分为依赖项构建和主应用构建
- Windows：Visual Studio 2019/2022 或 Ninja
- macOS：Xcode（默认）或 Ninja（-x 参数）
- Linux：GCC/Clang with Ninja

## 开发工作流

添加新打印设置
- 在 `PrintConfig.cpp` 中定义设置及其边界和默认值
- 在适当的 GUI 组件中添加 UI 控件
- 更新配置保存/加载中的序列化
- 添加工具提示和帮助文本
- 使用不同的打印机配置文件测试

修改切片算法
- 核心算法位于 `libslic3r/` 子目录
- 性能关键代码应进行性能分析和优化
- 考虑多线程影响（TBB 集成）
- 验证更改不会破坏现有配置文件
- 适当添加回归测试

GUI 开发
- GUI 代码位于 `src/slic3r/GUI/`
- 使用现有的 wxWidgets 模式和自定义控件
- 同时支持浅色和深色主题
- 考虑高分辨率显示器上的 DPI 缩放
- 保持跨平台兼容性

添加打印机支持
- 在 `resources/profiles/[manufacturer].json` 创建 JSON 配置文件
- 添加打印机特定的起始/结束 G-code 模板
- 配置构建体积、功能和材料兼容性
- 尽可能用实际硬件进行彻底测试
- 遵循现有配置文件结构和命名约定

性能考虑
- 切片算法：CPU 密集型，优化前先进行性能分析
- 内存使用：复杂模型可能占用大量内存
- 多线程：通过 TBB 广泛使用
- 文件 I/O：针对带嵌入纹理的大型 3MF 文件优化
- 实时预览：需要高效的网格处理

## 重要开发注意事项

代码库导航
- 代码库超过 50 万行，广泛使用搜索工具
- 关键入口点：`src/ElegooSlicer.cpp` 用于应用启动
- 核心切片：`libslic3r/Print.cpp` 编排切片管道
- 配置：`PrintConfig.cpp` 定义所有打印/打印机/材料设置

兼容性和稳定性
- 向后兼容：维护项目文件和配置文件的兼容性
- 跨平台：支持 Windows/macOS/Linux 至关重要
- 文件格式：更改需要仔细的版本处理
- 配置文件迁移：设置显著更改时需要迁移

质量和测试
- 回归测试：由于算法复杂性而重要
- 性能基准：帮助捕获性能回归
- 内存泄漏：对长时间运行的 GUI 应用很重要
- 跨平台：发布前需要测试

---
> Source: [ELEGOO-3D/ElegooSlicer](https://github.com/ELEGOO-3D/ElegooSlicer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
