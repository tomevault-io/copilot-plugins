## ai-writing-assist

> - 跨模块写正文只走 `modules.writing.facade`；不得直接依赖 Writing 的 model、repository 或 service。

# AGENTS.md — modules/imports

- 跨模块写正文只走 `modules.writing.facade`；不得直接依赖 Writing 的 model、repository 或 service。
- `ImportRecord` 只保存导入元数据，不保存正文原文；解析结果统一为 `list[dict{title, content}]`，
  新格式必须接入 `parsers.py` 的统一校验与分派。
- 上传文件必须校验内容签名并使用安全文件名，不得信任调用方路径。
- 不让导入记录长期停在 `processing`；成功、失败和空内容都必须落到明确状态。错误可读但不得
  泄露凭据、内部 URL 或原始路径。
- 深度导入的业务 LLM 只能消费持久化的 project execution snapshot，并通过 Project runtime
  恢复当前账户 Key；不得回退环境 Key。Context 与来源审计只走 Evidence facade/snapshot seam。
- 修改解析或上传时至少覆盖真实 happy path、空内容、非法类型/签名、大小限制和分页；任何对外
  宣称支持的文件格式都必须有未 mock 的真实文件验收。
- 表格迁移会话（ADR-0030）只在草稿期暂存有界单元格：采用或删除后必须清空 `rows_json`，
  回执只存 id/hash/被改字段原值与标签，不存正文；mapping/decisions 写入必须走 revision CAS。

---
> Source: [FengHuoLinShan/ai-writing-assist](https://github.com/FengHuoLinShan/ai-writing-assist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
