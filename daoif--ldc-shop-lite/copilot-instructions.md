## ldc-shop-lite

> `ldc-shop-lite` 是一个面向 Linux DO 生态的轻量级个人店铺：`connect.linux.do` 登录 + `credit.linux.do`（EPay）支付 + `D1` 存储，追求 **低请求/低 CPU**，但保留完整闭环（浏览 → 下单 → 支付 → 回调发货 → 查订单）与单管理员后台。

# AGENTS.md（ldc-shop-lite）

## 项目一句话

`ldc-shop-lite` 是一个面向 Linux DO 生态的轻量级个人店铺：`connect.linux.do` 登录 + `credit.linux.do`（EPay）支付 + `D1` 存储，追求 **低请求/低 CPU**，但保留完整闭环（浏览 → 下单 → 支付 → 回调发货 → 查订单）与单管理员后台。

## 术语口径（协作统一）

- **发布**：指把代码发布到 GitHub（例如 push、merge、打 tag）。
- **部署**：指代码已在 GitHub 后，继续在 GitHub/Cloudflare 完成上线动作（如绑定 D1、配置环境变量、执行迁移并成功启动到生产可用）。

## 关键约束（写代码时永远别忘）

- **项目完全独立**：`ldc-shop-lite/` 不允许依赖reference任何运行时代码/包/配置（只能参考思路与文档）。
- **稳定优先但不极端**：在 CF 免费额度下做性能控制，同时保证用户体验，不做“为了省资源牺牲可用性”的极端优化。
- **严格一致性点**：库存预留/锁单/解锁必须严谨；支付回调必须验签 + 金额校验 + 幂等。
- **产品范围**：只做 Linux DO 登录与 Credit 支付；不做退款/积分/邮件/图片处理/用户通知系统(管理员通知除外)。
- **权限模型极简**：只允许 1 个管理员（`ADMIN_USERNAME` 单值判断“是/不是”）。

## 文档架构规范（强制）

- 文档固定两层：`docs/public/`（正式输出）与 `docs/local/`（本地过程，不入库）。
- 根目录 `AGENTS.md` 只提供宏观规则与入口，不承载目录内细节清单。
- 目录内细则必须写在各自 `AGENTS.md`：`docs/public/AGENTS.md`、`docs/local/AGENTS.md`。
- 禁止在 `docs/` 根目录新增业务文档（`docs/README.md` 作为索引例外）。

## 前台文案与文档分层规范（强制）

- 前台/后台页面文案只保留用户操作必需信息（标题、字段、按钮、状态）。
- 禁止把开发过程说明写到页面里（例如“当前实现…/点击任一…/固定规则…/开发阶段…”）。
- 过程解释、设计取舍、阶段性说明：写在聊天回复或 `docs/local/`。
- `docs/public/` 只保留稳定结论与可执行步骤，不放会话过程与实现流水。
- 交付前若发现页面含“过程性说明文案”，必须清理后再提交。

## 无人干预的长期自主工作模式（你现在就在这个模式）

你将经常在“用户离线/睡觉”时工作。要求：

1) **每次开始先恢复上下文**（避免压缩失忆后跑偏）
- 先读：`docs/README.md`
- 必读：`docs/public/AGENTS.md`，并按其中指令读取 `docs/public/` 文档
- 若存在 `docs/local/AGENTS.md`，必须读取并按其中指令读取 `docs/local/` 文档
- 若 `docs/local/` 或 `docs/local/AGENTS.md` 不存在，可直接跳过 local 流程

2) **按闭环优先级推进**（不要做花活）
- 先确保：登录 → 下单 → 支付跳转 → `notify` 回调发货 → 订单查看
- 再做：前后台 UI 体验对齐与稳定性
- 最后才做：附加增强功能

3) **文档要跟着代码走**
- 文档更新细则以目录内 `AGENTS.md` 为准。
- 原则：过程信息先落到 `local`，稳定结论再提升到 `public`。

4) **持续可运行**
- 任何一步都要保证 `npm run check` 通过
- 前端改动后必须保证 `npm run build` 通过
- 重要里程碑后用 `npm run dev` 做一次最小启动验证

5) **小步提交（但目标完整）**
- 允许小步提交，但每一步都要指向最终形态，不做“临时中间态”长期遗留
- 在 `ldc-shop-lite/` 内定期 `git commit`（中文提交信息，描述清楚做了什么/为什么）
- 每次提交都能独立构建/运行，避免“堆一大坨最后一起炸”
- 在执行任何 `git commit`、打 tag 或创建 GitHub Release 前，必给须先将变更内容提交开发者审核，并得到明确确认
- 未经开发者审核确认，禁止直接提交、打 tag、推送发布或创建 Release

## 实施风格约束

- 不做过度防御性编程：只处理真实场景会触发的失败路径。
- 不牺牲体验换取“表面性能”：性能和体验同时达标。
- 不引入与当前任务无关的大改（例如重写支付闭环、改数据模型）。

---
> Source: [daoif/ldc-shop-lite](https://github.com/daoif/ldc-shop-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
