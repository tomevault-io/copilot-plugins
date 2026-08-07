## zterfusion

> > 写给 AI 编码代理的项目说明。读完后你应当能直接上手改代码，无需再问项目背景。

# AGENTS.md — zterfusion

> 写给 AI 编码代理的项目说明。读完后你应当能直接上手改代码，无需再问项目背景。

## 项目概述

**zterfusion**（内部代号 / 注释中常称 "Rhythm Party"）是一个纯前端的本地多人同屏节奏对战游戏，像素风 UI。

- **玩法**：2–6 名玩家共用一台键盘（含鼠标按键）。配置界面顶部有 MODE 选择（`config.mode` 持久化）：
  - `casual`（休闲模式）：每个回合由一名玩家担任进攻方，其余为防守方；
  - `challenge-a`（挑战模式 A）：系统作为虚拟 P0（`SYS_PLAYER`，id `sys`，进攻方 idx 固定为 -1）随机生成节奏，所有玩家都是防守方复刻计分，系统不计分；`rounds` 仍表示局数；
  - `challenge-b`（挑战模式 B）：合作冲分。轮换进攻与休闲相同，但每个进攻小节音数 ≥ `challengeB.minNotes`，且节奏型不能与**自己上一段**进攻相同（按玩家各自记录 `prevRhythm[pid]`，换人互不误伤；节奏型按拍归一化再比较，加速后同一手法仍能判重）；违规小节不作废（防守照常复刻得分），只记 MISS，满 `maxStrikes` 立即结束（`forcedEnd`）；团队总分每 `scorePerStep` 分升 `bpmStep` BPM（上限 `maxBpm`，只在回合边界换速，避免小节时长突变）；TEAM 总分显示在头部 `mode-info-display`，最高分存 `localStorage` 键 `rhythm-party-best-b`（RESET 不清）；
  - `challenge-c`：预留占位（按钮 disabled，未实现）。
  - 偶数小节 = 进攻小节：进攻方按键打出节奏（音符自动吸附到勾选的 8/12/16/24 分音网格）；
  - 奇数小节 = 防守小节：所有防守方复刻进攻方的节奏；
  - 防守小节结束后延迟 80ms（`tolerance`）统一比对，每个进攻音与最近的防守音按误差计分；休闲模式攻守双方同时加分，挑战模式只有防守方加分。
- **技术栈**：零依赖、零构建的原生 Web 应用 —— HTML + CSS + 原生 JavaScript（ES5 兼容写法 + IIFE 模块模式），Web Audio API 做音频，Canvas 2D 做节奏轨道渲染，`localStorage` 持久化设置。
- **运行前提**：必须通过 HTTP 服务访问（`fetch('audios/')` 依赖 `python -m http.server` 自动生成的目录索引来枚举官方音效库），直接双击 `index.html`（file:// 协议）会导致音效库列表加载失败。

## 运行与构建

没有任何构建步骤、包管理器、依赖安装或测试框架（无 package.json / pyproject.toml / 测试目录）。

- **启动**：双击 `启动游戏.bat`（Windows），它打开浏览器并运行 `python -m http.server 8000`；或手动在项目根目录执行：
  ```
  python -m http.server 8000
  ```
  然后访问 http://localhost:8000 。
- **验证改动**：改动是即时的，刷新浏览器即可。根目录的 `_wb_*.json` 是浏览器调试（web-bridge 类工具）留下的请求快照（evaluate / screenshot / reload），可参考其中的 `Game.init()`、`location.reload()` 等调试手法，但这些文件不属于应用代码，不要依赖它们。

## 代码结构

只有 4 个 JS 文件，全部以全局 IIFE 单例（`const X = (function() { ... })()`）导出到全局作用域，按依赖顺序在 `index.html` 中以普通 `<script>` 标签加载（无 ES module、无 bundler）：

| 文件 | 导出的全局对象 | 职责 |
|---|---|---|
| `config.js` | `GAME_CONFIG` | 游戏数值配置（判定窗口、软封顶、准备小节数等），注释齐全，调平衡就改这里 |
| `audio.js` | `AudioEngine` | Web Audio：节拍器（采样 `tick.wav`，重拍 +200cent）、攻守音效、立体声声像（B 左 30% / C 右 30% / A 居中）、防守方按 1/√人数 衰减、自截断与小节边界清尾；`playSystemNote()` 供挑战模式系统进攻音用（不做自截断，一次可预约整小节，但登记到 `attackerSources` 统一清理） |
| `input.js` | `InputSystem` | 键盘/鼠标捕获、按键绑定弹窗、按键冲突检测、按键显示名格式化 |
| `game.js` | `Game` | 游戏核心：模式选择（`isChallenge()` / `currentAttacker()`）、状态机（`idle/countdown/playing/paused`）、回合与小节推进、`requestAnimationFrame` 主循环、计分（准确度 + 防守匹配 + 软封顶）、挑战模式随机节奏生成（`generateChallengeNotes` / `scheduleSystemBar`）、Canvas 渲染、玩家/设置管理与 `localStorage` 持久化 |

