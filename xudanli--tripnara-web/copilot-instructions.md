## semantic-colors

> TripNARA 语义色规范 — 产品面中性 + success/warning/error


# 语义色 · 产品 UI

## 产品面必须用

| 语义 | Tailwind / Token | 非 Tailwind（地图/SVG） |
|------|------------------|-------------------------|
| 绿（通过/安全） | `gate-allow` / `text-success` | `SEMANTIC_SUCCESS_HEX` |
| 琥珀（需确认） | `gate-confirm` / `text-warning` | — |
| 红棕（拒绝/危险） | `gate-reject` / `text-error` | `SEMANTIC_ERROR_HEX` |
| 信息/数据 | `text-muted-foreground` + 中性面 | `SEMANTIC_NEUTRAL_HEX` |
| 主 CTA | `bg-primary text-primary-foreground` | — |

共享类：`src/lib/semantic-ui-classes.ts`（`productInfoPanelClass` 等）  
地图品牌色：`src/lib/brand-map-colors.ts`  
Hex 常量：`src/lib/semantic-colors.ts`

## 产品面禁止

- `nara-glacier-*` / `nara-tundra-*` / `nara-lava-*` Tailwind 类
- `text-green-*` / `bg-red-*` / `text-blue-*` 等彩虹色阶
- 语义色铺底/描边（卡片/Banner 必须中性）
- 主按钮 `bg-blue-*` 或冰川蓝按钮

## 品牌/地图层（Layer B）

仅 `website/`、`odyssey-intake/`、`full-journey-map/`、`**/map/**` 可用 `brand-map-colors.ts` 硬编码 hex。

`nara-*` CSS 变量保留兼容；`nara-tundra` = success，`nara-lava` 未启用。

```tsx
// ❌ 决策产品面
<div className="bg-nara-glacier-muted text-nara-glacier-foreground" />

// ✅
<div className={productInfoPanelClass}>
  <Icon className={productInfoIconClass} />
</div>
```

## 验收

```bash
npm run check:semantic-colors
```

---
> Source: [xudanli/tripnara-web](https://github.com/xudanli/tripnara-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
