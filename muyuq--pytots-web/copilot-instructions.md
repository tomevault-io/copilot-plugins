## pytots-web

> **Generated:** 2026-07-04

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-07-04
**Commit:** 5fd5a4a
**Branch:** main

## OVERVIEW

Python to TypeScript 双语学习站点。基于 Astro + Starlight 构建的静态文档网站，包含教程、算法题解和交互测验。

## STRUCTURE

```
./
├── src/
│   ├── components/          # 自定义组件 (QuizContainer, PathNavigator...)
│   ├── content/docs/        # 文档内容 (MDX)
│   │   ├── algorithms/      # 36 道算法题 (双语实现)
│   │   ├── paths/           # 学习路径
│   │   │   ├── preparation/ # 准备 2 课
│   │   │   ├── foundation/  # 基础 5 课
│   │   │   ├── migration/   # 迁移 7 课
│   │   │   └── advanced/    # 进阶 8 课
│   │   ├── handbook/        # 速查手册
│   │   └── practice/        # 练习测验
│   ├── lib/                 # 工具库
│   ├── pages/               # 自定义页面
│   └── styles/              # 全局样式
├── tests/
│   ├── unit/                # 单元测试
│   └── e2e/                 # Playwright E2E
├── scripts/                 # 构建脚本
└── docs/plans/              # 设计文档
```

## WHERE TO LOOK

| 任务         | 位置                                 | 说明                                                               |
| ------------ | ------------------------------------ | ------------------------------------------------------------------ |
| 新增算法题   | `src/content/docs/algorithms/`       | 见 [docs/algorithms-AGENTS.md](./docs/algorithms-AGENTS.md) |
| 修改课程内容 | `src/content/docs/paths/*/`          | 每课独立 MDX                                                       |
| 组件开发     | `src/components/`                    | Astro 组件                                                         |
| 添加测验题目 | `src/components/QuizContainer.astro` | 内置题库                                                           |
| 样式调整     | `src/styles/` + Starlight 主题       |
| 构建问题     | `astro.config.mjs`                   | 站点配置                                                           |

## CONVENTIONS

### 内容文件 (MDX)

- **Frontmatter 必填**: `title`, `kind`, `level`, `topic`, `difficulty`, `prerequisites`, `python_tags`, `ts_tags`, `description`
- **课程结构**: `## 场景与问题` → `## Python 回顾` → `## TypeScript 等价写法` → `## 差异与常见陷阱` → `## 练习` → `## 面试追问`
- **算法结构**: `## 问题描述` → `## 思路分析` → `## 复杂度分析` → `## Python 实现` → `## TypeScript 实现` → `## 面试变体`

### 代码示例

- 双语对照：Python 在上，TypeScript 在下
- 注释使用 `#` (Python) 和 `//` (TypeScript)
- 变量命名保持一致（如都用 `nums` 而不是 `arr`）

### 测验题目

- 存储于 `QuizContainer.astro` 的 `quizData` 对象
- 每题：question + 4 options (含 correct flag + explanation)

## ANTI-PATTERNS (THIS PROJECT)

- **禁止**: 使用 `any` 类型（使用 `unknown` 替代）
- **禁止**: 在课程内容中加 `## 标题` 重复 frontmatter title
- **禁止**: 过多使用"你""我"人称（技术文档保持客观）
- **禁止**: AI 式开场白（"作为开发者..."）
- **要求**: `const` 优先于 `let`，除非必须重新赋值

## COMMANDS

```bash
# 开发
npm run dev -- --host 127.0.0.1

# 质量检查 (lint + typecheck + test + build + linkcheck)
npm run check

# 测试
npm run test        # 单元测试
npm run test:e2e    # Playwright

# 构建
npm run build       # 静态站点
```

## NOTES

- **端口**: 开发服务器默认端口 4321（Astro 默认值），被占用时自动递增
- **内容检查**: `npm run linkcheck` 验证内部链接
- **搜索索引**: 构建时自动生成 Pagefind 索引
- **CI/CD**: GitHub Actions 自动部署到 GitHub Pages

---
> Source: [MuyuQ/PyToTS_WEB](https://github.com/MuyuQ/PyToTS_WEB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
