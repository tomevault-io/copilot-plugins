## dynamic-website-backgrounds

> 本目录里的每个背景都是独立单文件 HTML，主要用于 iframe 嵌入博客或网页。

# AGENTS.md

## 背景 HTML 维护约定

本目录里的每个背景都是独立单文件 HTML，主要用于 iframe 嵌入博客或网页。

### resize / layout 规范

所有背景页面都必须使用统一的 resize 调度模式：

1. 负责重算尺寸、重建 canvas、重建粒子数组的函数统一命名为 `layout()`。
2. 不要把 `layout()` 直接绑定到 `resize`。
3. 必须使用 `createResizeScheduler(layout)` 后再监听 resize。
4. 调度器必须满足：
    - resize 开始后会尽快执行一次；
    - resize 持续触发时，每隔固定时间最多执行一次；
    - resize 停止后必须再执行最后一次，保证最终尺寸正确。
5. 默认节流间隔使用 `160ms`，除非某个背景确实需要特别调整。
6. resize listener 使用 `{ passive: true }`。

推荐模板：

```js
function layout() {
	// 读取 window.innerWidth / window.innerHeight
	// 设置 canvas.width / canvas.height
	// ctx.setTransform(...)
	// 重建和尺寸相关的粒子、雨滴、星点等数组
}

function createResizeScheduler(callback, interval = 160) {
	let lastRun = 0
	let throttleTimer = 0
	let trailingTimer = 0

	function run() {
		lastRun = performance.now()
		callback()
	}

	return function scheduleLayout() {
		const now = performance.now()
		const elapsed = now - lastRun

		clearTimeout(trailingTimer)
		trailingTimer = setTimeout(run, interval)

		if (elapsed >= interval) {
			if (throttleTimer) {
				clearTimeout(throttleTimer)
				throttleTimer = 0
			}
			run()
			return
		}

		if (!throttleTimer) {
			throttleTimer = setTimeout(() => {
				throttleTimer = 0
				run()
			}, interval - elapsed)
		}
	}
}

const scheduleLayout = createResizeScheduler(layout)
window.addEventListener('resize', scheduleLayout, { passive: true })
layout()
```

## theme / dark mode 规范

所有背景页面都必须支持亮色 / 暗色两种模式，方便 iframe 外层页面通过 query 或 JS 控制。

1. HTML 直接打开时默认必须是亮色模式。
2. query 参数 `?darkdef=1` 表示页面初始就是暗色模式；没有该参数或参数不是 `1` 时保持亮色。
3. 页面必须暴露 `window.setDark(boolVal)`，语义与 `instructions/background.html` 保持一致：
    - 调用时先移除 `body` 上的 `dark` class；
    - `boolVal` 为真时再给 `body` 添加 `dark` class；
    - 允许 iframe 外层后续动态切换。
4. 推荐同时暴露 `window.setTheme(theme)`：
    - `theme === 'dark'` 时等价于 `setDark(true)`；
    - 其他值等价于 `setDark(false)`。
5. 初始化脚本必须尽早执行，推荐放在 `<body>` 起始处：

```js
window.setDark = (boolVal) => {
	document.body.classList.remove('dark')
	if (boolVal) document.body.classList.add('dark')
}
window.setTheme = (theme) => window.setDark(theme === 'dark')
window.setDark(new URLSearchParams(window.location.search).get('darkdef') === '1')
```

6. 样式建议使用 `body.dark` 或 `body:not(.dark)` 做主题差异，不要依赖系统 `prefers-color-scheme` 自动切换。
7. 每个背景原本的主要视觉效果必须完整保留在其中一个主题里：
    - 原本是亮色背景的文件，默认亮色必须尽量保持原样；
    - 原本是暗色背景的文件，`?darkdef=1` 暗色必须尽量保持原样。
8. 如果 canvas 绘制颜色和主题相关，必须让暗色 / 亮色切换后使用当前主题颜色；切换方法不能破坏动画循环。
9. `index.html` 里的每个卡片必须同时展示两个 iframe：
	- 上方亮色 iframe -> `${slug}.html`
	- 下方暗色 iframe -> `${slug}.html?darkdef=1`
10. 每个卡片提供两个预览按钮：
	- `预览亮色` -> `${slug}.html`
	- `预览暗色` -> `${slug}.html?darkdef=1`
11. `index.html` 的 `backgrounds` 数组必须使用当前简化结构：
	- `slug` 就是文件名去掉 `.html` 后的短横线标识，例如 `cherry-blossom`；
	- 不要再使用 `name` 属性；
	- 不要再使用 `file` 属性，文件路径统一由 `${slug}.html` 生成；
	- 不要再使用 `desc` 属性，卡片不显示描述文本；
	- `tag` 可以保留，用于卡片右上角分类标签。
12. 每个背景 HTML 的 `<title>` 必须等于对应 slug，例如 `cherry-blossom.html` 的 title 是 `cherry-blossom`。

### 性能约定

- 不要在 `resize` 高频事件里直接重建大量对象。
- canvas 动画循环只负责绘制和轻量更新；尺寸变化、数组重建放进 `layout()`。
- 粒子数量要按视口面积限制，并设置上限。
- `devicePixelRatio` 建议限制在 `2` 以内；重型纹理页可以更低。
- iframe 预览页应避免额外交互逻辑。

### 文件风格

- 保持单文件 HTML，不引入外部依赖。
- 背景页默认不显示文字卡片。
- CSS / JS 使用 tab 缩进，与现有文件保持一致。
- 新增背景后，要同步加入 `index.html` 的 `backgrounds` 数组，只添加 `slug` 和必要的 `tag`。

---
> Source: [xaidozy/dynamic-website-backgrounds](https://github.com/xaidozy/dynamic-website-backgrounds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
