## vibemux

> **VibeMux** 是一个基于 TUI (终端用户界面) 的 AI 智能体编排与管理终端。

# CLAUDE.md - VibeMux

## 1. 项目简介 (Overview)
**VibeMux** 是一个基于 TUI (终端用户界面) 的 AI 智能体编排与管理终端。
旨在为 "Vibe Coding" 提供流畅的并行开发体验，允许开发者在一个统一的界面中像使用 `lazydocker` 或 `k9s` 一样，管理、监控和操控多个 **Claude Code** 或 **Codex** 实例。

## 2. 核心文档 (Documentation)
> ⚠️ **开发注意**：具体的模块划分、功能规范、数据结构设计，请务必参阅以下文档进行开发：
> - 📄 **[详细功能与架构设计](docs/DESIGN.md)** (Feature Specs & Architecture)

## 3. 技术选型 (Tech Stack)
* **开发语言**: Go (Golang) 1.22+
* **TUI 框架**: [Bubble Tea](https://github.com/charmbracelet/bubbletea) (基于 Elm 架构的消息循环)
* **UI 样式**: [Lip Gloss](https://github.com/charmbracelet/lipgloss) (声明式布局与着色)
* **标准组件**: [Bubbles](https://github.com/charmbracelet/bubbles) (用于列表、输入框、视口管理)
* **进程模拟**: [creack/pty](https://github.com/creack/pty) (核心组件：用于伪终端交互与输出捕获)
* **配置存储**: JSON (标准库实现，轻量化)

## 4. 开发原则 (Principles)
1.  **非侵入性 (Non-Intrusive)**: 使用进程级隔离 (Env Vars injection) 和 PTY 包装，不修改用户原本的全局配置文件。
2.  **配置即代码 (Profile as Code)**: 支持多套环境配置（Profiles），实现不同项目使用不同 API Key 或不同启动驱动 (Native/CCR)。
3.  **高响应度 (High Responsiveness)**: UI 渲染必须与底层 PTY IO 异步分离，确保在 AI 输出大量内容时界面不卡顿。

## 5. 常用指令 (Commands)
* `go run main.go`: 启动开发模式
* `go build -o bin/VibeMux`: 编译发布版本

---
> Source: [UgOrange/vibemux](https://github.com/UgOrange/vibemux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
