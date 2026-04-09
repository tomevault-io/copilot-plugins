## clipmaster

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**ClipMaster** 是一款 macOS 剪切板管理工具,核心价值为"无感记录、极速检索、一键回填"。

### 核心功能
- **自动监听记录**: 后台静默运行,实时捕获文本、图片、文件路径、富文本等多种格式
- **菜单栏交互**: 点击状态栏图标弹出历史列表,支持鼠标点击复制或快捷键操作
- **搜索与过滤**: 支持关键字实时搜索历史记录
- **智能分类**: 自动识别链接、颜色代码、图片、文本片段
- **隐私保护**: 支持设置"黑名单"应用(如 1Password, Keychain),不记录敏感来源的剪切板
- **置顶与收藏**: 常用片段(如邮箱、地址、常用代码)可固定在顶部

## 技术架构

### 技术栈
- **开发语言**: Swift 5.x
- **UI 框架**: SwiftUI (用于现代化的界面) + AppKit (处理系统级菜单栏交互)
- **存储**: Core Data 或 SQLite (处理大量历史记录的持久化)
- **系统接口**: NSPasteboard (核心接口)

### 核心架构组件

#### 1. 剪切板监听机制
macOS 没有提供剪切板变化的直接通知,采用 **轮询(Polling)** 机制:
- 每隔 0.5 秒检查 `NSPasteboard.general.changeCount`
- 当值发生变化时,说明用户执行了新的复制操作
- 根据优先级提取数据(优先 String,其次是 Image 或 URL)

#### 2. 菜单栏界面 (NSStatusItem)
- 使用 `NSStatusBar.system.statusItem` 创建常驻图标
- 点击图标后弹出 `NSPopover` 或 `NSMenu`
- 使用 SwiftUI 构建自定义 Popover,支持搜索框和丰富的视觉效果

#### 3. 数据流架构
```
NSPasteboard 监听 → 数据解析与分类 → 持久化存储 → SwiftUI 展示层
```

## 开发指南

### 项目初始化

项目目前为空目录,需要创建标准的 macOS App 项目结构。

### 常用命令(待项目创建后补充)

### 关键设计决策

1. **轮询机制**: 使用 NSTimer 每 0.5 秒检查 `changeCount`,而非依赖系统通知
2. **SwiftUI + AppKit 混合**: SwiftUI 用于构建 Popover 内容,AppKit 处理菜单栏系统集成
3. **隐私保护优先**: 应用黑名单机制,确保敏感应用(密码管理器等)的剪切板内容不被记录

### 架构原则
- 轻量高效: 后台运行时资源占用最小化
- 无感操作: 用户无需手动触发,自动记录所有复制操作
- 快速检索: 实时搜索过滤,毫秒级响应

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NoBey)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NoBey)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
