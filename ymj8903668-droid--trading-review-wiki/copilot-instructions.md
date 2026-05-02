## trading-review-wiki

> **Trading Review Wiki** — A股交易复盘知识库应用。LLM 自动从交易资料（研报、交割单、笔记）构建结构化知识库，支持对话查询、知识图谱、Deep Research、交易订单导入与 FIFO 盈亏计算。

# CLAUDE.md — Trading Review Wiki

## 项目概述

**Trading Review Wiki** — A股交易复盘知识库应用。LLM 自动从交易资料（研报、交割单、笔记）构建结构化知识库，支持对话查询、知识图谱、Deep Research、交易订单导入与 FIFO 盈亏计算。

**仓库**: https://github.com/ymj8903668-droid/trading-review-wiki
**版本**: 0.6.x（遵循 semver）
**许可证**: GNU GPL v3.0

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 桌面壳 | Tauri v2 (Rust 后端) |
| 前端 | React 19 + TypeScript + Vite |
| UI | shadcn/ui + Tailwind CSS v4 + CVA |
| 编辑器 | Milkdown (ProseMirror) |
| 图表/图谱 | sigma.js + graphology + ForceAtlas2 |
| 状态管理 | Zustand |
| 国际化 | react-i18next |

---

## 开发命令

```bash
# 前端热重载开发
npm run dev

# 完整桌面应用开发（Rust 后端 + 前端）
npm run tauri dev

# 生产构建（生成可执行文件）
npm run tauri build

# 仅前端构建（dist/）
npm run build

# 单元测试
npm run test

# TypeScript 类型检查
npx tsc --noEmit
```

**首次设置**:
```bash
npm install
# Tauri CLI 环境自动检测，会提示安装 Rust（如果未安装）
```

---

## 代码规范

### 目录结构
```
src/                    # React 前端源码
  commands/             # Tauri 命令封装（Rust IPC 调用）
  components/           # React 组件（按功能分组）
  i18n/                 # 国际化文案
  lib/                  # 工具函数、辅助逻辑
  stores/               # Zustand 状态存储
  types/                # TypeScript 类型定义
src-tauri/              # Rust 后端（Tauri 核心逻辑）
  src/
    commands.rs         # Tauri 命令实现（文件读写、进程调用）
```

### 组件规范
- **组件文件**: PascalCase，如 `ChatMessage.tsx`
- **Store 文件**: `<name>-store.ts`，如 `activity-store.ts`
- **工具函数**: camelCase，如 `format-date.ts`
- **组件放在 `components/`** 按功能/页面组织，不要扁平堆叠

### TypeScript
- 严格模式：启用 `strict: true`
- 优先使用 `interface`，类型推导能推断时不写类型注解
- API 响应结构定义在 `types/` 目录
- 避免 `any`，用 `unknown` + 类型守卫

### Tailwind CSS
- 使用 Tailwind v4（CSS-first 配置，在 `index.css` 中用 `@theme` 定义设计系统变量）
- 颜色、间距、圆角优先用 design token（CSS 变量），不用硬编码值
- 使用 CVA（class-variance-authority）管理组件多变体

---

## 版本管理

### 语义化版本（semver）
- **MAJOR**: 不兼容的 API 变更（如 Tauri v1 → v2 级别）
- **MINOR**: 新增功能（向后兼容）
- **PATCH**: Bug 修复（向后兼容）

### CHANGELOG 规范（必须遵守）
文件：`CHANGELOG.md`，语言：**中文**

