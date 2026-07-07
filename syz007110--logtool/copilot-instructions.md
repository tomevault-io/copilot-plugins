## 10-vue-element-plus

> - 默认 `<script setup lang="ts">`

## Vue SFC
- 默认 `<script setup lang="ts">`
- 页面负责组合，业务逻辑放到 composable/service
- 文本走 i18n，不写死

## Element Plus 组件
- 表格：默认 el-table，列优先 formatter/slot，操作列按钮统一 `type="text"`
- 查询：需 loading/空状态/错误提示，大数据走分页/后端筛选

## 样式规范
- **语义优先**：用组件 props（type/status/variant），不用 CSS 解决
- **Token 定值**：全局样式通过 `design-tokens.css`，不重复定义
- **CSS 限制**：仅用于 Token/布局/业务个例，禁止覆盖 `el-*` 内部样式
- **Pattern 优先**：重复结构封装 Pattern 组件（≥3 次使用）
- **Pages 职责**：只组合组件，不定义系统级样式

---
> Source: [syz007110/logtool](https://github.com/syz007110/logtool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
