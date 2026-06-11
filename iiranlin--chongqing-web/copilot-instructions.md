## chongqing-web

> **语言要求**：所有回复、思考过程及任务清单，均须使用中文。

**语言要求**：所有回复、思考过程及任务清单，均须使用中文。
**固定指令**：`Implementation Plan, Task List and Thought in Chinese`

1. Before writing any code, describe your approach and wait for approval. Always ask clarifying questions before writing any code if requirements are ambiguous.
2. If a task requires changes to more than 20 files, stop and break it into smaller tasks first.
3. After writing code, list what could break and suggest tests to cover it.
4. When there’s a bug, start by writing a test that reproduces it, then fix it until the test passes.
5. Every time I correct you, add a new rule to the current rules file so it doesn’t happen again.
6. 重庆旅游官网中的 GSAP 动画必须遵循本地 GSAP 技能规范，优先使用 gsap-core、gsap-timeline、gsap-scrolltrigger、gsap-performance 和 gsap-react 的推荐模式。
7. 重庆旅游官网的纸飞机必须与景点图片形成同一画面层级；轨迹只能在飞机飞过后出现，不能预先显示完整路径。
8. 重庆旅游官网的纸飞机必须初始化在起飞点位置，景点图片需根据分辨率自适应放大保证清晰，页面主要滚动特效需统一由 GSAP 驱动并强化赛博视觉。
9. 重庆旅游官网的纸飞机在向上反向滚动时必须调转机头方向，飞行轨迹也必须同步表现为反向回收或反向播放。
10. 重庆旅游官网的滚动背景必须优先使用与前景同步的模糊放大图片轨道，不得在滚动中使用抽色或高成本动态调色作为主背景。
11. 重庆旅游官网的主要视觉区块都必须使用 GSAP 滚动驱动动画形成连续浏览节奏，不能只把动画集中在纸飞机飞行段。
12. 重庆旅游官网首屏主标题和核心文案必须保持文旅官网气质，避免使用过度赛博、电影化或概念化的主标题表达。

1. Always respond in Chinese-simplified.
2. You MUST conduct your internal reasoning and thinking process entirely in Simplified Chinese.
3. 不允许写测试用例和测试代码

---
> Source: [iiranlin/chongqing-web](https://github.com/iiranlin/chongqing-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