其他资源：

- `index.html`：三个全屏界面（`config-screen` / `game-screen` / `result-screen`）+ 绑键弹窗 + 暂停遮罩，靠 `.active` / `.hidden` class 切换。
- `style.css`：像素风样式，CSS 变量定义配色（`:root`），字体用 Google Fonts 的 Press Start 2P。
- `audios/`：官方音效库（`attack.wav`/`defend.wav`/`tick.wav` 是默认音，其余进音效库下拉框）。玩家也可上传自定义音效（仅存内存，刷新需重传）或用音效库音效（选择会持久化）。

## 关键架构细节（改代码前必读）

- **时间轴**：一切基于 `performance.now()`。`gameState.startTime` 在 READY 阶段设在未来，因此 `elapsed` 从负值走到 0 才正式开始；READY 长度 = `CFG.readyBars`（2）个小节。
- **小节角色**：`barIndex % 2 === 0` 为进攻小节。判定有 `tolerance`（80ms）的跨小节容错：小节末提前按归下一小节（offset 为负，音效预约到小节点播放），小节初晚按归上一小节（只记录不出声）。
- **计分时机**：防守小节结束后延迟 `tolerance` 再 `scorePair()`（容纳晚按几十毫秒的防守音）；时间耗尽不立即结束，置 `ending` 等当前一组攻守走完。**触发 ending 时所在的进攻小节允许打完**（`endingAtBar` 记录小节号，`onPlayerKeydown` 按容错路由后的落点小节比较，更后面的进攻小节才忽略）；收尾阶段计时器显示 `LAST`。
- **挑战模式调度**：系统节奏在进攻小节开始前 0.1s 由主循环生成并预约音效（`systemScheduledFor` 防重复；预约窗口只留 0.1s，避免暂停时整小节预约音漏进暂停）；卡顿错过窗口时 `onBarBoundary` 兜底补生成。
- **BPM 取值**：所有节奏计算（小节时长、网格吸附、准确度、节拍器、模式 A 生成）必须走 `effectiveBpm()` 而不是 `config.bpm`——挑战模式 B 用 `gameState.bpm` 覆盖实现加速，其它模式原样返回设定值。
- **音画一致**：进攻音符吸附到网格后，音效也按吸附点预约播放；取消未发声的预约音时必须同步删音符（`removeLatestEarlyNote`）。
- **音效库列表**靠 `fetch('audios/')` 解析目录索引 HTML 中的 `href`，这只在 `python -m http.server` 等生成目录索引的服务器下有效。
- **设置持久化**：`localStorage` 键 `rhythm-party-settings`，存配置和玩家（名字/绑定键/音效选择/音量 dB），自定义上传的音频文件本身不保存。

## 代码风格约定

- **注释主要用中文**（`config.js`、`audio.js`、`game.js` 内部注释均为中文），新代码注释也请用中文，并保持现有“解释为什么这么写”的注释密度——大量注释记录了踩过的坑（rAF 一帧延迟、音画一致、边界截断等），改动相关逻辑时同步更新这些注释。
- 模块用 IIFE 返回公开方法对象；不引入构建工具、框架或 npm 依赖。
- 数值/手感调优只改 `config.js` 的 `GAME_CONFIG`，`game.js` 内有缺失时的兜底默认值。
- UI 文案用英文（像素风），界面切换用 `.active`/`.hidden` class。

## 测试

没有自动化测试。手动验证路径：

1. `python -m http.server 8000` 起服务，浏览器开 http://localhost:8000 ；
2. 配置界面：双击 Enter/Space 快速开始（`quickStart: 'double'`）、绑键冲突提示、音效试听、READY 期间按数字键改拍数；
3. 游戏中：节拍器重拍、攻守小节交替、音符吸附、计分动画、ESC 暂停/恢复；
4. 结束后：结算界面名次与 REPLAY。

## 安全与其他注意事项

- 纯本地静态页面，无后端、无网络请求（除 Google Fonts），无密钥。
- 渲染玩家名字等使用字符串模板拼 `innerHTML`（玩家名输入被 `maxlength="8"` 限制），如果重构 UI 请注意不要引入 XSS。

---
> Source: [ZizhiLotika/Zterfusion](https://github.com/ZizhiLotika/Zterfusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
