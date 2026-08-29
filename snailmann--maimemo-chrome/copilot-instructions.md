## maimemo-chrome

> 本文件定义本项目的开发约定，AI 助手与人类协作者都必须遵守。违反规范的代码不应被合并。

# AGENTS.md — 墨墨背单词 Chrome 扩展开发规范

本文件定义本项目的开发约定，AI 助手与人类协作者都必须遵守。违反规范的代码不应被合并。

---

## 1. 注释规范（强制）

### 1.1 中文注释范围

以下元素**必须**有中文注释：

- **关键变量**：跨函数使用、影响全局状态、含义不直观的变量（如 `currentVocId`、`mnemonicEditMode`、`mnemonicCardHistory`）。
- **关键方法**：对外暴露或承担核心职责的 function（如 `renderMnemonicCard`、`submitInlineEdit`、`queryMnemonic`）。
- **关键路径**：网络请求 URL、消息类型字符串、storage key、文件模块依赖路径。

### 1.2 注释写法

```js
// ✅ 推荐：说明"为什么"而非"是什么"
// 锁定卡片宽度，防止 inline 编辑替换 DOM 后 shrink-to-fit 重算导致面板变窄
const card = lockMnemonicCardWidth();

// ❌ 不推荐：复述代码
// 设置 card.hidden = false
card.hidden = false;
```

- 常量/枚举值旁注明来源（API 字段名、消息协议）。
- 复杂分支必须写"意图"注释，而不是描述控制流。

---

## 2. 架构与代码风格（强制）

### 2.1 顶层架构

本项目分为三层，依赖方向单向向下，禁止反向依赖或循环依赖：

```
content.js      ← 注入页面，负责 DOM 渲染与用户交互
    │  (chrome.runtime.sendMessage)
    ▼
background.js   ← Service Worker，负责网络请求、鉴权、消息路由
    │  (导入)
    ▼
maimemo.js      ← 墨墨 OpenAPI 客户端封装（纯函数，无 DOM/无 chrome.*）
    │
shared.js       ← 跨层共享的类型/常量/纯工具函数
```

- **content.js** 不直接发起网络请求，所有墨墨 API 调用必须经 `background.js` 中转。
- **maimemo.js** 不得引用 `chrome.*` 或 `document`，保持可在 Node 环境单测。
- **shared.js** 不得依赖任何上层模块。

### 2.2 消息协议

content ↔ background 的消息统一为：

```js
{ type: "CREATE_NOTE" | "UPDATE_NOTE" | "FETCH_NOTES" | ..., payload: {...} }
// 响应：
{ ok: true, ... } | { ok: false, userMessage: "...", code: "..." }
```

新增消息类型时：
1. 在 `background.js` 的 `onMessage` 中新增 `if (message.type === "XXX")` 分支。
2. 在 `maimemo.js` 中封装对应 API 函数（若函数涉及墨墨接口）。
3. 响应必须携带 `userMessage`（面向用户的中文提示）和 `code`（用于调试）。

### 2.3 代码风格

| 维度 | 要求 |
|------|------|
| 命名 | 变量/函数小驼峰；常量 `UPPER_SNAKE`；类名首字母大写 |
| 函数长度 | 单函数不超过 60 行；超过则拆子函数 |
| 状态管理 | 全局可变状态集中声明在文件顶部，带中文注释说明用途 |
| 错误处理 | 网络/IO 操作必须 `try/catch`，错误转为 `userMessage` 提示 |
| 防御性 | 外部输入（message.payload、API 响应）必须做存在性与类型校验 |
| 幂等 | 重复调用同一操作不应产生副作用叠加（如 `mnemonicSaveInFlight` 守卫） |

### 2.4 健壮性要求

- 所有 `chrome.runtime.sendMessage` 调用结果需判空：`if (!response || !response.ok)`。
- DOM 查询后使用必须判空（如 `const el = x(); if (!el) return;`）。
- 异步函数完成后必须清理临时状态（编辑态、loading 态、宽度锁定）。
- 事件监听器需在合适的生命周期移除，避免内存泄漏与重复触发。

### 2.5 可拓展性

- 新增助记类型（`note_type`）不应改动渲染逻辑，仅依赖数据驱动。
- 新增消息类型遵循 2.2 协议，无需改动中转框架。
- UI 样式与逻辑分离：结构/样式在 `content.css`，行为在 `content.js`。

---

## 3. 单元测试（强制）

### 3.1 覆盖范围

以下**必须**有单元测试：

- **核心纯函数**：`maimemo.js` 中所有请求封装、`shared.js` 中所有工具函数。
- **关键状态机**：编辑态进入/取消/提交（`startInlineEdit`/`cancelInlineEdit`/`submitInlineEdit` 的逻辑分支）。
- **消息路由**：`background.js` 中每个 `message.type` 分支的正常与异常路径。
- **边界条件**：空输入、超长输入、网络失败、token 缺失、响应缺字段。

