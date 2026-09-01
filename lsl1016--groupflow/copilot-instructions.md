## groupflow

> 每次 `git commit` 必须随提交附带一条增量变更日志，并在相关时更新 `README.md`。该规约由

# GroupFlow

## 提交规约：变更日志（强制）

每次 `git commit` 必须随提交附带一条增量变更日志，并在相关时更新 `README.md`。该规约由
`.claude/hooks/changelog-guard.py`（PreToolUse 钩子）强制执行：若暂存区中没有
`docs/changelogs/` 下的文件，提交会被拦截。

提交前按以下步骤操作，使 README、变更日志与代码改动进入**同一个 commit**：

1. 如本次改动影响功能 / 用法 / 架构 / 部署，更新 `README.md`。
2. 在 `docs/changelogs/` 新增一个文件，文件名格式为 `年-月-日:时-分-秒:变更摘要.md`：
   - 时间戳：`date "+%Y-%m-%d:%H-%M-%S"`
   - 示例：`docs/changelogs/2026-06-30:14-23-05:新增提交变更日志钩子.md`
   - 内容建议：变更摘要、动机 / 背景、涉及文件或模块。
3. `git add README.md 'docs/changelogs/<新文件>'`，再执行 `git commit`。

> 注：文件名中的冒号 `:` 在 Linux/macOS 终端与 git 中可正常使用，但 macOS 访达会将其显示为
> `/`，且 Windows 不支持该字符。如需跨平台协作，可改用 `年-月-日_时-分-秒` 形式并同步更新本规约与钩子。

---
> Source: [lsl1016/GroupFlow](https://github.com/lsl1016/GroupFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
