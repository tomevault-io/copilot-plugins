## bfb-gasifier

> 鼓泡流化床一维稳态模型的核心科学建模约束


# 科学建模核心规则集（BFB 气化炉）

## 1. 物理量单位强制规范

- 所有物理量变量在定义处必须带有单位注释，优先采用国际单位制。
- 仅在实现显式转换函数（如 `convert_bar_to_pa(p_bar: float) -> float`）时，才允许出现非 SI 单位。
- 在同一计算链路中严禁混用不同单位。

```python
# ✅ GOOD
T_gas: float = 1200.0  # [K]
P_bed: float = 2.5e6   # [Pa]

# ❌ BAD（缺少单位标注）
T = 1200
P = 25
```

## 2. 维度与张量校验

- 在 `calc_hydrodynamics()`、`calc_exchange()`、`calc_gas_balance()` 等数值核心函数中，必须对数组 / 张量的维度进行显式校验。
- 推荐使用 `assert`、`typing.Annotated` 或 `npt.NDArray` 类型提示约束 `cells` 数量、`species` 维度等关键 Shape。

```python
import numpy as np
import numpy.typing as npt

def calc_hydrodynamics(u_g: npt.NDArray[np.float64], eps_b: npt.NDArray[np.float64]) -> npt.NDArray[np.float64]:
    assert u_g.shape == eps_b.shape, "u_g 与 eps_b 轴向离散单元数量不一致"
    # ...
```

## 3. 数值稳定性优先

- 在实现动力学速率常数（如 Table 5.2/6.3 中的 Arrhenius 项）时，必须检查指数项是否存在溢出风险。
- 对 `np.exp()` 的输入使用 `numpy.clip` 进行裁剪，或在求和场景采用 log-sum-exp 等数值稳定技巧。

```python
import numpy as np

def arrhenius(A0: float, Ea: float, T: float) -> float:
    R = 8.314  # [J/(mol·K)]
    x = -Ea / (R * T)
    x = np.clip(x, -100.0, 100.0)  # 防止指数溢出
    return A0 * np.exp(x)
```

## 4. 引用溯源与文献标注

- 所有关键算法实现（例如 `K_bd` 交换系数、`u_br` 压力修正、气泡动力学方程等）必须在文档字符串或紧邻函数定义的注释中给出明确来源。
- 引用格式建议：`Ref: Hamel (1999) Gleichung 3.50` 或 `Ref: Hamel & Krumm (2001), Section 2.1`。

```python
def calc_kbd(...):
    """计算相间传质系数 K_bd

    Ref: Hamel (1999) Gleichung 3.50; Sit & Grace 混合模型
    """
    # ...
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dachou5224)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dachou5224)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
