## learning-by-coding-skill

> Trigger right after writing a non-trivial code unit (function, class, file, meaningful diff) to attach a two-card inline explanation that helps the user build their programming knowledge base. Only fires on agent-authored code in the execution phase. Skip for trivial edits, user-authored code under review (use questioning voice instead), or when the user says go fast.


## learning-by-coding

写完一段非平凡代码后，紧跟着附上**两段卡片**讲解。讲解嵌在代码下方，不是回答末尾的长文。

定位：本 skill 只服务**局部代码单元**的迭代期讲解，不做项目骨架蓝图（那是 brainstorming / feature-dev 的事）。

---

### 触发与跳过

**触发**：agent 自己刚写完一个**可交付局部代码单元**（函数 / 类 / 一次有意义的修改），且认知负荷非平凡。

**跳过（不讲解）**：
- 用户明说"快" / "别讲解" / "跳过 learning"
- 改 typo / 改一行 / 改变量名 / 调配置值
- 同一概念已讲过的相似代码（同文件第 N 个同类函数）→ 只点差异，不复述机制
- **代码不是 agent 写的**：
  - 用户写的、agent 在 review → 切**提问语气**（"你这里用 X 是因为...？"），不要套讲解模板说教
  - explorer agent 找到的他人代码 → 讲架构惯例和约定，不是"这段做了什么"

**复杂度判断（看认知负荷，不看行数）**：
- 引入新概念 / 非显然副作用 / 跨模块边界 / 时序或并发依赖 → 给完整两段
- 纯线性调用链 / 纯字段映射 / 显然的胶水 → 只给 📌 一句话，🔧 整段省

---

### 内容契约：两段卡片

#### 📌 意图与定位（由大到小四层；简单代码可省略后面层）

| 层 | 一句话回答 |
|---|---|
| ① 业务意图 | 用户/系统因此得到什么 |
| ② 项目位置 | 在哪个子系统/模块（**明确提模块名**） |
| ③ 调用链   | 被谁调用 / 调用谁（直接上下游） |
| ④ 联动模块 | 隐式依赖、副作用对象、跟谁交互 |

> ④ 是防"项目一大就忘了模块联动"的关键，**别省**。

#### 🔧 机制与决策（由浅入深四拍；拍 1 必给，拍 2-4 视情况）

| 拍 | 任务 |
|---|---|
| ① 现象 | 先指代码具体行/动作做了什么——**不给术语** |
| ② 命名 | 把这个动作归一个名（**术语首次出现在这里，加粗**） |
| ③ 坐标 | 这个术语属于哪个知识族（邻居是谁） |
| ④ 决策 | 为什么选它，不选 Y/Z（tradeoff，**核心学习价值**） |

> **拍 1 必须先于拍 2**。读者必须先看到行为再贴标签，否则术语永远飘在空中。

**字数（软上限）**：
- 📌 ≤100 字
- 🔧 ≤200 字
- 真复杂的核心逻辑允许溢出，但**拍 1（现象）永远不能省**
- 超字数压缩顺序：先压拍 3-4，后压拍 1-2；📌 先压 ④③，保 ①②

---

### 对话级收尾：🧭 知识坐标

**每完成一组可交付任务时**（不是每段卡片）追加一次：

```
🧭 知识坐标
本次涉及：[具体技术点]
属于：[上层知识族]
延伸：[1 个相关进阶点]
```

不要每个代码单元都来一次（噪）。

---

### 完整示例

```python
def validate_order(order: Order) -> Result:
    if not order.items:
        return Result.fail("EMPTY_ORDER")
    if order.total > user.credit_limit:
        return Result.fail("CREDIT_EXCEEDED")
    return Result.ok(order)
```

