## opencode-copilot-account-switcher

> 本文件是仓库级流程约束，优先级高于一般执行习惯；它的职责是防止已明确出现过的流程漂移再次发生。

# 项目级流程护栏

本文件是仓库级流程约束，优先级高于一般执行习惯；它的职责是防止已明确出现过的流程漂移再次发生。

## Release Notes

- GitHub Release 正文必须以 `docs/release-notes-template.md` 为唯一模板来源。
- Release 正文必须包含：
  - 一句价值导语
  - `## 适合谁升级`
  - `## 你会看到的变化`
  - `## 升级方式`
- `## 升级方式` 必须给出带**明确版本号**的升级命令，不能只写 `latest` 或裸包名。
- 只有在确有必要时，才额外加入：
  - `## 实验性功能`
  - `## 注意事项`
- 不允许把 Release 正文写成单行 `Summary + Test Plan`。
- 不允许只罗列底层文件、命令或实现过程，而不说明用户会感受到什么变化。

## Release Verification

- 发版前必须有 fresh 验证证据，默认以当前仓库完整 `npm test` 为准。
- 不允许沿用旧测试结果、隔夜测试结果或“前面已经跑过一次”的历史输出。
- 如果当前版本的发版内容依赖 GitHub Release / npm publish 成功，只有在这些远端状态也被确认后，才算 release 完成。

## Release Chain

- 版本 bump、release commit、tag、push、GitHub Release 是同一条完整链路。
- 不允许只推 tag 就当 release 完成。
- 不允许只创建本地 release commit 就当 release 完成。
- 如果 GitHub Release 创建失败，必须显式修复并补齐这一步，而不是假设后续自动化已经接管。

## Template First

- 仓库里已经有稳定模板与历史修正文档时，优先接回现有模板，不重新临时发明新的 release 正文格式。
- 若发现模板与现实流程再次脱节，应先修流程约束，再考虑是否需要扩展模板本身。

## Scope Control

- 本文件只做顶层流程护栏，不替代已有 spec/plan。
- 更细的执行细节继续放在 `docs/` 中；这里不重复维护长篇操作手册。

---
> Source: [jiwangyihao/opencode-copilot-account-switcher](https://github.com/jiwangyihao/opencode-copilot-account-switcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
