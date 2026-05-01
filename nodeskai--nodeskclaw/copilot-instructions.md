## docs-first-workflow

> 先改文档后写代码 — 文档驱动开发工作流


# 文档驱动开发工作流

## 核心规则

**任何功能实现之前，必须先更新对应的设计文档。** 不允许跳过文档直接写代码。

## 工作流程

1. **接到需求** → 先确认涉及哪些文档
2. **更新文档** → 在设计文档中补充/修改对应章节
3. **Review 文档** → 让用户确认文档变更
4. **再写代码** → 文档确认后才开始实现

## README 规则

**任何新建的目录/项目/模块，必须包含 README。** 没有 README 不算完成。

- 新建子项目目录（如 `nodeskclaw-artifacts/`）→ 必须写 README
- 新建功能模块目录 → 如有独立用途，必须写 README
- README 内容至少包含：项目/模块用途、目录结构、使用方法
- 已有目录补 README 时，随改动一并提交

## 代码变更必须同步文档

**每次改完代码，在同一轮操作中必须更新所有受影响的文档。** 不是"下次补"，是"这次就补"。

文档分两类，**都要检查**：

### 1. ee/docs/ 设计文档（按对照表更新）

设计文档位于 EE 私有仓库的 `ee/docs/` 目录中（包含敏感内容，不纳入 CE 公开仓库）。

| 变更内容 | 必须更新 |
|----------|----------|
| 新增/修改 API、路由、中间件 | `ee/docs/后端架构设计.md` |
| 新增/修改页面、交互流程 | `ee/docs/页面线框图.md`、`ee/docs/PRD-NoDeskClaw产品需求文档.md` |
| K8s 资源定义变更（Deployment/Service/Ingress 等） | `ee/docs/K8s交互层实现设计.md`、`ee/docs/VKE部署方案.md` |
| 部署流程、镜像构建变更 | `ee/docs/VKE部署方案.md`、`ee/docs/DeskClaw镜像构建规范.md` |
| 技术栈、整体架构变更 | `ee/docs/PRD-NoDeskClaw产品需求文档.md`、`ee/docs/后端架构设计.md` |
| 数据库模型、表结构变更 | `ee/docs/后端架构设计.md` |
| 新增基础设施组件（如日志、Ingress Controller） | `ee/docs/后端架构设计.md`、`ee/docs/VKE部署方案.md` |

### 2. README 文档（按影响范围更新）

- **根 README** (`README.md`)：整体架构、项目结构变更时更新
- **后端 README** (`nodeskclaw-backend/README.md`)：后端配置项、启动方式、目录结构、新功能模块变更时更新
- **Portal 前端 README** (`nodeskclaw-portal/README.md`)：Portal 前端配置项、启动方式变更时更新
- **Admin 前端 README** (`ee/nodeskclaw-frontend/README.md`)：Admin 前端（EE）配置项、启动方式变更时更新
- **制品 README** (`nodeskclaw-artifacts/README.md`)：部署制品、镜像变更时更新
- **子目录 README**：新建目录时必须创建

### 3. 注释 / docstring

- 函数签名、参数变了 → 同步更新注释和 docstring

**判断标准**：如果一个新人看了文档但没看代码，会不会被误导？会就说明文档没更新到位。

## 禁止行为

- 不准在文档没更新的情况下创建新文件或目录
- 不准先写代码再"补"文档
- 不准跳过文档确认直接进入实现阶段
- 不准新建目录后不写 README
- **不准改了代码不更新文档** — 代码和文档必须在同一次操作中同步完成
- **不准说"后续再更新文档"** — 没有后续，当下就更新
- **不准只写脚本不写说明** — 新建 Shell 脚本、CI/CD 配置、Dockerfile、K8s 清单等基础设施文件时，必须同时写 README 和更新对应的 ee/docs/ 设计文档

---
> Source: [NoDeskAI/nodeskclaw](https://github.com/NoDeskAI/nodeskclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
