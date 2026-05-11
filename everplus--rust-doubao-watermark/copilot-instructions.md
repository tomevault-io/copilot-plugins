## rust-doubao-watermark

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个用 Rust 实现的豆包 AI 图片去水印命令行工具。该工具通过监听剪贴板变化获取两张图片，并进行拼接以去除水印。支持在终端中直接预览图片。

## 核心依赖

- `arboard` - 跨平台剪贴板访问库，用于读取和设置剪贴板内容
- `image` - 图片处理库，支持多种格式（JPEG、PNG、WebP 等）的读取、转换和拼接
- `viuer` - 终端图片显示库，支持 Sixel、Kitty Graphics Protocol 等多种终端图像协议

## 构建和运行

```bash
# 开发构建
cargo build

# 发布构建（优化版本）
cargo build --release

# 运行程序
cargo run

# 运行发布版本
.\target\release\doubao-watermark.exe
```

## 工作流程

程序分为以下步骤：

1. **初始化** - 清空剪贴板
2. **获取上半部分图片** - 提示用户通过浏览器复制图片，监听剪贴板获取图片一
3. **显示图片一预览** - 在终端中显示获取的第一张图片（需要终端支持 Sixel 或其他图像协议）
4. **获取下半部分图片** - 提示用户直接复制图片，监听剪贴板获取图片二
5. **显示图片二预览** - 在终端中显示获取的第二张图片
6. **拼接图片** - 使用图片二的上半部分和图片一的下半部分进行拼接
7. **显示拼接结果** - 在终端中显示最终拼接的图片
8. **保存结果** - 将拼接后的图片保存到桌面，命名为 `doubao_image_{时间戳}.png`

## 终端图片显示支持

`viuer` 会自动检测并使用最佳协议：
- **Sixel** - 经典的终端图像协议，支持 xterm、mintty、ConEmu 等
- **Kitty Graphics Protocol** - Kitty 终端的高性能图像协议
- **iTerm2 Inline Images** - macOS iTerm2 的图像协议
- **ASCII Art fallback** - 不支持图像时降级为 ASCII 字符显示

## 核心模块说明

### 剪贴板操作 (`clear_clipboard`, `wait_for_image`)

- `clear_clipboard()`: 清空剪贴板，确保开始时没有旧数据
- `wait_for_image()`: 轮询监听剪贴板变化，等待用户复制图片

### 图片处理 (`convert_to_png`, `stitch_images`)

- `convert_to_png()`: 将剪贴板获取的图片数据解析为 `DynamicImage`，自动处理各种格式
- `stitch_images()`: 核心拼接逻辑
  - 验证两张图片尺寸是否一致
  - 从垂直中点分割
  - 使用图片二的上半部分 + 图片一的下半部分拼接成完整图片

### 图片显示 (`display_image`)

- `display_image()`: 使用 `viuer` 在终端显示图片预览
  - 配置显示尺寸为 80x25 字符
  - 自动选择最佳终端协议
  - 不支持时显示友好提示，不影响后续处理

### 保存 (`save_image`, `get_desktop_path`)

- `get_desktop_path()`: 跨平台获取桌面路径（支持 Windows 和 Unix-like 系统）
- `save_image()`: 将处理后的图片保存为 PNG 格式到桌面

## 架构设计

程序采用单文件结构 (`src/main.rs`)，所有功能模块按职责分离为独立函数：

- 使用 `Result<T, String>` 进行错误处理，提供友好的中文错误信息
- 每个步骤都有清晰的用户提示和进度反馈
- 图片格式自动转换，最终统一输出为 PNG
- 在关键步骤提供终端图片预览，方便用户确认

## 使用场景

该工具专门用于处理豆包 AI 生成的带水印图片：
- 第一步通过浏览器「复制图片」获取无水印的上半部分
- 第二步直接「复制」获取完整图片
- 通过拼接去除中间的水印区域
- 支持终端预览，无需打开图片查看器即可确认结果

---
> Source: [everplus/rust-doubao-watermark](https://github.com/everplus/rust-doubao-watermark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
