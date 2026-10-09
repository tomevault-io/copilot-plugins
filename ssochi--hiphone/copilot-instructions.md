## hiphone

> 1. 纯函数（velocity.ts / rubberBand.ts / thresholds.ts / springFromVelocity.ts / projection.ts）必须有 100% 测试覆盖

# src/platform/gesture/ — 手势引擎

## 规范
1. 纯函数（velocity.ts / rubberBand.ts / thresholds.ts / springFromVelocity.ts / projection.ts）必须有 100% 测试覆盖
2. 阈值改动必须同步更新测试
3. 橡皮筋公式来源: WebKit UIScrollView 近似 `f(x) = (1 - 1/(x/c + 1)) * c`, `c = containerSize * 0.55`
4. **Projection (iOS scroll physics)**: `project(pos, vel) = pos + vel * 499` (normal rate 0.998) — 单一物理决策,取代"距离 OR 速度"两阈值。用于 dismiss 和 home gesture 的 commit 判定。velocity 单位 px/ms。
5. **Rubber-band 只用于超边界**: 主交互方向 (比如 card 向上 dismiss) 必须 1:1 跟手,不能 rubberBand,否则手感粘滞。只有在 "拖到边界之外继续拖" 的场景才用 rubberBand。

---
> Source: [ssochi/hiphone](https://github.com/ssochi/hiphone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
