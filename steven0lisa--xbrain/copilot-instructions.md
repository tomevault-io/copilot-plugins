## xbrain

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个基于邮件和Claude Code的AI助手项目，用于自动处理邮件并调用Claude Code生成智能回复。

## 常用命令

### 开发与运行
- `npm start` - 启动邮件助手服务
- `npm run dev` - 以开发模式启动（NODE_ENV=development）
- `npm test` - 运行测试套件（使用vitest）

### 调试与测试
- `node scripts/run-assistant.js` - 单独运行助手测试脚本
- `DRY_RUN=1 npm start` - 干运行模式（跳过网络连接和实际调用）

## 核心架构

### 主要模块
- **src/index.js** - 主入口，处理邮件轮询和助手触发逻辑
- **src/config.js** - 配置管理，从环境变量加载邮箱和Claude API配置
- **src/assistant.js** - Claude Code调用核心逻辑，处理prompt构建和结果解析
- **src/logger.js** - Winston日志配置，支持文件轮转和控制台输出
- **src/mail/imap.js** - IMAP邮件接收和附件下载
- **src/mail/smtp.js** - SMTP邮件发送

### 数据流程
1. 邮件接收：通过IMAP定期拉取新邮件，保存到 `data/kb/` 目录
2. 助手触发：检测邮件正文/主题中是否包含 `@助手` 关键词
3. Claude调用：将邮件内容和附件作为输入，调用Claude Code生成回复
4. 邮件回复：通过SMTP发送Claude生成的回复内容和附件

### 关键特性
- **状态持久化**：使用 `data/state.json` 记录已处理的邮件UID
- **附件处理**：自动下载和保存邮件附件，支持传递给Claude Code
- **工作目录管理**：每次Claude调用在 `data/assistant/` 下创建独立工作目录
- **多种回复格式**：支持纯文本、Markdown和HTML格式的回复

## 环境变量配置

### 邮箱配置（必需）
- `MAIL_USER` - 邮箱用户名
- `MAIL_PASS` - 邮箱密码/授权码
- `IMAP_HOST` - IMAP服务器地址（默认：imap.163.com）
- `IMAP_PORT` - IMAP端口（默认：993）
- `SMTP_HOST` - SMTP服务器地址（默认：smtp.163.com）
- `SMTP_PORT` - SMTP端口（默认：465）

### Claude Code配置（必需）
- `ANTHROPIC_BASE_URL` - Claude API基础URL
- `ANTHROPIC_AUTH_TOKEN` - Claude API认证令牌
- `ANTHROPIC_SMALL_FAST_MODEL` - 快速模型名称
- `ANTHROPIC_MODEL` - 主模型名称

### 可选配置
- `POLL_INTERVAL_MS` - 邮件轮询间隔（默认：60000ms）
- `CLAUDE_BIN` - Claude二进制路径（默认：claude）
- `LOG_LEVEL` - 日志级别（默认：info）

## 项目目录结构

```
src/
├── index.js          # 主入口文件
├── config.js         # 配置管理
├── assistant.js      # Claude Code调用逻辑
├── logger.js         # 日志配置
└── mail/
    ├── imap.js       # IMAP邮件接收
    └── smtp.js       # SMTP邮件发送

data/                 # 运行时数据目录
├── kb/              # 邮件知识库
├── state.json       # 状态持久化
└── assistant/       # Claude工作目录

logs/                # 日志文件目录
tests/               # 测试文件
scripts/             # 工具脚本
```

## 开发注意事项

### Claude Code集成
- 项目使用spawn调用外部Claude Code CLI工具
- 支持JSON输出格式和特定工具权限配置
- 具有超时机制和错误回退逻辑

### 邮件处理
- 使用UID范围增量获取邮件，避免重复处理
- 自动处理邮件附件并保存到本地
- 支持HTML和纯文本邮件格式

### 错误处理
- 完善的日志记录机制，便于调试和监控
- 网络错误和Claude调用异常的容错处理
- 支持干运行模式进行开发调试

---
> Source: [steven0lisa/xbrain](https://github.com/steven0lisa/xbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
