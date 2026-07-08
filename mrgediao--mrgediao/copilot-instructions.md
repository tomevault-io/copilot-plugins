## mrgediao

> GitHub 个人首页仓库：`README.md` + `assets/svg/` 定制资源。当前视觉系统是 2026-07-03 定稿的「纸面版」。

# AGENTS.md — MrGeDiao/MrGeDiao

GitHub 个人首页仓库：`README.md` + `assets/svg/` 定制资源。当前视觉系统是 2026-07-03 定稿的「纸面版」。

## 设计系统（改动必须遵守）

- 面板底 `#FCFBF9`，描边 `#E8E4DD`，内嵌块 `#F6F4F0` / 描边 `#EBE7E0`
- 文字：`#1F2328` 主 / `#57606A` 次 / `#8A939E` 弱
- 全页唯一强调色：朱砂 `#D93B2B`——印章、标题方块、增长曲线、活动图、Email 徽章同一支红，不引入第二个强调色
- 宋体栈（`'Songti SC', STSong, SimSun, 'Noto Serif SC', serif`）只用于 banner 主文案那一句，其余系统 sans + `ui-monospace`
- 动效只用 SVG SMIL，静帧必须完整成立；当前仅 banner 三句格言轮播 + 光标
- 已试过并明确排除的方向，不要回退：深色面板贴浅色页面、霓虹发光与流动渐变、第三方 stats 卡 / 奖杯 / 贡献蛇、浅深两套资产、纯极简无视觉

## 页面结构

banner → 自述两段 → 作品（两张卡 + shields 动态徽章 + 描述）→ star 增长曲线 → 工具箱（skillicons + AI 工具链一行）→ 正在做 + 提交活动图 → Email 徽章

## 维护点

- `assets/svg/star-growth.svg` 是静态快照。更新：`python3 scripts/gen-star-growth.py`（依赖已登录的 gh CLI），脚本会拉最新 stargazer 数据并按本设计系统重绘。
- shields 徽章实时，无需维护；paper-reading-zh 的 release 徽章必须带 `include_prereleases` 参数，去掉会显示 invalid（shields 对该仓库的怪癖）。
- skillicons.dev 与 github-readme-activity-graph.vercel.app 是外部服务，URL 参数已调成纸面配色（`theme=light`；活动图 `bg_color=fcfbf9&line=d93b2b` 等），不要改回默认配色。
- SVG 里 CJK 文本宽度 ≈ 字号 × 字数，没有自动布局；改主标题长度要同步挪副标签的 x。
- 改完 SVG 至少渲染一遍再提交：`rsvg-convert -b '#ffffff'` 或 Chrome headless 截 900px 宽白底预览。

## 文案

- 中文为主，过「说人话」的去 AI 味标准（作者是该 skill 的作者，这页就是示范；规则见 MrGeDiao/shuorenhua）
- 事实红线：paper-reading-zh 的覆盖平台是 Codex、Claude Code、Claude Project、ChatGPT Project 四个，别漏 Claude Project
- 不写简历级细节（公司、任职、指标）；这页的目标是让访客对作者和作品产生兴趣
- 联系邮箱：zcoeus@protonmail.com

## 协作约定

- 以仓库所有者名义提交，commit message 用中文、`docs:` 前缀、不加 AI 署名
- 涉及视觉的改动先出渲染预览给作者确认再 push；纯数据刷新（如重跑增长曲线）可直接提交

---
> Source: [MrGeDiao/MrGeDiao](https://github.com/MrGeDiao/MrGeDiao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
