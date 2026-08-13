## ai-course-maker

> AI课程制作助手 - 基于工作流的企业培训课程自动生成系统。用户输入课程目标，系统自动生成课程框架、PPT内容、讲师脚本、语音合成和视频合成。

# AGENTS.md

## 项目概览

AI课程制作助手 - 基于工作流的企业培训课程自动生成系统。用户输入课程目标，系统自动生成课程框架、PPT内容、讲师脚本、语音合成和视频合成。

## 版本技术栈

- **Framework**: Next.js 16 (App Router)
- **Core**: React 19
- **Language**: TypeScript 5
- **UI 组件**: shadcn/ui (基于 Radix UI)
- **Styling**: Tailwind CSS 4
- **Database**: Supabase (PostgreSQL + Drizzle ORM)
- **AI SDK**: coze-coding-dev-sdk (LLM + TTS + Storage) + Provider抽象层(BYOK)
- **Package Manager**: pnpm (严禁 npm/yarn)

## 目录结构

```
├── public/                 # 静态资源
├── scripts/                # 构建与启动脚本
├── src/
│   ├── app/                # 页面路由与布局
│   │   ├── page.tsx        # 首页 - 课程列表
│   │   ├── layout.tsx      # 根布局
│   │   ├── courses/[id]/   # 课程工作台页面
│   │   ├── models/         # AI模型配置页面
│   │   ├── settings/       # BYOK服务配置页面(LLM/TTS/S3)
│   │   └── api/            # API Routes
│   │       ├── courses/    # 课程 CRUD + 生成接口
│   │       │   ├── route.ts
│   │       │   └── [id]/
│   │       │       ├── route.ts       # GET详情 / DELETE
│   │       │       ├── generate/route.ts  # LLM生成(框架/PPT/脚本)
│   │       │       ├── slides/route.ts    # 幻灯片CRUD
│   │       │       ├── audio/route.ts     # TTS语音合成
│   │       │       └── video/route.ts     # 视频合成(多模式)
│   │       ├── models/     # 视频模型配置 CRUD
│   │       └── providers/  # BYOK服务配置 CRUD + 测试连接
│   ├── components/
│   │   ├── ui/             # Shadcn UI 组件库
│   │   ├── course/         # 课程相关组件
│   │   │   └── CourseWorkspace.tsx
│   │   └── workflow/       # 工作流相关组件
│   │       └── WorkflowStepper.tsx
│   ├── hooks/              # 自定义 Hooks
│   ├── lib/
│   │   ├── utils.ts        # 通用工具函数 (cn)
│   │   ├── workflow.ts     # 工作流定义(步骤/Prompts/TTS音色)
│   │   ├── providers.ts    # Provider抽象层(LLM/TTS/S3 BYOK)
│   │   └── storage.ts      # 对象存储客户端(Coze默认)
│   └── storage/
│       └── database/
│           ├── supabase-client.ts   # Supabase客户端
│           └── shared/schema.ts     # Drizzle表定义
├── next.config.ts
├── package.json
└── tsconfig.json
```

## 核心数据模型

| 表名 | 说明 | 关键字段 |
|------|------|----------|
| courses | 课程项目 | title, objective, status, current_step, render_mode |
| workflow_steps | 工作流步骤 | course_id, step_type, step_index, status, input/output_data |
| slides | PPT页面 | course_id, page_index, title, bullet_points, notes |
| audio_files | 音频文件 | slide_id, course_id, file_key, duration_seconds |
| artifacts | 最终产物 | course_id, type(ppt/audio/video), file_key |
| video_models | 用户AI模型配置 | name, provider, api_endpoint, api_key_encrypted |
| user_providers | BYOK服务配置 | type(llm/tts/storage), provider, config(JSON), is_active |

## 工作流5步骤

1. **framework** - LLM生成课程框架大纲 (3-5模块/2-4章节/2-4知识点)
2. **ppt** - LLM根据框架生成PPT逐页内容 (标题/要点/配图描述)
3. **script** - LLM根据PPT生成口语化讲师脚本 (2-3分钟/页)
4. **audio** - TTS逐页合成音频 (多音色可选)
5. **video** - 视频合成 (静态幻灯片/Remotion动画/用户AI模型)

## 构建与测试命令

- `pnpm dev` - 开发环境启动 (端口5000)
- `pnpm build` - 生产构建
- `pnpm start` - 生产环境启动
- `pnpm ts-check` - TypeScript类型检查
- `pnpm lint --quiet` - ESLint检查

## 编码规范

- 默认 TypeScript strict 模式，禁止隐式 any
- 函数参数/返回值/事件对象必须有明确类型
- 前端动态内容必须使用 'use client' + useEffect + useState 防止 Hydration 错误
- 禁止在 JSX 中直接使用 typeof window / Date.now() / Math.random()
- 集成服务(LLM/TTS/Storage)只通过 coze-coding-dev-sdk 调用，禁止 Mock
- API Key 等敏感信息禁止硬编码，使用环境变量

---
> Source: [Pxx0101/ai-course-maker](https://github.com/Pxx0101/ai-course-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
