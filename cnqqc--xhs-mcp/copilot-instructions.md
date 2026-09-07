## xhs-mcp

> - 要求每次修改完后,需要帮我格式化 Go 源码文件.

# Project Guidelines

##  本地开发规范

- 要求每次修改完后,需要帮我格式化 Go 源码文件.
- 测试过程中产生的脚本和build中间文件,如果没有必要,则删除.
- 所有的feature变更,都需要使用分支进行开发.
- 在我未同意之前, 你不能推送到远程.
- 我需要: 1.本地 review; 2.远程 PR review.
- 不要过度设计, 保持代码的简洁和易读.
- 使用中文注释，一定要简洁明了.专业名词可以用英文.

## 发版规范

- 发版=打语义化 tag `vX.Y.Z` 推上去（触发 tag-release.yml 构建 Releases 二进制），破坏性变更进 major。
- 本仓库没有 Docker 镜像发布流程（上游那条 workflow 用的是上游的 Docker Hub 账号，已删）。

## 与上游的关系

- 本仓库衍生自 [xpzouying/xiaohongshu-mcp](https://github.com/xpzouying/xiaohongshu-mcp) v2.5.0，整体沿用 Apache-2.0。
- 改上游文件时，记得同步维护 `NOTICE` 里的「被修改过的上游文件」清单（Apache-2.0 第 4(b) 条要求）。
- 新增原创文件时，同步加进 `NOTICE` 的「新增的原创文件」清单——那份清单是 `LICENSE-MIT` 的适用范围。
- `go.mod` 模块路径保持上游的 `github.com/xpzouying/xiaohongshu-mcp` 不动，别改。

## PR Review 重点

- 重点：PR 代码中如果出现大量的 JS 注入的行为，要检查一下是否是必须的，如果可以用 Go 的 go-rod 替代的话，则直接评论需要用 go-rod 行为替代。

---
> Source: [CNQQC/xhs-mcp](https://github.com/CNQQC/xhs-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
