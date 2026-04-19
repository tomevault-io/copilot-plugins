## agentprj

> D同学规则 - M4前端展示与验证层主责，M3 Agent辅责，规范UI、数值映射、正则拦截和测试行为


# D同学 AI 助手规则（M4 主责 / M3 辅责）

## 角色身份

你正在辅助 **D同学** 完成「杀类桌游 AI Agent 系统」的开发工作。D 的核心价值是构建用户可见的交互界面，并通过代码规则网关确保系统输出的规范性——包括数值映射和术语拦截。

## 第一步：读取上下文

**每次对话开始时，必须先读取以下文件，再执行任何编码任务：**

```
主任务包：workflow/stage{当前阶段}/D同学/context.md
  → 先检查顶部修订记录，确认是否有新版本
接口契约（接收方）：workflow/interfaces/M3-M4-contract.md
服务注册表：workflow/service-registry.md
项目总目标：docs/项目定义书.md
```

如果 context.md 不存在或信息不完整，停止编码，先让管理者生成/更新该文件。

---

## Day 1 交付门控

**每个阶段的第一个工作日结束前必须完成以下事项，这是硬性要求：**

1. 确保 `streamlit run app/main.py` 可成功启动（端口 8501）
2. 更新 `workflow/stage{N}/D同学/status.md` 为 🟢 并注明 "前端 UI 已就绪"
3. 更新 `workflow/service-registry.md` M4 行的状态
4. `git commit -m "[M4] Day 1 前端骨架就绪" && git push`

若 Day 1 结束时前端未就绪：更新 status.md 为 🔴 并说明原因。

---

## 编码行为约束

### 数值映射规则（核心约束，不可妥协）

```python
def map_health_points(qualitative_power: str) -> int:
    mapping = {"高": 4, "中": 4, "低": 3}
    return mapping.get(qualitative_power, 3)

def map_faction(faction_hint: str) -> str:
    mapping = {"主公型": "君", "辅佐型": "臣", "独立型": "民", "对抗型": "群雄"}
    return mapping.get(faction_hint, "民")
```

- ❌ 禁止在卡牌渲染中直接使用 AI 生成的数字
- ❌ 禁止在渲染函数中内联映射逻辑，必须通过独立函数

### 正则拦截网关
- 拦截后行为：返回 retry 信号给 M3，不直接展示给用户
- 正则拦截函数必须有 pytest 用例，合法+非法各至少 5 条

### 测试优先
- 测试文件放在 `tests/test_m4_*.py`
- 必须覆盖：数值映射函数、阵营映射函数、正则拦截函数
- ❌ 禁止在没有 pytest 用例的情况下提交上述三类核心函数

---

## 反馈行为约束

### 降级时的通知义务
当你自主执行降级决策时（如正则规则放宽），必须：
1. 更新 context.md 第 10 章节的"降级决策记录"
2. 更新 `workflow/service-registry.md` 状态
3. git commit 并在 commit message 中标注 `[DEGRADED]`

### 发现上游问题时
1. 先运行上游 `/health` 和 `/contract-check`
2. 若失败，在上游成员 `status.md` 添加问题上报
3. 使用内联 mock 数据继续 UI 开发，不阻塞自己

### 每次收工前
1. 更新 `workflow/stage{N}/D同学/status.md`（状态灯 + 今日完成 + 明日计划）
2. 如有进度变化，更新 context.md 第 8 章节
3. `git commit && git push`

---

## 服务端口注册表

| 模块 | 端口 | 主要端点 |
|------|------|---------|
| M1 模型服务 | 8001 | `/generate`, `/health`, `/contract-check` |
| M2 RAG服务 | 8002 | `/search`, `/health`, `/contract-check` |
| M3 Agent服务 | 8003 | `/run`, `/state`, `/confirm`, `/health` |
| M4（我） | 8501 | Streamlit UI |

---

## 与他人的协作接口

### 我需要验证 C同学 给了我什么（M3 → M4）
- C 提供 Agent 服务 `http://localhost:8003`
- 接入前运行 `/health` + `/contract-check`
- 若 C 未就绪，使用内联 mock 数据

### 我辅助 C同学（M3 前端事件串联）
- 实现"确认/修改特征词"按钮事件
- 将确认后特征词 POST 到 `http://localhost:8003/confirm`
- 实现打字机效果

### 我需要 A同学 提供（M1 → M4 术语词汇表）
- A 提供 `data/valid_terms.json`
- 若 A 未提供，使用硬编码基础词汇表

---

## 阶段性验收自查

```bash
pytest tests/test_m4_*.py -v
python -c "
from src.m4.validators import map_health_points, map_faction, validate_skill_text
assert map_health_points('高')==4 and map_health_points('低')==3
assert map_faction('主公型')=='君'
assert validate_skill_text('伤害值为100')==False
assert validate_skill_text('【观星】——回合开始阶段，你可以观看牌堆顶的X张牌。')==True
print('M4 全部验证通过')
"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chaojiwudidashuaige492) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
