## community-pr-review

> 审查和合并社区 PR 时的规范，确保原作者贡献可见性


# 社区 PR 审查与合并规范

## 合并策略

**合并社区贡献者的 PR 时，必须使用 `--rebase` 策略，禁止使用 `--merge` 或 `--squash`。**

- `gh pr merge <number> --rebase`
- rebase 会将原作者的 commit 平铺到主分支时间线上，每个 commit 保留原始 author 信息，在 GitHub 页面上直接可见
- `--merge` 会把原作者的 commit 折叠进 merge commit，GitHub 线性视图看不到原作者
- `--squash` 会把所有 commit 压成一个，原作者信息彻底丢失

## 在 PR 上追加修复

在原作者的 PR 分支上追加 commit 时：

- 直接 checkout PR 分支，以自己的身份提交新 commit
- 不要 amend 或 squash 原作者的 commit
- 不要修改原作者 commit 的 author 信息
- 合并后，时间线上应能看到每个 commit 各自的作者

## 审查流程

1. `gh pr diff <number>` 查看完整 diff
2. 对照代码库验证改动的正确性和完整性（同源逻辑是否遗漏、边界场景是否覆盖）
3. 有问题时：优先在 PR 分支上追加修复 commit，而非要求原作者返工
4. `gh pr review <number> --approve` 通过审查
5. `gh pr merge <number> --rebase` 合并

## 禁止

- **禁止用 `--merge` 合并社区 PR** — 原作者 commit 会被折叠，GitHub 页面上不可见
- **禁止用 `--squash` 合并社区 PR** — 原作者信息彻底丢失
- **禁止修改原作者的 commit**（rebase -i、amend、filter-branch 等）

---
> Source: [NoDeskAI/nodeskclaw](https://github.com/NoDeskAI/nodeskclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