### 3.2 测试技术栈

- 运行环境：Node + 原生 `node:test`（`node --test`），零额外依赖。
- 网络层：用轻量 mock（如 `sinon` 或手写 stub）替代真实 `fetch`/`chrome.runtime`。
- DOM 相关：用 `jsdom` 或 `linkedom` 提供最小 DOM 环境（仅测试需 DOM 的函数）。

### 3.3 测试文件约定

```
项目根/
  test/
    maimemo.test.js      ← API 封装测试
    shared.test.js       ← 工具函数测试
    background.test.js   ← 消息路由测试
    content-edit.test.js ← 编辑状态机测试（需 DOM 环境）
```

- 每个测试文件对应一个源码模块。
- 测试命名格式：`方法名__场景__期望结果`，如 `createNote__缺少vocId__抛出INVALID_VOC_ID`。
- CI 中必须通过 `node --test` 才允许合并。

### 3.4 运行方式

```bash
# 运行全部测试
npm test

# 等价的原生 Node 命令
node --test "test/**/*.test.js"

# 运行单个文件
node --test test/maimemo.test.js
```

---

## 4. UI 设计准则（强制）

UI 必须**主流、美观、现代化**，拒绝丑化审美。任何新增或修改的界面元素都需满足以下要求：

### 4.1 视觉语言

- **圆角与层次**：卡片/按钮统一使用 8–14px 圆角；通过阴影（`box-shadow`）与 1px 描边建立层次，避免生硬直角和纯边框堆叠。
- **留白与间距**：遵循 4px 基准网格（4 / 8 / 12 / 16…），元素间留白充足，杜绝拥挤。
- **毛玻璃质感**：浮层面板使用半透明背景 + `backdrop-filter: blur()`，与页面内容自然融合。
- **字体系统**：使用系统字体栈（`-apple-system, BlinkMacSystemFont, "Segoe UI", ...`），字号 11–13px，行高 1.5–1.6，保证可读性。
- **配色克制**：以中性灰蓝（slate）为主色，强调色（如墨墨橙 `#ad6420`）仅用于关键操作与状态，避免高饱和大色块。

### 4.2 交互反馈

- **悬停/点击**：所有可点击元素必须有 hover 态（背景/颜色微变）与 `:active` 按压态（轻微 `scale`）。
- **过渡动画**：状态变化使用 0.15–0.2s `ease` 过渡，禁止无动画的硬切。
- **加载与禁用**：异步操作期间按钮 `disabled` + 视觉降级；加载态用旋转指示器（不阻塞信息展示）。
- **焦点可见**：输入框/按钮获得焦点时显示清晰的 `outline` 或 `box-shadow` 环，保证键盘可达性。

### 4.3 布局与响应式

- **固定定位浮层**：弹层（助记卡片、操作条）使用 `position: fixed` + 极高 `z-index`，避免被页面层级遮挡。
- **边界感知**：浮层自动避让视口边缘（如超出底部则上移），不溢出屏幕。
- **内容自适应**：文本使用 `word-break: break-word` + `white-space: pre-wrap`，长内容内部滚动而非撑破容器。
- **尺寸约束**：面板设置 `min/max-width` 与 `max-height`，保证在不同内容量下都美观。

### 4.4 审美红线（禁止）

- ❌ 系统默认灰色按钮、无圆角直角、无交互反馈。
- ❌ 纯黑纯白高对比刺眼配色、荧光色、无序渐变。
- ❌ 文字溢出、重叠、被裁剪或撑破容器。
- ❌ 旋转/闪烁等令人眩晕的动画、过慢（>400ms）或突兀的过渡。
- ❌ 与页面原生控件样式冲突导致的"违和感"。

### 4.5 设计参考

以主流现代产品为标杆：Notion、Linear、Raycast、Apple 原生控件、Tailwind UI 默认风格。新增组件前先确认是否与既有 `content.css` 设计语言一致。

---

## 5. 提交前自检清单

- [ ] 关键变量/方法/路径是否有中文注释？
- [ ] 是否引入了新的全局状态？是否已集中声明并注释？
- [ ] 新增消息类型是否同时更新了 `background.js` 与 `maimemo.js`？
- [ ] 异步操作是否在 finally/成功/失败路径都清理了临时状态？
- [ ] 对应核心逻辑是否补充了单元测试？`node --test` 是否通过？
- [ ] 新增/修改的 UI 是否符合第 4 节设计准则（圆角、留白、交互反馈、边界感知）？是否触发审美红线？

---
> Source: [SnailMann/maimemo-chrome](https://github.com/SnailMann/maimemo-chrome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
