## japanese-text-cleaner

> 日语文本清洗工具 — 网页端应用，支持将未清洗的日语 TXT 文本进行自动化清洗，包括 Unicode 正规化、全角半角转换、空白规范化、标点统一等多种清洗规则。同时提供 v1 API 供 Agent / 工作流集成。

# 项目上下文

## 项目概述

日语文本清洗工具 — 网页端应用，支持将未清洗的日语 TXT 文本进行自动化清洗，包括 Unicode 正规化、全角半角转换、空白规范化、标点统一等多种清洗规则。同时提供 v1 API 供 Agent / 工作流集成。

### 版本技术栈

- **Framework**: Next.js 16 (App Router)
- **Core**: React 19
- **Language**: TypeScript 5
- **UI 组件**: shadcn/ui (基于 Radix UI)
- **Styling**: Tailwind CSS 4

## 目录结构

```
├── docs/                          # 项目文档
│   └── skill-japanese-text-cleaner.md   # 通用设计 Skill
├── public/                        # 静态资源
├── scripts/                       # 构建与启动脚本
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── clean/             # 前端清洗 API 路由
│   │   │   │   └── route.ts       # POST /api/clean 接口
│   │   │   └── v1/clean/          # Agent 专用 API 路由
│   │   │       └── route.ts       # POST /api/v1/clean 接口（固定深度清洗）
│   │   ├── layout.tsx             # 根布局
│   │   ├── page.tsx               # 主页面（清洗工具 UI）
│   │   └── globals.css            # 全局样式
│   ├── components/ui/             # shadcn/ui 组件库
│   ├── hooks/
│   │   ├── use-japanese-cleaner.ts  # 清洗逻辑 Hook
│   │   └── use-mobile.ts
│   ├── lib/
│   │   ├── japanese-cleaner.ts    # 核心清洗引擎
│   │   └── utils.ts
│   └── server.ts                  # 自定义服务器入口
├── .env.example                   # 环境变量示例
├── next.config.ts
├── package.json
└── tsconfig.json
```

## 核心模块

### `src/lib/japanese-cleaner.ts` — 清洗引擎

- `cleanJapaneseText(input, options)` → `CleanResult`
- 纯函数设计，零外部依赖，可独立打包
- 16 步 Pipeline 执行：Unicode 正规化 → HTML 移除 → 全角半角 → 片假名 → 换行 → 分隔线 → 控制字符 → 引号状态机 → 括号 → 标点 → 行首尾空白 → 括号内容删除 → 长音 → 空格删除 → 单句成行 → 空行处理
- 引号规范化使用状态机，自动匹配 ASCII `"` 为 `「」` 配对
- 括号内容删除使用栈式解析，支持嵌套
- 单句成行使用两阶段算法：Phase 1 合并连接性标点后的换行，Phase 2 在句末标点后断行
- `・`（U+30FB）归类为连接性标点，不触发断行（避免人名断行）

### `src/app/api/clean/route.ts` — 前端 API 路由

- `POST /api/clean` — 接受 `{ text, options? }` 返回 `{ success, data: { cleaned, stats } }`
- `GET /api/clean` — 返回 API 说明与默认选项
- 前端页面通过此接口调用清洗引擎

### `src/app/api/v1/clean/route.ts` — Agent 专用 API 路由

- `POST /api/v1/clean` — 固定深度清洗模式，专为 Agent / 工作流设计
- 三种输入方式：纯文本 `text`、文件 base64 `file_base64`、批量 `items`
- `file_base64` 模式自动检测编码（UTF-8 / ShiftJIS / EUC-JP），可通过 `source_encoding` 手动指定
- API Key 鉴权：`Authorization: Bearer <key>` 或 `?api_key=<key>`
- 未配置 `CLEAN_API_KEY` 环境变量时鉴权自动关闭
- 批量模式最多 100 条，返回 `results` + `summary`
- 错误码：EMPTY_TEXT / MISSING_INPUT / INVALID_BASE64 / EMPTY_ITEMS / TOO_MANY_ITEMS / AUTH_FAILED / INTERNAL_ERROR
- `GET /api/v1/clean` — 返回 API 说明

