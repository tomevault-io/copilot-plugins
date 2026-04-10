## eqp-parameter-spec-api

> <!-- OPENSPEC:START -->

<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# 重要
- 測試 flask api server 要另外開 terminal 啟動 flask server
- 完成 api server 後，請記得更新 README.md 裡面的 api 文件說明
- 開發進度 100% 後，MUST 進行 BDD 測試腳本撰寫，並確保所有測試皆通過，才能進行 code review 與合併
- BDD 測試腳本撰寫完成後，請更新 README.md 裡面的 BDD 測試說明
- API 測試 MUST 使用 Playwright MCP 進行測試腳本撰寫
- API 測試案例 MUST 遵守 GIVEN-WHEN-THEN 格式撰寫
- API 測試案例 MUST 覆蓋**必要的**功能與**重要的**邊界情況；不需要覆蓋所有可能的情況；請專注於關鍵路徑和常見使用情境

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuanhsunliu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yuanhsunliu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
