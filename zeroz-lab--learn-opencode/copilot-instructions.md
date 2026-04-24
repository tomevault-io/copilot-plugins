## learn-opencode

> 这是 **learn-opencode** 学习文档项目的 Agent 协作规范。本项目用于学习和记录 OpenCode 源码分析。

# AGENTS.md - 代理开发指南

这是 **learn-opencode** 学习文档项目的 Agent 协作规范。本项目用于学习和记录 OpenCode 源码分析。

## 📋 项目概览

- **类型**: 学习文档项目（非源码项目）
- **主要内容**: 中文技术文档、源码分析、架构图
- **源码位置**: `source/opencode/` (Git Submodule，只读)
- **文档位置**: `docs/` (主要工作区)

## 🔧 环境与命令

### 初始化项目
```bash
# 首次克隆后，初始化源码子模块
git submodule update --init --recursive

# 更新源码到最新版本
git submodule update --remote
```

### 源码项目命令（仅供参考）
在 `source/opencode/` 目录下：
```bash
# 安装依赖
bun install

# 运行 OpenCode
bun dev  # 在 packages/opencode 下

# 类型检查
bun turbo typecheck

# 测试（注意：从 root 禁止运行测试）
# 必须在具体的 package 下运行
cd packages/opencode && bun test
cd packages/sdk/js && bun test

# 单个测试文件
bun test path/to/test.test.ts

# 格式化检查
prettier --check .
```

### 文档项目命令
```bash
# 本项目没有构建命令，主要是编写和查看 Markdown 文档
# 建议使用 Markdown 预览工具查看文档
```

## 📝 代码风格（仅供源码参考）

基于 `source/opencode/STYLE_GUIDE.md`：

### 函数与结构
- ✅ 保持函数简洁，除非可复用否则避免过度拆分
- ✅ 使用单词变量名（如 `result`, `client`, `data`）
- ❌ 避免不必要的解构：用 `obj.a` 而非 `const { a } = obj`
- ❌ 避免 `else` 语句（使用 early return）
- ❌ 避免 `try/catch`（除非必要）
- ❌ 避免 `let`（优先使用 `const`）

### 类型系统
- ✅ 使用 Zod Schema 定义数据结构
- ✅ 使用 TypeScript 严格类型
- ❌ 避免使用 `any` 类型
- ✅ 导出类型：`export type Info = z.infer<typeof Info>`

### 导入规范
```typescript
// 1. 外部库
import { z } from "zod"
import { Hono } from "hono"

// 2. 内部模块（使用路径别名）
import { Log } from "@/util/log"
import { Session } from "@/session"

// 3. 相对导入
import { helper } from "./helper"
```

### Bun API 优先
```typescript
// ✅ 使用 Bun API
const file = Bun.file(path)
const text = await file.text()
await Bun.write(file, content)

// ❌ 避免 Node.js fs
import fs from "fs/promises"
```

### 错误处理
```typescript
// ✅ 使用 NamedError
throw new Session.NotFoundError({ sessionID })

// ✅ Early return
if (!session) {
  throw new Error("Not found")
}
// 继续处理...

// ❌ 避免 else
if (condition) {
  return a
} else {  // 不推荐
  return b
}
```

### 格式化配置
```json
{
  "semi": false,           // 不使用分号
  "printWidth": 120,       // 行宽 120
  "tabWidth": 2,           // 缩进 2 空格
  "singleQuote": false     // 使用双引号
}
```

## 📚 文档编写规范

### 核心原则
- **语言**: 始终使用**中文**（参考 `CLAUDE.md`）
- **角色**: 作为 OpenCode 项目的专家导游
- **格式**: 清晰的 Markdown，多用列表、表格、引用

### 文档结构
```
docs/
├── packages/      # 包分析文档
├── concepts/      # 协议和概念文档
├── internals/     # 内部模块深入分析
├── flow/          # 流程图和时序图
├── templates/     # 文档模板
└── learning_paths.md
```

### Markdown 规范
```markdown
# 标题使用中文

## 1. 概览 (Overview)

- **路径**: `packages/xxx/`
- **定位**: 简短描述
- **核心职责**: 列表说明

### 代码示例

\`\`\`typescript
// 带注释的代码示例
export const Example = z.object({
  id: z.string(),
})
\`\`\`

### Mermaid 图表

\`\`\`mermaid
graph TB
    A[开始] --> B[处理]
    B --> C[结束]
\`\`\`
```

### 视觉风格
图片生成提示词模板：
```
[具体描述]. Clean, modern, flat vector style, dark theme with neon accents. High resolution technical illustration.
```

## 🎯 工作流程

### 创建新文档
1. 参考 `docs/templates/` 下的模板
2. 使用中文编写，保持风格一致
3. 添加必要的代码示例和图表
4. 在相应的 README 或索引中添加链接

### 分析源码
1. 先阅读 `source/opencode/` 下的源码
2. 理解模块的职责和数据流
3. 提取关键代码片段（带注释）
4. 创建流程图和架构图

### 文档审查
- ✅ 中文表达准确流畅
- ✅ 代码示例可读性强
- ✅ 图表清晰易懂
- ✅ 文件路径正确
- ✅ 链接有效

## 🚫 注意事项

### 禁止操作
- ❌ **不要修改 `source/opencode/` 下的任何文件**（这是 Git Submodule）
- ❌ **不要运行 `bun test` 在 root**（会报错）
- ❌ **不要创建英文文档**（除非特殊需求）

### 推荐做法
- ✅ 使用并行工具调用提高效率
- ✅ 阅读现有文档了解风格
- ✅ 使用 Mermaid 绘制流程图
- ✅ 代码示例保持简洁清晰
- ✅ 交叉引用相关文档

## 📖 参考资源

### 关键文档
- `README.md` - 项目总览
- `CLAUDE.md` - AI 协作规范
- `docs/learning_paths.md` - 学习路径
- `source/opencode/AGENTS.md` - OpenCode 源码开发规范
- `source/opencode/STYLE_GUIDE.md` - 源码风格指南

### 重要概念
- **Git Submodule**: `source/opencode/` 是只读的外部仓库引用
- **Monorepo**: 源码项目使用 Turborepo + Bun workspaces
- **三层协议**: ACP (IDE集成) + MCP (工具扩展) + LSP (代码智能)

## 🤖 Agent 特别提示

当你作为 AI Agent 在此项目工作时：

1. **明确角色**: 你是文档编写者，不是源码开发者
2. **只读源码**: 只参考 `source/opencode/`，不修改
3. **中文优先**: 所有文档和交流使用中文
4. **质量优先**: 宁可少写，也要保证质量
5. **引用准确**: 引用源码时使用正确的文件路径

---

**版本**: 1.0  
**更新**: 2025-01-09  
**目标受众**: AI 编程助手（Claude, Copilot, 等）

---
> Source: [ZeroZ-lab/learn-opencode](https://github.com/ZeroZ-lab/learn-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
