## ce-ee-boundary

> CE/EE 开发边界 — 文档与设计资产默认归属 EE 私有仓库


# CE/EE 开发边界

## 核心原则

**所有开发已进入 CE（社区版）/ EE（企业版）双版本模式。文档、设计资产默认放 EE（`ee/` 私有仓库），CE（公开仓库）仅保留代码和最小必要的公开文件。**

默认放 EE，除非有明确理由放 CE。

## 文档归属

### 必须放 EE（`ee/docs/`）

| 文档类型 | 示例 |
|----------|------|
| 产品需求文档 | PRD、功能规划 |
| 架构 / 设计文档 | 后端架构、K8s 设计、页面线框图 |
| 部署 / 运维文档 | VKE 部署方案、镜像构建规范 |
| 测试 / 回归文档 | 测试清单、回归报告、i18n 审计 |
| 技术调研 / 参考 | Three.js 参考、六边形网格参考 |
| 任何含业务逻辑或基础设施细节的文档 | — |

### 允许放 CE

仅以下文件可以存在于 CE 公开仓库中：

- **README.md**（根目录和各子项目）— 通用结构说明、开发命令，不含敏感信息
- **CLAUDE.md / AGENTS.md** — 通用开发规范
- **features.yaml** — Feature 注册表（ID + 描述）
- **.cursor/rules/*.mdc** — 编码规范（不含敏感信息）

### 判断标准

不确定放哪时：**这份文档如果被公开，会不会泄露业务细节或基础设施信息？** 会 → EE。不会且属于通用规范 → CE。拿不准 → EE。

## 新建文档流程

1. 确认文档类型，对照上方归属表
2. 默认在 `ee/docs/` 中创建
3. 仅 README / 公开规范放 CE 对应目录
4. 提交前确认文件在正确的仓库中（CE 的 git 和 `ee/` 的 git 是独立的）

## 代码归属

| 归属 | 内容 |
|------|------|
| CE | 核心功能代码（`nodeskclaw-backend/`、`nodeskclaw-portal/` 等） |
| EE | 企业功能扩展（`ee/backend/`、`ee/nodeskclaw-frontend/`、`ee/frontend/portal/`），通过 FeatureGate 控制加载 |

EE 代码通过接口扩展 CE（工厂注册、路由追加、hook 订阅），**不直接修改 CE 代码**。

## 禁止行为

- **禁止在 CE 仓库根目录新建 `docs/` 目录** — 已清空移除，不得重建
- **禁止在 CE 仓库中新建设计文档、PRD、部署方案等**
- **禁止在 CE 的 README / AGENTS.md 中暴露 EE 文档的具体内容** — 用间接引用（"详见 EE 文档"）

---
> Source: [NoDeskAI/nodeskclaw](https://github.com/NoDeskAI/nodeskclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