格式：
```markdown
## v0.6.6 — 2026-04-19

### 修复（Bug Fix）
- **修复 XXX 问题**：[文件] 问题描述。已修复描述。

### 改进（Improvement）
- **优化 XXX**：[文件] 改进内容

### 新功能（Feature）
- **新增 XXX**：功能描述

---

## Git 工作流

### 分支命名
```
feature/<功能名>        # 新功能，如 feature/vector-search
fix/<问题描述>          # Bug 修复，如 fix/save-to-wiki-loop
chore/<任务>           # 杂项，如 chore/update-deps
```

### 提交信息（Conventional Commits）
```
fix: 修复 Save to Wiki 无限循环
feat: 新增向量语义搜索（可选）
chore: 升级依赖到 React 19
docs: 更新 README
```

### 发布流程
1. 在 `CHANGELOG.md` 顶部添加新版本条目（版本号 + 日期 + 变更说明）
2. `npm run tauri build` 生成安装包（Windows: `.msi`，macOS: `.dmg`）
3. Git tag: `git tag v0.6.7 && git push origin v0.6.7`
4. GitHub Actions 自动构建并发布到 Releases

**注意**: 不要在 CHANGELOG.md 的小标题里写英文缩写（如 `fix`），全用中文。

---

## 常见任务参考

### 修复 Bug
1. 用 `npm run tauri dev` 本地复现
2. 定位问题文件（优先看 `src/` 下相关组件/Store）
3. 修复后手动验证
4. 更新 CHANGELOG.md（修复条目）
5. Commit → Push →（必要时）打 tag 发布

### 新增功能
1. 在 `src/` 相关目录添加组件/逻辑
2. 用 `npm run tauri dev` 验证 UI
3. 添加对应 i18n 文案（`src/i18n/`）
4. 更新 CHANGELOG.md（新功能条目）
5. Commit → Push → 打 tag

### 上传到 GitHub
```bash
git add .
git commit -m "fix: 修复 Save to Wiki 无限循环"
git push origin main
# 如果有 tag 需要同步：
git push origin v0.6.7
```

---

## 重要项目约定

- **Wiki 输出目录使用中文命名**：写入 `wiki/股票/`、`wiki/概念/`、`wiki/查询/` 等中文目录，不创建英文目录
- **文件路径标准化**：所有路径用 `normalizePath()` 统一处理（避免 Windows `\` 和 `/` 混乱）
- **Rust 后端文件操作**：优先用 Tauri 命令（`commands.rs`），不要在前端直接 fs 操作
- **auto-ingest 异步处理**：耗时操作通过队列串行处理，支持取消和重试
- **对话持久化**：Chat 历史存在 `.llm-wiki/chats/` 下，重启不丢失

---

## 测试规范

### 测试框架
- **前端**: Vitest（已配置在 `vite.config.ts`）
- **Rust 后端**: `cargo test`

### 测什么
- 工具函数（`src/lib/` 下的纯函数）
- Store 的核心状态转换逻辑
- Rust 命令的输入校验和边界情况

### 不测什么
- UI 组件渲染（不引入 React Testing Library，避免维护负担）
- 第三方库封装（信任上游）
- 需要真实文件系统/网络的操作（用 mock）

### 文件组织
```
src/lib/__tests__/format-date.test.ts   # 与被测文件同目录
src/stores/__tests__/wiki-store.test.ts
```

### 命名规则
- 测试文件: `<被测文件名>.test.ts`
- 测试用例: `describe` 用被测函数名，`it` 描述行为而非结果

```ts
// 好
describe('calculateFifoPnl', () => {
  it('先买后卖，盈利计算正确', () => { ... })
  it('卖出数量超过持仓，抛出错误', () => { ... })
})

