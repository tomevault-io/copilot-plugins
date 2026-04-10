## lensdate

> - 产品：约拍平台（支持 iOS/Android）

# 约拍平台 AGENTS 规范

## 项目概览
- 产品：约拍平台（支持 iOS/Android）
- 前端：Flutter
- 后端：Rust
- 角色：摄影师/摄影团队、平台管理员、用户

## 文档目录
- 所有项目文档统一放在 `docs/`。
- 主 PRD：`docs/PRD.md`。
- 规划类文档（如需）：`docs/task_plan.md`、`docs/findings.md`、`docs/progress.md`。

## 工作规范
- 所有文档只在 `docs/` 下维护，并同步更新路径引用。
- 修改 PRD 时，保留章节编号并在文首更新版本/日期。
- 新增文档需在文首写明用途说明（简短一句）。
- 项目沟通与文档输出统一使用中文。
- 依赖包统一使用“最新稳定版本”，避免 alpha/beta；版本变更需同步到依赖清单与文档。
- 前端项目需符合 lint 规范（以零警告为准），新增规则需同步到工程配置与文档。
- 后端需通过 `cargo clippy --all-targets -- -D warnings`，避免引入新增警告。
- 执行任何 `git add` / `git commit` / `git push` 前必须先获得用户明确同意。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MAQSOODAWANhaha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MAQSOODAWANhaha)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
