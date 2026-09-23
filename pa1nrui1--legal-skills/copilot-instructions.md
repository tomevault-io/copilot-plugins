## legal-skills

> When a legal task involves a document, report, memo, case compilation, pleading, evidence list, legal opinion, client-facing material, court-facing material, or any `.docx` output, classify the deliverable before writing substantive content:

# Repository Instructions

## Legal Formal Deliverable Hard Gate

When a legal task involves a document, report, memo, case compilation, pleading, evidence list, legal opinion, client-facing material, court-facing material, or any `.docx` output, classify the deliverable before writing substantive content:

1. `工作草稿`: internal draft only; may be Markdown; must be named with `草稿` or `未出稿审查` if not reviewed.
2. `律师内部报告`: internal strategy analysis; may include risk assessment and litigation suggestions; must still include source boundary, reading review, and legal verification when it cites materials, cases, or law.
3. `提交法院/客户的正式材料`: court-facing or client-facing; must follow the applicable legal Skill template, stay clean and professional, and include required source boundary records.
4. `Word正式交付物`: any formal `.docx`; must pass the full legal export chain before being called final or formal.

For every legal `.docx` or formal legal deliverable, stop and verify these gate items before export:

- Current matter matches `【自定义工作目录】/_系统记录/当前事项.md`; if it does not, switch/build the matter first or ask before continuing.
- The correct legal Skill has been selected, including whether the output is a 法官版、律师版、诉讼文书、证据目录、法律意见 or other type.
- Before writing substantive content, the corresponding legal template/reference file has been identified and recorded, with its key format requirements stated. After generation, run a reverse structure check against that template. For `证据目录`, the controlling template is `诉讼文书起草/templates/证据目录格式.md`; it must use grouped textual paragraphs with `第一组证据` and `证明目的`, and must not be output as a table unless the user expressly overrides this template.
- All cited case/material files have a `读取复查摘要`.
- All cited statutes, regulations, judicial interpretations, or current legal rules have a `法规校验摘要`.
- A `来源边界记录` exists and states verified, unverified, and output boundaries.
- Any user choice that changes scope, version, strategy, amount, claim, authorization, or whether to include/exclude negative cases is recorded in `用户确认记录`.
- Formal Word output has `draft.html`, `preflight-meta.json`, `draft_checked.html`, `出稿前审查报告.md`, and the report status is `PASS` or `FIXED_PASS`.
- Formal Word output is generated only through `法律文书模板与导出` (`html_to_docx.py`) and passes `health_check.py`.

Never treat “content completed” as “legal work completed.” A formal legal deliverable is complete only when the required matter routing, template confirmation, reverse structure check, source evidence files, preflight review, export chain, and health check are complete. Direct `pandoc md -> docx` export is allowed only for clearly labeled experiments or drafts, never for a formal legal deliverable.

---
> Source: [pa1nrui1/legal-skills](https://github.com/pa1nrui1/legal-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
