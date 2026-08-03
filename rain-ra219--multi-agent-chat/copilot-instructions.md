## multi-agent-chat

> 多模型协作对话平台 — 可视化界面，可创建多个 AI 智能体，设定不同角色，以流水线或讨论模式协作。

# CLAUDE.md

## 项目简介

多模型协作对话平台 — 可视化界面，可创建多个 AI 智能体，设定不同角色，以流水线或讨论模式协作。

## 我的角色

我是产品经理，正在边学边搭建这个项目。请用**老师带徒弟**的方式引导我：每一步先解释为什么，再动手。用中文沟通。

## 技术栈

- Next.js 16 (App Router) + TypeScript
- React 19 + Tailwind CSS 4
- API Routes 做后端中转
- 纯第三方 AI API（OpenAI 兼容格式），不用本地模型

## 项目结构

```
src/app/
├── layout.tsx        # 网站外壳
├── page.tsx          # 聊天首页
├── globals.css       # 全局样式
└── api/chat/route.ts # AI API 中转
```

## 代码风格偏好

- 不需要写注释，函数和变量命名语义化即可
- 保持简单，不过早抽象
- 每次改动后提醒我提交到 Git

## 禁止事项

- 不要修改 .env.local 文件的内容
- 不要删除 LEARNING.md

## 当前进度

- [x] 单模型聊天（DeepSeek）
- [x] localStorage 聊天记录持久化
- [ ] 多模型管理
- [ ] 角色/系统提示词
- [ ] 流水线协作
- [ ] 讨论模式

---
> Source: [rain-ra219/multi-agent-chat](https://github.com/rain-ra219/multi-agent-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
