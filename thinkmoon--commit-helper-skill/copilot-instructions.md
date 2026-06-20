## commit-helper-skill

> 在提交前按当前仓库约定分析变更，优先按领域拆分为多个原子 commit，并生成可直接使用的 git commit message。


请基于当前仓库的变更，为即将执行的提交生成符合仓库约定的 commit message。

## 提交规范
- 优先使用约定式提交：
```txt
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```
- `type` 使用：`feat`、`fix`、`refactor`、`docs`、`test`、`chore`
- 提交说明使用中文描述**变更目的**，专业术语保留英文
- 保持原子化提交：一次 commit 只包含一个逻辑变更

## 你的任务
1. 先查看当前变更，优先读取 staged diff；如果没有 staged 变更，再看工作区变更
2. 先做“是否需要拆分提交”的判断，而不是默认把所有变更合并成 1 次提交
3. 按业务领域、子系统边界、变更目的评估是否属于不同原子变更；只要跨领域、跨模块且可以独立提交，就应拆分
4. 常见应拆分场景包括但不限于：服务端与客户端、算法与接口层、测试与生产代码、重构与功能修复、文档与代码、不同目录下互不依赖的业务变更
5. 如果变更其实服务于同一个目的，即使涉及多个文件，也应保持为 1 次提交，不要机械按文件数拆分
6. 对每个建议提交分别判断最合适的 `type`，必要时补充简短 `scope`
7. 直接产出可用于提交的 commit message；如果需要拆分，按提交顺序给出多条 message，并明确每条对应的文件/变更范围
8. 只有在用户已经提供 commit message 草稿时，才做规范性修正
9. 提交后记得先pull, 然后push

## 输出约束
- 以可直接提交的 commit message 为核心输出
- 不要直接执行 `git commit`
- 不要输出与提交规范无关的长篇说明
- `subject` 保持简洁，聚焦“为什么改”而不是流水账式“改了什么”
- 如果无法判断 `scope`，可省略 scope
- 如果建议拆分提交，输出格式优先为：`Commit 1` / `Commit 2` ...，每项包含“建议纳入的文件或领域”与对应 commit message
- 如果判断不需要拆分，也要明确说明“建议单次提交”的原因

用户补充背景：
`$ARGUMENTS`

---
> Source: [thinkmoon/commit-helper-skill](https://github.com/thinkmoon/commit-helper-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
