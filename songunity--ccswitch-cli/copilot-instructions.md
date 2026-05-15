## ccswitch-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Claude Code Switcher (CC Switch) 是一个 TypeScript CLI 工具，用于管理和切换 Claude Code 的 settings.json 配置文件中的 URL 和 token 设置。项目支持存储多个配置并一键切换，提升开发效率。

## 构建和开发命令

### 基本构建
```bash
npm run build        # 编译 TypeScript 到 dist/ 目录
npm run clean        # 清理 dist/ 目录
npm run dev          # 监听模式构建（tsc --watch）
npm start            # 运行编译后的 CLI 工具
```

### 安装和测试
```bash
./install.sh         # Unix/Linux 安装脚本
install.bat          # Windows 安装脚本  
./install-npm.sh     # npm 全局安装脚本
npm run test         # 测试（当前无测试，返回 exit 0）
ccs --help           # 查看 CLI 帮助
```

### 开发工作流
```bash
npm run dev          # 启动监听模式
node dist/cli.js     # 直接运行测试
ccs diagnostics      # 运行系统诊断检查
```

## 核心架构

### 目录结构
- `src/cli.ts` - CLI 入口点，使用 Commander.js 定义所有命令
- `src/commands/` - 各个 CLI 命令的实现（add, list, switch, remove, current, backup, restore, diagnostics）
- `src/managers/` - 核心业务逻辑管理器
  - `ConfigManager.ts` - CCS 配置文件管理（~/.ccswitcher/configs.json）
  - `ClaudeConfigManager.ts` - Claude Code settings.json 文件操作
  - `Validator.ts` - 输入验证逻辑
- `src/types/` - TypeScript 类型定义和错误类
- `src/utils/` - 辅助工具函数（文件操作、安全处理、常量定义）

### 关键设计模式
1. **管理器模式** - ConfigManager 和 ClaudeConfigManager 分别处理不同配置文件
2. **命令模式** - 每个 CLI 命令独立模块化实现
3. **工厂模式** - 错误处理通过自定义错误类统一管理
4. **单一职责** - 每个模块职责明确，ConfigManager 处理存储，ClaudeConfigManager 处理 Claude 配置

### 配置文件管理
- **CCS 配置**: `~/.ccswitcher/configs.json` - 存储用户的多个配置
- **Claude 配置**: `~/.claude/settings.json` - Claude Code 的实际配置文件
- **备份机制**: 切换前自动备份，保留最近 5 个备份文件
- **权限安全**: 配置文件权限设置为 600（仅所有者可读写）

### 数据流
1. 用户执行 `ccs add` → ConfigManager 验证并存储到 CCS 配置文件
2. 用户执行 `ccs switch` → ConfigManager 读取配置 → ClaudeConfigManager 更新 Claude settings.json
3. 备份和恢复通过 ClaudeConfigManager 的文件操作实现

## CLI 命令结构

所有命令支持简写形式：
- `add|a <name> <url> <token>` - 添加配置
- `list|ls` - 列出所有配置  
- `switch|sw <name>` - 切换配置
- `remove|rm <name>` - 删除配置
- `current|cur` - 显示当前配置
- `backup` - 手动备份
- `restore <backup-file>` - 恢复备份
- `diagnostics|diag` - 系统诊断

## 开发注意事项

### TypeScript 配置
- 目标版本: ES2020
- 输出目录: `dist/`
- 启用严格模式和所有类型检查
- 生成声明文件和 source maps

### 安全考虑
- Token 显示时智能遮蔽（根据 sk-ant-, sk-, pk- 等前缀）
- 文件权限自动设置为安全模式
- 原子写入操作防止配置损坏

### 错误处理
使用自定义错误类 `ConfigError` 和 `ClaudeConfigError`，包含错误代码：
- `CONFIG_NOT_FOUND` - 配置不存在
- `CONFIG_EXISTS` - 配置名称冲突
- `CLAUDE_CONFIG_NOT_FOUND` - Claude 配置文件不存在
- `PERMISSION_DENIED` - 权限不足

### 跨平台支持
支持 macOS、Linux、Windows，通过 package.json 的 `os` 字段限定

## 开发原则

### 代码质量
* 遵循 KISS 原则，保持代码简洁
* 优先实现核心功能，避免过度设计
* 注重代码的可维护性和可读性

### 开发流程
* 在开始开发前进行充分的需求分析
* 通过多轮迭代逐步完善功能
* 保持良好的错误处理和用户体验

### 技术选择
* 使用 TypeScript 确保类型安全
* 采用模块化设计便于维护
* 重视跨平台兼容性

## 架构深入理解

### 依赖关系
- **CLI 层**: `src/cli.ts` → 各个 command 文件
- **命令层**: Commands → Managers + Utils
- **业务逻辑层**: ConfigManager, ClaudeConfigManager, Validator 
- **工具层**: utils/helpers, utils/security, utils/pathResolver

### 核心类交互
1. `ConfigManager` - 管理 CCS 自己的配置存储 (~/.ccswitcher/configs.json)
2. `ClaudeConfigManager` - 操作 Claude Code 的配置文件 (~/.claude/settings.json)
3. `PathResolver` - 动态解析配置文件路径
4. `BackupManager` - 处理配置文件备份和恢复
5. `Validator` - 统一验证用户输入的 URL 和 token

### 配置文件检测优先级
ClaudeConfigManager 按以下优先级检测 Claude 配置文件：
1. `~/.claude/settings.json` (用户级别，优先)
2. `.claude/settings.json` (项目级别)
3. `.claude/settings.local.json` (项目本地)

### 关键开发模式
- **原子操作**: 所有配置文件写入使用临时文件 + 原子重命名
- **权限安全**: 配置文件自动设置为 600 权限
- **错误恢复**: 配置损坏时自动回退到备份
- **智能检测**: 自动检测 Claude Code 配置文件位置和权限

---
> Source: [songunity/ccswitch-cli](https://github.com/songunity/ccswitch-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
