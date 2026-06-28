## datalayer-studio

> <!-- BEGIN:git-rules -->

<!-- BEGIN:git-rules -->
# Always use Simplified Chinese as git commit message 
- 每次完成代码或项目文件修改后，判断这次变更是否适合单独做一个 git 提交；如果适合，就提交并推送到远端。
- 每次修改后判断这次变更是否需要重新 build 才能看到效果；如果需要，就在完成后重新 build。
<!-- END:git-rules -->

## Product Naming

- 这个 App 对外发布和讨论时统一使用英文名：`DataLayer Studio`。

## App Store Server Notifications

- 未来涉及 App Store Connect（ASC）的提交、元数据、TestFlight、构建、IAP、审核信息等操作时，优先使用已安装的 `asc` 操作，并先查找/加载对应的 `app-store-connect` skill；除非用户明确要求其它方式，不要默认走网页手工流程。
- App Store / TestFlight 构建号统一使用 `yyyyMMddNN` 格式，其中 `NN` 是当天从 `01` 开始的顺序号；例如 `2026062601` 表示 2026 年 6 月 26 日的第 1 个构建。
- App Store Server API / Server Notification 的 `JWSTransactionDecodedPayload.price` 是货币的 milliunits，展示价格前必须除以 1000。
- `currency` 是 ISO 4217 三字母货币代码；通知里优先展示为 `金额 CODE（中文货币名称）`，例如 `38 CNY（人民币元）`。
- 不要用 `currency` 推断 storefront；需要地区时读取交易里的 `storefront`。
- 本地 App Store Connect / App Store Server API 变量在 `.env.local`：`APP_STORE_API_ISSUER_ID`、`APP_STORE_API_KEY_ID`、`APP_STORE_API_PRIVATE_KEY_PATH`，以及 `APP_STORE_SERVER_API_*` 别名。不要在输出中打印密钥值或 signedPayload。

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---
> Source: [leeeboo/DataLayer-Studio](https://github.com/leeeboo/DataLayer-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
