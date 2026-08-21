## digital-hero-island

> 本项目当前版本为回合制数学冒险 V4，包含家长成长记录与里程碑奖励。新增玩法必须先更新设计规格和状态机测试。

# 数字勇者岛：协作规则

## 当前阶段

本项目当前版本为回合制数学冒险 V4，包含家长成长记录与里程碑奖励。新增玩法必须先更新设计规格和状态机测试。

## 项目约定

- 面向 6 岁儿童，优先使用数字、图形、语音和动画，避免依赖识字。
- 题目只包含数学；正式乘除法不在 V4 范围内。
- 普通加减题必须至少包含一个两位数，禁止恢复低价值纯个位数题。
- 题目提示不得自动播报；只保留主动喇叭和答错讲解。
- 主要交互适配 iPad 触屏，核心按钮触控区域不小于 64px。
- 图片、音效、字体和题目素材必须具备合法使用权；参考截图不可直接复用。
- 关卡、战斗、计分和道具规则以 `docs/superpowers/specs/2026-08-06-number-hero-island-turn-battle-design.md` 为准。

## 技术栈

- HTML、CSS、JavaScript ES modules
- Node 内置测试运行器
- Web Speech API 中文语音合成
- `localStorage` 版本化存档
- 原创 PNG 游戏素材

## 代码约定

- 纯状态逻辑放在 `src/game.js`，不得依赖 DOM。
- 界面、语音和动画编排放在 `src/app.js`。
- 样式放在 `src/style.css`；位图保持比例，不使用 `background-size: 100% 100%` 拉伸。
- 新增或修改状态逻辑前先写失败测试。

## 验证方式

- 自动化测试：`npm test`
- JavaScript 语法：`node --check src/game.js && node --check src/app.js`
- 浏览器全流程：地图、答题、战败、三连败、道具、成长记录、里程碑、存档和第 10 关
- 视觉验证：iPad 竖屏 820 × 1180，对照 `design-qa.md`

## 修改后反馈

每次修改后说明：修改文件、原因、影响范围、运行或验证方法，以及下一步建议。

---
> Source: [yuningceng518-cloud/digital-hero-island](https://github.com/yuningceng518-cloud/digital-hero-island) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
