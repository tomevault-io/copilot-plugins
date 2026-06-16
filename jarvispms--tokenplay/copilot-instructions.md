## tokenplay

> > **同步提醒**: 本文件与 `CLAUDE.md` 内容一致，更新时必须同步修改两个文件。

# AGENTS.md

> **同步提醒**: 本文件与 `CLAUDE.md` 内容一致，更新时必须同步修改两个文件。

## 项目概要

TokenPlay — 多平台 AI 对话管理工具。Next.js 15 + React 19 + TypeScript + Tailwind + shadcn/ui。

## 技术栈

- 框架: Next.js 15 (App Router)
- 数据存储: 本地 JSON 文件 (`data/` 目录)，**绝对不要删除 `data/` 下的文件**
- 流式传输: NDJSON over ReadableStream
- API 协议: OpenAI / Anthropic / 自定义（通过 `unified-client` 统一）

## 核心功能

- **平台管理** (`/platforms`): 多协议 API 平台配置，共享模型列表
- **群聊** (`/chat`): 广播模式（并行回复）+ 接力模式（轮流链式回复）
- **单聊** (`/single-chat`): 单模型对话
- **批量测试** (`/test`): 多平台并行测试对比
- **角色管理** (`/settings`): 角色绑定平台+模型+头像，群聊中作为独立参与者
- **系统提示词** (`/settings`): 多层提示词组合

## 数据模型要点

- `PlatformConfig.models` 是共享的，所有协议公用同一个模型列表
- `Role` 绑定一个平台+一个模型，可设自定义头像（图片路径或 emoji）
- `ChatSession.participantType` 决定参与者是平台还是角色（互斥）
- `ChatMessage` 中 `roleId`/`roleName`/`roleAvatar` 冗余存储，角色删除后历史消息不受影响

## 项目结构

```
src/
├── app/              # 页面和 API 路由
│   ├── api/          # 后端 API (chat, roles, platforms, upload, test...)
│   ├── chat/         # 群聊页面
│   ├── single-chat/  # 单聊页面
│   ├── platforms/    # 平台管理页面
│   ├── settings/     # 系统设置页面
│   ├── test/         # 批量测试页面
│   └── history/      # 测试历史页面
├── components/       # React 组件
│   ├── ui/           # 通用 UI 组件 (shadcn + 自定义)
│   ├── platforms/    # 平台相关组件
│   ├── layout/       # 布局组件
│   └── test/         # 测试相关组件
├── lib/
│   ├── types.ts      # 所有类型定义
│   ├── prompt-builder.ts  # 4 层提示词组合
│   ├── store/        # 数据 CRUD (JSON 文件读写)
│   └── api-clients/  # 统一 API 客户端
data/                 # 数据文件 (platforms.json, roles.json, chat-sessions/, ...)
public/avatars/       # 角色头像上传目录
```

## 编码约定

- 数据格式变更用迁移函数 `migrateIfNeeded` 处理，不要重建数据文件
- API 路由使用 NDJSON 流式传输格式
- 前端与用户交互使用中文
- 组件优先使用 shadcn/ui

---
> Source: [JarvisPMS/tokenplay](https://github.com/JarvisPMS/tokenplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
