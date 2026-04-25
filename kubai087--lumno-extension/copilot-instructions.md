## lumno-extension

> 当用户要求“打包插件上传商店”时，默认使用白名单打包，避免把开发文件带入发布包。

# AGENTS

## 发布打包规范（Chrome Web Store）
当用户要求“打包插件上传商店”时，默认使用白名单打包，避免把开发文件带入发布包。

### 1) 仅打包运行必需文件
- `manifest.json`
- `background.js`
- `newtab.html`
- `newtab.js`
- `options.html`
- `options.js`
- `blacklist-utils.js`
- `input-ui.js`
- `document-pip-picker.js`
- `hotkey-listener.js`
- `auto-pip-blacklist.js`
- `site-auto-pip.js`
- `youtube-auto-pip.js`
- `youtube-auto-pip-page.js`
- `_locales/`
- `assets/`

### 2) 必须排除
- `.git/`、`.github/`
- `.vscode/`
- `README.md`
- `AGENTS.md` 及其他 agent/流程说明文件
- 所有 `.DS_Store`
- 其他非运行必需文件

### 3) 打包命令（标准）
在仓库根目录执行：

```bash
mkdir -p dist
VERSION=$(node -p "require('./manifest.json').version")
ZIP="dist/lumno-store-v${VERSION}.zip"
rm -f "$ZIP"
zip -r "$ZIP" \
  manifest.json background.js \
  newtab.html newtab.js \
  options.html options.js \
  blacklist-utils.js \
  input-ui.js document-pip-picker.js hotkey-listener.js auto-pip-blacklist.js \
  site-auto-pip.js youtube-auto-pip.js youtube-auto-pip-page.js \
  _locales assets \
  -x "*.DS_Store"
```

### 4) 打包后校验（必须）
- 用 `unzip -l <zip>` 检查内容。
- 确认不包含 `.git`、`.github`、`.vscode`、`README.md`、`AGENTS.md`、`.DS_Store`。
- 确认 `manifest` 引用资源都在包内。

## 搜索逻辑一致性规范（必须）
- 任何涉及“搜索建议/排序/打分/匹配权重”的改动，必须同步检查 `background.js` 与 `newtab.js`，避免入口行为不一致。
- `newtab.js` 的输入建议默认通过 `chrome.runtime.sendMessage({ action: 'getSearchSuggestions' })` 复用 `background.js` 结果；若新增本地排序分支，必须与后台同权重策略并在 PR 说明中注明。
- 涉及 URL 路径关键词（如 `/release/`）或最近访问频次（`lastVisitTime`、`visitCount`、`typedCount`）的权重调整时，必须做双入口回归：普通网页浮层 + 新标签页浮层。

## 代码修改策略（必须）
- 现在改代码时允许做破坏性更新；如果旧结构已经明显阻碍实现、可维护性或一致性，不必优先维持“最小改动”表象。
- 默认尽量避免“补丁式更新”；与其在原有逻辑外层继续叠条件、叠兼容分支、叠局部修补，优先直接整理结构、替换不合理实现或重写相关片段。
- 做这类改动时，仍需自行控制影响范围，确保行为边界清晰，并避免误伤与当前任务无关的功能。
- 如果本次需求虽然完成，但局部代码已经出现明显的重构信号，收尾时应主动提出后续可做的重构建议，而不是等用户来问。

## 样式治理规范（必须）
- 后续写代码时，默认避免新增 `!important`，也不要把 `style.setProperty(..., 'important')` 当作常规实现手段。
- 优先使用稳定类名、CSS 变量、明确的承载层级，以及“外层保护壳 + 内层内容层”的分层方式解决样式优先级问题。
- `newtab.html`、`newtab.js`、`options.html`、`options.js` 这类自有页面，原则上不应再新增 `!important`；若必须新增，需要先证明普通层叠、变量或结构调整无法满足。
- 对注入路径（尤其 `background.js`、`input-ui.js`）保持保守：只有在宿主页面样式污染、几何定位同步、进入/退出动画保护这类明确高风险位点，才允许保留少量 `!important`。
- 任何涉及浮层容器、输入框壳层、tooltip、toast、站内搜索前缀、AI 特效承载层的样式改动，优先检查 `overflow`、`contain`、`transform`、`backdrop-filter` 与挂载层级，不要先用 `!important` 硬压。
- 若确实需要保留或新增 `!important`，必须把范围收敛到最小，并确认不会影响已有样式、加载时机与快捷键行为。

## 浮层隔离规范（必须）
- 普通网页浮层默认按“整块隔离”处理，不只隔离输入框；输入区、结果区、tooltip、prefix、mode badge、按钮等整块内容都必须放进扩展自有的隔离挂载层，默认不得直接暴露在宿主页的 light DOM 样式污染面里。
- 任何宿主页样式或其他插件注入样式，只要可能命中浮层内部元素（如 `input`、`button`、`div`、`span`、`li`、通配符、属性选择器、`:hover`、`:focus` 等），都应视为真实风险，优先通过 shadow root / 自有样式根解决，而不是继续局部补丁。
- 涉及浮层 UI 的改动时，默认检查“输入区 + 结果列表 + hover/focus/selected/empty state + AI mode 装饰层”整块是否仍完全受控，不能只验证输入框单点。
- 若当前改动无法保证浮层整块样式隔离，则不要提交为完成态；必须继续修到宿主页样式无法干扰为止。

## 注入图标稳定性规范（必须）
- 普通网页浮层、注入 tooltip、注入 toast、站内搜索前缀、AI mode 装饰层等注入 UI，默认不得把宿主页上下文里的字体图标链路当作稳定依赖；不能假设页面、shadow root 外层、其他插件注入仍会提供正确的 `font-face`、`::before` 内容或图标样式。
- 对上述注入 UI 的关键图标，优先使用扩展自有可控的本地 SVG runtime；若保留字体图标链路，只能作为兜底，不能作为主路径。
- 本地 SVG runtime 必须可追溯到上游图标源；同步上游时，优先通过脚本从官方包生成运行时产物，不要手工维护一份长期分叉的图标副本。

## 人工验证规范（必须）
- 默认不要在每次改动后都主动使用浏览器 DevTools、Computer Use、截图比对等人工验证手段反复手测；只有当用户明确要求“你现在去验证 / 复现 / 手动检查 / 抓包 / 看页面效果”时，才执行这类人工验证。
- 在未被明确要求做人工验证时，优先做代码级检查、语法检查、静态推断与最小必要的本地验证，不要把“每次改完都跑一遍网页手测”当作默认流程。
- 若问题本身必须依赖真实页面现象才能定位，可先说明需要该类验证，再在用户同意后执行；不要默认把手动网页验证升级为常规收尾步骤。

---
> Source: [kubai087/lumno-extension](https://github.com/kubai087/lumno-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
