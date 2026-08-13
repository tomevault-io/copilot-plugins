## git-workflow

> 用户说 PR/部署 dev 时参考——从 develop 建分支、PR 到 develop；不自动建 develop→main


# Git 工作流（按需参考）

- **PR/提PR**：从 develop 拉分支（feature/xxx 或 fix/xxx）→ 提交 → push → `gh pr create --base develop`。禁止直接提交 develop/main；PR 目标为 develop；**不要**自动创建 develop→main 的 PR，由用户手动操作。
- **部署 dev**：SSH 后执行 `./scripts/dev/deploy-from-git.sh`。**部署 prod**：见 production-deployment.mdc。
- **分支**：develop → Dev 环境（admin.dev.supplynexus.store）；main → Prod。提交前确认不在 develop/main 上直接改。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
