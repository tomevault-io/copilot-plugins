## xsearch

> - 单独的 `review`、诊断或状态查询保持只读，不自动修改、合并或发布。

# xsearch 项目 Agent 规则

## 默认交付闭环

- 单独的 `review`、诊断或状态查询保持只读，不自动修改、合并或发布。
- 当用户要求“修复”“更新”“完成交付”或“发布最新”时，除非用户明确指定 `PR-only`、`不发布` 或更低 Stop line，默认完成到：改动验证通过 → PR 合入 `main` → 发布下一个 patch 版本 → 等待 Release 资产成功 → 更新本机安装。
- 本机安装目标固定为 `/Users/envvar/.agents/skills/xsearch`。发布后必须读回：`bin/xsearch --version`、`SKILL.md`、`config.example.toml`、`references/runtime.md` 和 `references/leaf.md`。
- 发布前必须确认 PR CI 全绿、版本号与 tag 一致、工作区无不明改动。发布后必须确认 GitHub Release 的五个平台资产和 `checksums.txt` 均存在。
- 不因默认闭环获得删除旧 release/tag、改写 Git 历史、跳过失败门禁或发布 major/minor 版本的权限。失败时停在失败点，保留旧版本并报告。

---
> Source: [catoncat/xsearch](https://github.com/catoncat/xsearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
