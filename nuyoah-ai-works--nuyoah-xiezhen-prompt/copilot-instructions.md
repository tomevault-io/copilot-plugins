## nuyoah-xiezhen-prompt

> - 本目录是唯一可编辑真源；独立 GitHub 仓库和知识站下载只接收白名单生成的发行快照。

# 南鸢写真提示词 Skill 约定

## 真源与发行

- 本目录是唯一可编辑真源；独立 GitHub 仓库和知识站下载只接收白名单生成的发行快照。
- 当前公开许可证为 MIT，版本从 `1.1.0` 开始。历史会员版 `1.0.0` 不改写、不覆盖，也不重新使用该版本号。
- 每次公开发布必须同步更新 `SKILL.md` 的 `metadata.version` 与 `CHANGELOG.md`，并通过 `scripts/release_public_skill.py --skill nuyoah-xiezhen-prompt` 构建。

## 行为保护

- 普通提示词请求默认不生图；只有用户明确要求才进入真实生成。
- 身份参考只决定人物身份，写真参考决定妆发、服装、场景、光线、事件与成像。
- 重做必须回到原始输入重新编译；生成图只用于诊断，除非用户明确要求编辑上一张。
- 没有图片生成工具时必须降级为 Prompt 与摄影方案交付，不得伪造生成结果。
- Eagle、个人知识库、私有路径和其他个人 Skill 不得成为公开包的必要依赖。

## 验证

- 修改后运行合同测试、YAO 结构校验、触发正反近邻评测与仓库检查。
- 公开构建必须通过白名单、安全扫描、版本与许可证门禁；正式发布还需干净真源、公开仓库和发行物回读。
- 结构测试不等于真实生图回归；涉及生图行为变化时，另做真实生成与原作对照验收。

---
> Source: [nuyoah-ai-works/nuyoah-xiezhen-prompt](https://github.com/nuyoah-ai-works/nuyoah-xiezhen-prompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
