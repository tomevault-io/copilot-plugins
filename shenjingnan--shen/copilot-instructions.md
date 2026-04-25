## shen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Shen 是一个探索性的 AI 命令行工具项目，旨在处理编程以外的日常工作任务。项目受 Claude Code 启发，但专注于非编程领域的自动化和效率提升。

### 核心特点
- **语言**: Python
- **类型**: CLI 工具
- **MCP 集成**: 已实现 Model Context Protocol 客户端，支持多种传输方式
- **目标**: 文档整理、垃圾清理、信息提供、Office 文档制作、安全检查、软件安装等日常任务

## 项目状态

**当前版本**: 0.0.0-beta.0
- ✅ 基础 CLI 框架已实现
- ✅ MCP 客户端和服务管理器已完成
- ✅ 支持 HTTP、WebSocket 和 STDIO 传输
- ✅ 基础测试覆盖
- 🚧 任务执行引擎开发中
- 🚧 插件系统开发中

## 开发指南

### Python 项目初始化建议

当开始实现代码时，建议：

1. 使用 `pyproject.toml` 进行项目配置
2. 使用 `poetry` 或 `pip` + `venv` 管理依赖
3. 集成代码质量工具：
   - `black` 或 `ruff` 进行代码格式化
   - `mypy` 进行类型检查
   - `pytest` 进行单元测试
   - `pre-commit` 进行提交前检查

### 当前架构

项目采用以下架构设计：

1. **CLI 界面**: 使用 Click 构建，支持子命令和选项
2. **MCP 集成**: 完整的 MCP 客户端实现，支持多种传输方式
3. **服务管理**: MCPManager 管理所有 MCP 服务连接
4. **配置系统**: 基于 Pydantic 的配置管理
5. **日志记录**: Rich 集成的美观日志输出

### 当前目录结构

```
shen/
├── src/
│   └── shen/
│       ├── __init__.py
│       ├── cli.py          # CLI 入口点
│       ├── core/           # 核心功能模块
│       │   ├── app.py      # 主应用类
│       │   ├── config.py   # 配置管理
│       │   └── plugin_manager.py  # 插件管理器
│       ├── mcp/            # MCP 集成模块
│       │   ├── client.py   # MCP 客户端
│       │   ├── manager.py  # MCP 服务管理器
│       │   └── models.py   # MCP 数据模型
│       ├── plugins/        # 插件目录
│       └── utils/          # 工具函数
│           └── logging.py  # 日志工具
├── tests/                  # 测试文件
├── pyproject.toml         # 项目配置
├── CLAUDE.md              # Claude 使用指南
└── README.md              # 项目说明
```

## MCP 集成指南

### 常用命令

```bash
# 安装依赖
poetry install

# 运行 Shen
poetry run shen

# 查看 MCP 服务
poetry run shen mcp list

# 连接 MCP 服务
poetry run shen mcp connect service-name

# 查看可用工具
poetry run shen mcp tools
```

### MCP 服务配置

MCP 服务配置示例位于 `~/.shen/mcp/` 目录，支持：
- **STDIO**: 进程通信方式
- **HTTP**: REST API 方式  
- **WebSocket**: 实时通信方式

## 与 Claude Code 的差异化定位

根据 README.md 的描述，Shen 与 Claude Code 的主要区别：

1. **Claude Code**: 专注于编程任务（代码编写、调试、重构等）
2. **Shen**: 专注于非编程任务（文档管理、系统维护、办公自动化等）

两者都是 CLI 工具，都强调跨平台性和强大的操作权限，但应用领域不同。

## 注意事项

- 项目仍在探索阶段，架构和功能定位可能会调整
- 优先考虑安全性，特别是涉及系统操作和文件管理时
- 保持良好的错误处理和用户反馈机制

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shenjingnan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
