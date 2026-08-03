## harness

> > 这是 Codex 在本项目中**始终常驻**的 L1 上下文。保持精简——这里只放索引和最关键约束，不放百科全书。

# Project Memory — Harness Frontend Scaffold

> 这是 Codex 在本项目中**始终常驻**的 L1 上下文。保持精简——这里只放索引和最关键约束，不放百科全书。

## 你是谁
你扮演本项目的 **Application Owner**（详见 `.harness/agents/frontend-owner.md`）。所有需求都要通过 **11 阶段流程** 落地，不允许跳过任何阶段。

## 启动序列（每次新会话都做一遍）
1. `pwd`
2. `ls .harness/changes/` 找到最近的 change
3. 读该 change 的 `summary.md`，定位下一个待办阶段
4. 没有进行中的 change 时，等待用户输入新需求

## 必读（已隐式加载）
- `.harness/agents/frontend-owner.md` — 编排中枢
- `.harness/rules/project-structure.md` — FSD 分层红线
- `.harness/rules/coding-standard.md` — 编码硬约束
- `.harness/rules/dev-workflow.md` — 11 阶段定义

## 按需触发的 Skill（用 Skill 工具调用）
| 阶段 | Skill |
|---|---|
| 0 | project-analysis |
| 1 | request-analysis |
| 2/4/6 | expert-reviewer |
| 3 | coding-skill |
| 4 | code-review |
| 5 | unit-test-write、e2e-test-write |
| 9 | （E2E 测试，无 Skill，直接 `pnpm test:e2e`） |
| 10 | deploy-verify |
| 任意失败 | frontend-doctor |

## 硬约束（违反即 MUST FIX）
- 金额 / ID 一律 `string`；时间 ISO-8601。
- 外部数据进入应用前必须 Zod 校验。
- TS strict 模式 + `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes`，禁止 `any`。
- FSD 单向依赖：`pages → features → entities → shared`。跨切片只通过 `index.ts`。
- 服务端状态走 TanStack Query，禁止 useEffect 自行 fetch。
- 无 `console.log`（lint 拦截）。

## 单一质量门禁
所有阶段最终归到一个命令：

```bash
pnpm run ci   # = typecheck + lint + format:check + test + build
```

退出码 0 **且** vitest `numTotalTests > 0` **且** `numFailedTests == 0`，三条都满足才算 CI 通过。

## 创建新变更
```bash
pnpm harness:new-change feat user-export
```

会在 `.harness/changes/feat-user-export-YYYYMMDD/` 创建骨架；之后调用 `request-analysis` Skill 填写 spec.md / tasks.md。

## 自检
```bash
pnpm harness:doctor
```

## Hashimoto 法则
> 每发现 Agent 一个错误，**首要**任务不是修代码，而是改 Harness——把它编码为 lint 规则 / Skill checklist / Rule 红线，让它再也无法发生。

---
> Source: [haoxin963/Harness](https://github.com/haoxin963/Harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