**📌 意图与定位**
- **业务意图**：订单提交前的准入校验，过不了就拒绝下单。
- **项目位置**：电商下单子系统 / `core` 业务规则引擎层。
- **调用链**：被 `submit_order()` 调用 → 通过后进 `inventory.reserve()` 扣库存。
- **联动模块**：隐式依赖 `user`（拿信用额度，全局态，未来宜改参数注入）；不读 DB、不发消息，纯函数。

**🔧 机制与决策**
- **拍 1 · 现象**：每个 `if` 都是"条件不满足 → 立刻 `return Result.fail(...)`"，把坏分支挡在前面，最后只剩一行主路径 `Result.ok(order)`。
- **拍 2 · 命名**：这种"入口处先把非法情况一一挡掉，不进主逻辑"的写法叫 **Fail-fast**（或 Early Return）。
- **拍 3 · 坐标**：属于「错误处理风格」这一族，邻居有 exception-driven、Result/Either monad、defensive programming——同一问题（怎么传错误）的不同流派。
- **拍 4 · 决策**：
  - 不用 `raise`：校验失败是**预期分支**不是异常，上层要转成 4xx；raise 会逼上层每处 try/except。
  - 不用 `Optional[Order]`：失败要带**原因**（`EMPTY_ORDER` vs `CREDIT_EXCEEDED`），`None` 丢语义。
  - 不用嵌套 if/else：early return 让主路径始终在最浅缩进。

---

### 反例库（❌ 不要这样写）

**❌ 反例 1：术语飘在空中**
> 🔧 这个函数采用了 Fail-fast 模式，提高了代码健壮性。

问题：直接抛术语没指代码，读者不知道"哪几行"是 Fail-fast。**必须先有拍 1（现象）**。

**❌ 反例 2：白话只是术语的同义复述**
> 🔧 采用 Fail-fast 模式。→ 通俗说就是"快速失败"。

问题：把英文翻成中文不算解释。**通俗描述要讲行为，不是同义词**。

**❌ 反例 3：把 README 塞进卡片**
> 🔧 ... 使用方法：调用 validate_order(order)。性能：O(1)。TODO：未来支持异步。已知限制：不支持跨币种...

问题：两段卡片只承载意图/定位/机制/决策。**使用指南/演进局限/依赖清单是 README 内容，不是 learning 内容**。

**❌ 反例 4：给用户代码套讲解模板**
> 用户：[贴代码] / agent：📌 这段做了什么...

问题：变成班主任改作业，伤合作关系。**用户代码切提问语气**，让用户先表达。

**❌ 反例 5：所有讲解堆在末尾**
> 代码 → 代码 → 代码 → ... → [末尾长篇讲解]

问题：违反"代码与讲解交替"。**每个可交付单元紧跟卡片**，不是攒到最后一次性输出。

---

### 与 skill_routing 的协作

| 流程阶段 | learning-by-coding 是否触发 | 形态 |
|---|---|---|
| brainstorming / writing-plans / 纯设计 | ❌ | 没代码可讲 |
| 执行（agent 写代码） | ✅ | 每个可交付单元后两段卡片 |
| code-reviewer agent 输出 | △ | 只在 reviewer 提到用户没接触过的概念时补一段 🧭 坐标 |
| systematic-debugging | ✅ 但变形 | 讲"为什么之前不工作 + 这个 fix 工作的原理"，不讲"这段做了什么" |

---

### 输出格式硬约束

1. **代码块只在讲解上方出现**：📌🔧 段内引函数名 / 变量名 / 行号即可，**不要再贴代码块**
2. **emoji 作为视觉锚点**：必须用 📌🔧🧭 帮用户扫描层级
3. **拍 1 · 现象**：用"看这几行..." / "每个 if 都..." / "函数先 X 再 Y" 这种**指代式**语言，**不要术语**
4. **拍 2 · 命名**：术语第一次出现，**加粗**
5. **语言**：简体中文为主，技术术语英文保留（与全局 personality 一致）

---
> Source: [EugeneWEI2022/learning-by-coding-skill](https://github.com/EugeneWEI2022/learning-by-coding-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
