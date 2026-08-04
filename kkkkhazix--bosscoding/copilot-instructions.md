## bosscoding

> 给 AI 协作项目的开发流程地基：`npx bosscoding init` 一条命令装好规则文件、守卫、CI 质检口、决策档案与技能，任何 coding agent 通吃（守卫数量是会变的状态，跑 `node bin/bosscoding.mjs check` 看）。纯 Node ESM，运行时零第三方依赖。

# BossCoding 项目规则

给 AI 协作项目的开发流程地基：`npx bosscoding init` 一条命令装好规则文件、守卫、CI 质检口、决策档案与技能，任何 coding agent 通吃（守卫数量是会变的状态，跑 `node bin/bosscoding.mjs check` 看）。纯 Node ESM，运行时零第三方依赖。
包 https://www.npmjs.com/package/bosscoding ｜ 仓库 https://github.com/KKKKhazix/BossCoding （公开，MIT）
线上是哪个版本跑 `npm view bosscoding version` 看——版本号是会变的状态，不写死在本文件里。

本文件是唯一的规则真身；CLAUDE.md 只是指过来的门牌，改规则只改本文件。
机器守卫：`npm run preflight`（单测＋`node bin/bosscoding.mjs check`）。

## 导师模式

老板是产品经理，不是程序员。默认先给结果，常规回复尽量在 5 行内；只有需要老板理解新方案或拍板时才展开。

- 新术语第一次出现时，用半句话说明它在本项目里是什么；不贴大段代码、日志或报错。
- 需要展开时，讲清「改什么、为什么、效果、风险」；复杂方案只用一个贯穿比喻。
- 汇报顺序：「影响 → 结论与行动 → 需要老板决定的」；没有决定项就直接结束。
- 数字换成老板能判断的价格、时间或人数。

## 干活流程

- 一个任务一个 PR，默认 Draft；push 前跑 `npm run preflight`；禁止直推 main。
- 转 Ready 前跑 `node bin/bosscoding.mjs merge` 问一句轮不轮得到——并行干活，串行合并。
- 多个 agent 并行时各开独立工作区（`git worktree add`），主工作区只留在 main 上；有 git hook 盯着。
- CI 红了默认响应是撤销或修复，不是加新检查。
- 发版本：改 `package.json` 版本号走 PR 合并，老板确认后 `gh workflow run publish.yml` 触发。走可信发布，不用令牌也不用验证码；本机 `npm publish` 只是应急退路。
- 合并不等于已发布：版本号进了 main 而没人触发 publish，npm 上就一直是旧版。发包是红线不能自动，所以改版本号的 PR 正文必须写明谁来触发；`npm view bosscoding version` 是唯一真实状态。

## 知识只有三个所在

| 想知道什么 | 去哪 |
|---|---|
| 机器允不允许这么写 | `node bin/bosscoding.mjs check` 与 `node --test`——规则真身是守卫和测试 |
| 当初为什么这么定 | `docs/decisions/`——只追加、不修改 |
| 现在实际什么状态 | 跑命令看真实输出，不要读文档猜 |

写作纪律：本文件只写「做什么」，「为什么这么定」一律留在 `docs/decisions/`。想加规则先问三遍——能写成守卫吗？是一条裁决吗？是跑命令能看到的吗？三个都不是就别写。

## 红线（先说明并等老板确认，再动手）

- 发包（触发 publish workflow 或本机 `npm publish`）、仓库转公开、以老板名义对外发布任何内容——这三件是本项目的「上线」，一律先确认。
- 任何花钱的操作；动权限、密钥、账号设置。
- 密钥、token 永不进代码，只放 `.env`（守卫盯着）。
- 违反后伤到谁：框架的用户是拿它当地基的小白，一次坏发布会同时伤到所有装了它的项目——发布纪律比功能进度重要。

## 事故与立法（框架自身的立法纪律）

- 运行时零第三方依赖：筹备队必须在任意网络环境一条命令跑通，依赖树越深失败面越大。
- `npx bosscoding update` 与任何更新机制永不改写用户的 AGENTS.md／CLAUDE.md——规则是老板的规则。
- 新增守卫的前提：挂着一次真实事故，且误报经实测收敛。误报多的守卫会被用户关掉，比没有更糟。
- 模板与文案改动同样走 PR 与守卫；本仓库必须永远能被自己的 `node bin/bosscoding.mjs check` 检查通过（吃自己的狗粮）。

---
> Source: [KKKKhazix/BossCoding](https://github.com/KKKKhazix/BossCoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
