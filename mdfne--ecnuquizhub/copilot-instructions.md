## ecnuquizhub

> 帮助高中数学教师建设个人题库的本地 Web 工具。教师可将分散的纸质资料、截图、PDF 等整理成可编辑、可检索、可组卷的结构化题目资源。

# 花狮题苑 - 项目说明

## 项目概述

帮助高中数学教师建设个人题库的本地 Web 工具。教师可将分散的纸质资料、截图、PDF 等整理成可编辑、可检索、可组卷的结构化题目资源。

**团队**：林依鹏、汪宇轩（技术总负责人）、张嘉盈、郑智心、吴妍霏

**第一阶段目标**（2026年10月前）：本地可部署的教师端最小可用系统 + 样例题库

## 技术栈

- **前端**：React 18 + TypeScript + Vite + Ant Design 5 + Zustand
- **后端**：Express 4 + TypeScript + Mongoose 8
- **数据库**：MongoDB（本地实例，数据库名 `quizhub`）
- **公式渲染**：MathJax 3（通过 better-react-mathjax）
- **包管理**：pnpm workspaces（monorepo）

## 项目结构

```
ECNUQuizHub/
├── client/                  # 前端 React 项目
│   └── src/
│       ├── api/             # Axios API 封装
│       ├── components/      # QuestionCard、MathJaxRenderer、KnowledgeTree
│       ├── pages/           # QuestionList、QuestionDetail、QuestionEdit
│       ├── stores/          # Zustand 状态（questionStore）
│       ├── types/           # TypeScript 类型定义（question.ts）
│       ├── App.tsx          # 路由 + 布局（fixed header + main 容器滚动）
│       └── main.tsx
├── server/                  # 后端 Express 项目
│   └── src/
│       ├── models/          # Mongoose Models（Question.ts）
│       ├── routes/          # API 路由（questions.ts）
│       ├── middleware/       # 错误处理
│       ├── types/           # 类型声明
│       └── app.ts           # Express 入口
├── data/                    # 样例数据（sample-questions.json）
├── pnpm-workspace.yaml
├── 技术选型说明.md
└── proto.png                # UI 原型图
```

## 本地运行

```bash
# 确保 MongoDB 服务已启动（Windows 服务名：MongoDB）
pnpm dev
# 前端：http://localhost:5173
# 后端：http://localhost:3000
# Vite 会将 /api 请求代理到后端
```

## API 端点

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/questions | 题目列表（分页、关键词、标签、题型、难度筛选） |
| GET | /api/questions/:id | 题目详情 |
| POST | /api/questions | 新建题目 |
| PUT | /api/questions/:id | 编辑题目 |
| DELETE | /api/questions/:id | 删除题目 |

## 题目数据结构

```typescript
{
  content: string;           // 题干（支持 LaTeX，\( ... \) 行内，\[ ... \] 块级）
  options?: string[];        // 选项（选择题）
  answer: string;            // 答案
  analysis: string;          // 解析
  images?: string[];         // 配图路径
  type: 'choice' | 'fill' | 'solve';
  difficulty: 1 | 2 | 3 | 4 | 5;
  tags: string[];            // 知识点、方法标签
  source: string;            // 来源
  year?: number;             // 年份
  status: 'draft' | 'reviewed' | 'published';
}
```

## 布局设计

- **Header**：fixed 定位，导航项：个人题库、精选题目、资源库、知识图谱、备课、使用文档；右侧：新建、导入按钮 + 头像
- **主内容区**：`height: calc(100vh - 64px)`，内部滚动
- **题目列表页**：左栏知识点树（独立滚动）+ 右栏题目卡片列表
- **题目详情页**：三栏——左栏知识点树、中间题目详情（绿色标题栏 + 底部工具栏）、右栏预览卡片
- **知识点树**：知识点/解题方法 tab、搜索过滤、多选开关

## 已知注意事项

- `tsconfig.app.json` 中 `verbatimModuleSyntax` 设为 false，因为 `@ant-design/icons` 是 CJS 包
- antd 图标使用命名导入：`import { XxxOutlined } from '@ant-design/icons'`
- 用户的全局指令：不要删除代码注释，如果错误可以修正

## 暂未实现的功能

- 精选题目（收藏功能）
- 资源库（PDF/图片导入）
- 知识图谱
- 备课（组卷/组讲义）
- 使用文档页面
- 分步解析、步骤级知识点
- 自动 OCR 导入
- 学生端
- 语义检索
- 可视化指令自动生成
- 白板、相似题功能

---
> Source: [MdFne/ECNUQuizHub](https://github.com/MdFne/ECNUQuizHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