### `src/hooks/use-japanese-cleaner.ts` — 前端 Hook

- 封装输入/输出状态、清洗选项、API 调用、文件上传、复制/下载等操作
- 多文件状态管理：`files: FileEntry[]`、`activeFileIndex`、`pendingFiles`
- 编码自动检测：先 UTF-8，乱码则回退 ShiftJIS
- 批量导出：`batchExportDownload`（JSZip 压缩下载）
- 文件删除：`removeFileEntry` 删除已上传文件

### `src/app/page.tsx` — 主页面

- 左右分栏布局：左侧输入输出区域，右侧清洗选项面板
- 功能：文本粘贴、TXT 文件上传（拖放支持）、清洗选项配置（4 个分组共 15 项）、清洗模式选择（深度/标准/轻量）、清洗统计、一键复制/下载
- 移动端：操作按钮位于输入区域下方，单列纵向排列
- 批量导入卡片：拖放 + 文件列表 + 编码选择 + 进度条
- Textarea 固定高度 + 滚动 + 回顶部按钮

## 构建与测试命令

- `pnpm dev` — 启动开发服务（端口 5000）
- `pnpm build` — 构建生产版本
- `pnpm lint` — ESLint 检查
- `pnpm ts-check` — TypeScript 类型检查

## 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `CLEAN_API_KEY` | v1 API 鉴权密钥，留空关闭鉴权 | 空 |
| `PORT` | 服务端口 | `5000` |
| `COZE_PROJECT_ENV` | 运行环境 `DEV` / `PROD` | `DEV` |

## 包管理规范

**仅允许使用 pnpm** 作为包管理器，**严禁使用 npm 或 yarn**。
**常用命令**：
- 安装依赖：`pnpm add <package>`
- 安装开发依赖：`pnpm add -D <package>`
- 安装所有依赖：`pnpm install`
- 移除依赖：`pnpm remove <package>`

## 开发规范

### 编码规范

- 默认按 TypeScript `strict` 心智写代码；优先复用当前作用域已声明的变量、函数、类型和导入，禁止引用未声明标识符或拼错变量名。
- 禁止隐式 `any` 和 `as any`；函数参数、返回值、解构项、事件对象、`catch` 错误在使用前应有明确类型或先完成类型收窄，并清理未使用的变量和导入。

### next.config 配置规范

- 配置的路径不要写死绝对路径，必须使用 path.resolve(__dirname, ...)、import.meta.dirname 或 process.cwd() 动态拼接。

### Hydration 问题防范

1. 严禁在 JSX 渲染逻辑中直接使用 typeof window、Date.now()、Math.random() 等动态数据。**必须使用 'use client' 并配合 useEffect + useState 确保动态内容仅在客户端挂载后渲染**；同时严禁非法 HTML 嵌套（如 <p> 嵌套 <div>）。
2. **禁止使用 head 标签**，优先使用 metadata，详见文档：https://nextjs.org/docs/app/api-reference/functions/generate-metadata
   1. 三方 CSS、字体等资源可在 `globals.css` 中顶部通过 `@import` 引入或使用 next/font
   2. preload, preconnect, dns-prefetch 通过 ReactDOM 的 preload、preconnect、dns-prefetch 方法引入
   3. json-ld 可阅读 https://nextjs.org/docs/app/guides/json-ld

## UI 设计与组件规范 (UI & Styling Standards)

- 模板默认预装核心组件库 `shadcn/ui`，位于`src/components/ui/`目录下
- Next.js 项目**必须默认**采用 shadcn/ui 组件、风格和规范，**除非用户指定用其他的组件和规范。**

---
> Source: [Albertaworlds/japanese-text-cleaner](https://github.com/Albertaworlds/japanese-text-cleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