// 坏
describe('calculateFifoPnl', () => {
  it('works', () => { ... })
  it('returns correct value', () => { ... })
})
```

### 运行时机
- 本地提交前: `npm run test` 通过
- CI: 同

---

## 错误处理约定

### 错误分级

| 级别 | 用户可见 | 场景 | 处理方式 |
|------|---------|------|---------|
| **用户错误** | 是 | 文件不存在、输入格式错误、网络超时 | Toast 提示，中文，可操作建议 |
| **系统错误** | 否 | 内部状态不一致、未预期的分支 | 记录日志，Toast 只显示"操作失败，请重试" |
| **致命错误** | 是（简化版） | Rust panic、数据损坏 | 弹窗提示重启应用，完整错误写日志 |

### Rust → 前端错误映射

Rust 命令统一返回 `Result<T, String>`，错误字符串格式：

```
[错误码] 中文提示
```

示例：
```rust
// Rust
Err("FILE_NOT_FOUND 文件不存在，请检查路径: /wiki/股票/xxx.md".into())
```

前端解析：
```ts
const [code, message] = error.split(' ', 2)
// code = "FILE_NOT_FOUND"
// message = "文件不存在，请检查路径: ..."
```

### 错误码规范（Rust）

| 错误码 | 含义 | 前端处理 |
|--------|------|---------|
| `FILE_NOT_FOUND` | 文件/目录不存在 | Toast: 具体路径 |
| `INVALID_PATH` | 路径越权或格式错误 | Toast: 路径非法 |
| `WRITE_FAILED` | 写入失败（磁盘满/权限） | Toast: 写入失败，请检查磁盘空间 |
| `PARSE_ERROR` | 数据解析失败 | Toast: 数据格式错误 |
| `TIMEOUT` | 操作超时 | Toast: 操作超时，请重试 |
| `UNKNOWN` | 未分类错误 | 记录日志，Toast: 操作失败 |

### 前端错误展示

统一用 `src/components/ui/sonner.tsx`（或现有 toast 组件），禁止：
- `alert()`
- 控制台裸 `console.error` 不处理
- 把 Rust 原始错误直接抛给用户

---

## 性能红线

### 图谱渲染（sigma.js）
| 指标 | 上限 | 超限处理 |
|------|------|---------|
| 节点数 | 3000 | 提示归档旧页面，只渲染最近更新的 3000 个 |
| 边数 | 10000 | 按权重截断，只保留权重前 80% 的边 |
| 力导向迭代 | 150 次（约 1-3s） | 超时 5s 自动停止，fallback 到缓存布局 |
| 目录遍历深度 | 30 层 | 代码已硬编码，无需额外处理 |

### 文件加载
| 场景 | 上限 | 超限处理 |
|------|------|---------|
| 文本文件 (.md, .txt) | 50 MB | 拒绝读取，Toast: "文件过大，请拆分或归档" |
| PDF/DOCX/Office | 100 MB | 同上 |
| Excel (交割单) | 20 MB | 同上，提示检查是否含无关 sheet |
| 批量导入文件总数 | 1000 个 | 分批处理，每批 200 个，进度条展示 |

### 聊天记录
| 场景 | 上限 | 处理 |
|------|------|------|
| 单对话消息数 | 100 条 | 提示"对话过长"，提供"新建对话并归档"按钮 |
| 对话列表总数 | 50 个 | 侧边栏自然滚动，不额外处理 |
| LLM 上下文历史 | 10 条 | 代码已配置，可调 |

### 响应时间
| 操作 | 最大可接受时间 | 超时处理 |
|------|---------------|---------|
| 点击打开文件 | 1s | 显示骨架屏 |
| 保存到 Wiki | 5s | 进度条，可取消 |
| Deep Research | 120s | 必须支持取消按钮 |
| 图谱布局 | 5s | 停止计算，使用上次缓存布局 |

---

## CLAUDE.md 维护规则

### 谁可以改
- **兔子妈妈**：最终审批权，所有修改需确认
- **Claude Code**：可起草修改，但必须经用户确认后写入

### 什么时候必须改
- 新增技术栈或工具（如换测试框架、加新库）
- 发现现有规范导致反复踩坑（如某类 bug 出现 3 次以上）
- 发布 MINOR 版本时（至少回顾一次是否需要更新）
- 团队约定变化（如分支策略调整）

### 什么时候不能改
- 紧急修 bug 时（先修，事后补文档）
- 未经讨论的个人偏好调整

### 修改流程
1. **提议**：在修改前说明"要改 CLAUDE.md 的哪条、为什么"
2. **确认**：用户明确同意后再写入
3. **记录**：在 CLAUDE.md 末尾加版本记录

### 冲突解决
- 与旧代码冲突：优先改代码适配规范，除非改动量过大
- 规范之间冲突：以"更具体"的条款为准，随后修正矛盾

---

## 规范变更记录

| 日期 | 版本 | 变更内容 | 变更人 |
|------|------|---------|--------|
| 2026-04-20 | v0.6.6 | 新增测试规范、错误处理约定、性能红线、CLAUDE.md 维护规则 | Claude |

---

## 项目特定背景（给 AI 的上下文）

这个 Wiki 应用是杰哥的 A股交易复盘工具。核心用户是杰哥本人，背景：
- A股多年实战经验，有完整交易体系
- 关注情绪周期、龙头战法、仓位管理
- 知识库内容来源：公众号文章、朋友圈、群聊消息、研报、交割单

这意味着：
- 界面语言默认中文
- 生成的 Wiki 页面要有交易视角（股票、板块、情绪、策略）
- FIFO 盈亏计算要精确（浮点精度用 Rust 后端保证 4 位小数）

---
> Source: [ymj8903668-droid/trading-review-wiki](https://github.com/ymj8903668-droid/trading-review-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
