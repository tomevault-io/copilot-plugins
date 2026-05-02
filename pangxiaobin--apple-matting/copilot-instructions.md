## src-tauri-overview

> apple-matting Tauri 后端项目架构总结，在涉及 src-tauri 相关开发时自动引用


# src-tauri 后端项目总结

## 项目概述

**apple-matting** 是一个基于 Tauri 2 的 macOS 桌面应用，核心功能是利用 Apple Vision 框架进行图片背景移除（抠图）。

- **后端**：Rust (Tauri 2) + Swift FFI
- **原生抠图**：Swift 调用 Vision (`VNGenerateForegroundInstanceMaskRequest`) + Core Image (`CIBlendWithMask`)
- **前端**：pnpm 管理，构建产物输出到 `../dist`，开发端口 `localhost:1420`
- **平台要求**：macOS 14.0+（Vision API 限制），非 macOS 平台返回 `UnsupportedPlatform`

## 目录结构

```
src-tauri/
├── build.rs                 # 构建脚本：编译 Swift -> 静态库，链接 Apple 框架
├── Cargo.toml               # Rust 依赖配置
├── tauri.conf.json          # Tauri 应用配置（窗口、权限、构建命令）
├── capabilities/default.json # 主窗口权限声明
├── icons/                   # 应用图标
├── src/
│   ├── main.rs              # 二进制入口，调用 lib::run()
│   ├── lib.rs               # Tauri Builder 初始化，注册插件和命令
│   ├── commands.rs          # Tauri 命令：process_single_image, process_batch_images
│   └── matting.rs           # FFI 桥接层：perform_matting, MattingError, derive_output_path
└── swift/
    └── MattingBridge.swift  # Swift 原生抠图实现（Vision + Core Image）
```

## 模块关系

```
lib.rs (Tauri 入口)
├── commands.rs
│   ├── 调用 matting::perform_matting
│   └── 导出: process_single_image, process_batch_images
└── matting.rs
    ├── extern "C" { matting_process_image } (FFI -> Swift)
    └── 导出: perform_matting, derive_output_path, MattingError
```

## Tauri 命令（前端可调用）

### `process_single_image`
- **参数**：`input_path: String`, `output_path: Option<String>`
- **返回**：`MattingResult { success, output_path, error }`
- **功能**：单张图片背景移除

### `process_batch_images`
- **参数**：`items: Vec<BatchInputItem>`（每项含 `input_path`, `output_path`）
- **返回**：`Vec<BatchItemResult>`（每项含 `input_path`, `success`, `output_path`, `error`）
- **功能**：批量背景移除，每项独立线程执行，单项失败不影响其他项

## 核心数据结构

| 结构体 | 位置 | 用途 |
|--------|------|------|
| `MattingResult` | commands.rs | 单图结果：success, output_path, error |
| `BatchItemResult` | commands.rs | 批量单项结果：input_path, success, output_path, error |
| `BatchInputItem` | commands.rs | 批量输入项：input_path, output_path |
| `MattingError` (enum) | matting.rs | 错误类型，含 10 个变体 |

## MattingError 枚举

| 变体 | Swift 错误码 | 含义 |
|------|-------------|------|
| `LoadFailed` | -1 | 无法加载输入图片 |
| `VisionRequestFailed` | -2 | Vision 请求失败 |
| `NoForeground` | -3 | 未检测到前景实例 |
| `MaskFailed` | -4 | 蒙版生成失败 |
| `BlendFailed` | -5 | 混合滤镜无输出 |
| `WriteFailed` | -6 | 写入 PNG 失败 |
| `UnsupportedOs` | -99 | macOS 版本过低（需 14.0+）|
| `InvalidPath` | — | 路径无效 |
| `UnsupportedPlatform` | — | 非 macOS 平台 |
| `Unknown` | 其他 | 未知错误 |

## 数据流

1. 前端调用 `invoke('process_single_image' | 'process_batch_images', params)`
2. `commands.rs` 验证输入，调用 `matting::perform_matting`
3. `matting.rs` 解析输出路径（默认 `*_nobg.png`），转换为 CString，调用 FFI
4. `MattingBridge.swift` 执行：加载图片 → Vision 前景分割 → 生成蒙版 → CIBlendWithMask 混合 → 写入 PNG
5. 返回错误码，Rust 侧映射为 `MattingError`，最终包装为 JSON 返回前端

## 依赖

| 包 | 版本 | 用途 |
|----|------|------|
| `tauri` | 2 | 核心框架 |
| `tauri-plugin-opener` | 2 | 打开外部 URL/文件 |
| `tauri-plugin-fs` | 2 | 文件系统读写 |
| `tauri-plugin-dialog` | 2 | 原生对话框 |
| `serde` / `serde_json` | 1 | 序列化 |
| `thiserror` | 1 | 错误类型派生 |

## 构建流程（build.rs）

1. 调用 `tauri_build::build()`
2. macOS 下编译 Swift：`swiftc` 编译 `MattingBridge.swift` → 归档为 `libmatting_bridge.a`
3. 链接：`matting_bridge` 静态库 + Swift 运行时 + Vision/CoreImage/Foundation/CoreGraphics 框架
4. 支持 arm64 和 x86_64 架构，最低目标 macOS 12.0

## 权限配置（capabilities/default.json）

- **文件系统**：可读写 `$APPDATA`, `$DESKTOP`, `$DOCUMENT`, `$DOWNLOAD`, `$PICTURE`, `$HOME` 及子目录
- **对话框**：文件选择、确认、保存对话框
- **opener**：系统默认应用打开文件/URL

## 开发注意事项

- 新增 Tauri 命令需在 `lib.rs` 的 `invoke_handler` 中注册
- Swift FFI 函数签名需在 `matting.rs` 的 `extern "C"` 块中声明
- 输出路径默认规则：输入文件同目录下 `{原文件名}_nobg.png`
- 批量处理使用多线程，注意线程安全
- `build.rs` 中 `cargo:rerun-if-changed=swift/MattingBridge.swift` 确保 Swift 变更触发重编译

---
> Source: [pangxiaobin/apple-matting](https://github.com/pangxiaobin/apple-matting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
