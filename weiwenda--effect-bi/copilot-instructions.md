## modal-select-dropdown

> 弹窗内 Select 下拉 — 避免裁切/层级/焦点问题（与「添加过滤条件」一致的手写遮罩模式）


# 弹窗内可搜索下拉（Select + portal）

## 问题现象

- 使用 **Radix `Dialog`** 时，`DialogContent` 常见 **`overflow-hidden`**，即使 `Select` 使用 **`portal`** 挂到 `document.body`，仍可能出现 **视觉裁切**、**焦点进不了搜索框**、或与 **焦点陷阱** 冲突。
- 在 **`DialogContent` 的 `className` 里加 `relative`** 会 **覆盖** 组件自带的 **`fixed` 居中**，模态框会 **跑偏/贴底**（Tailwind 后声明的 `position` 覆盖前者）。
- 把 **`portalContainer` 设在带 `overflow-y-auto` 的层内部**，下拉仍会被 **overflow 裁切**。

## 推荐做法（本项目已验证）

与 **`StaticFilterConfigZone`**「添加 / 编辑过滤条件」一致：

1. **不用 Radix `Dialog`**，改用手写层叠：
   - 外层：`fixed inset-0 z-50 flex items-center justify-center bg-black/30`，点击遮罩关闭。
   - 内层：白底卡片 `bg-white rounded-lg shadow-xl border ...`，`onClick={e => e.stopPropagation()}`。
2. **`Select` / `GravitinoTripleSelect` 仅使用 `portal`（挂到 `document.body`）**，**不要**传 `portalContainer`，除非你很清楚层级且容器 **无 overflow 裁切**。
3. 需要时用 **`src/components/ui/select.tsx`** 里已为 `portal` 分支设置的 **高层级（如 z-900/z-1000）**，保证下拉在遮罩 `z-50` 之上。
4. 补充 **Escape 关闭**、`role="dialog"` / `aria-labelledby` 等与 Radix 接近的无障碍习惯。

## 参考文件

- 范例：`src/components/query-explore/StaticFilterConfigZone.tsx`（过滤条件编辑弹层）
- 对齐范例：`src/components/etl-explore/EtlRuntimeDepsModal.tsx`（配置运行依赖）

## 若仍要用 Radix Dialog

- 可参考看板 **`DashboardFilterEditModal`**：`<Dialog modal={false}>` + Select **`portal`** 且 **不设 `portalContainer`**；但仍可能受 Dialog 内部样式影响，复杂场景优先用手写遮罩模式。

---
> Source: [WeiWenda/effect-bi](https://github.com/WeiWenda/effect-bi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
