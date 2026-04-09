## agentprj

> A同学规则 - M1模型与微调主责，M4验证层辅责，规范模型服务、数据处理和测试行为


# A同学 AI 助手规则（M1 主责 / M4 辅责）

## 角色身份

你正在辅助 **A同学** 完成「杀类桌游 AI Agent 系统」的开发工作。A 的核心价值是确保大模型能输出符合桌游规范的技能文本，是整个系统的"语言能力"基石。

## 第一步：读取上下文

**每次对话开始时，必须先读取以下文件，再执行任何编码任务：**

```
主任务包：workflow/stage{当前阶段}/A同学/context.md
  → 先检查顶部修订记录，确认是否有新版本
接口契约（提供方）：workflow/interfaces/M1-M2-contract.md
服务注册表：workflow/service-registry.md
项目总目标：docs/项目定义书.md
```

如果 context.md 不存在或信息不完整，停止编码，先让管理者生成/更新该文件。

---

## Day 1 交付门控

**每个阶段的第一个工作日结束前必须完成以下事项，这是硬性要求：**

1. 完成 `services/m1_mock_server.py` 并确保 `/health` 端点返回 200
2. 更新 `workflow/stage{N}/A同学/status.md` 为 🟢 并注明 "mock server 已就绪"
3. 更新 `workflow/service-registry.md` M1 行的状态和 URL
4. `git commit -m "[M1] Day 1 mock server 就绪" && git push`

若 Day 1 结束时 mock 未就绪：
- 更新 status.md 为 🔴 并说明原因
- 下游成员（B同学）自动启用内联硬编码 mock 数据，不等待

---

## 服务架构规范

### 必须实现的端点（FastAPI，端口 8001）

```python
GET  /health          # 健康检查，返回 {"status": "ok", "module": "M1", "port": 8001}
GET  /contract-check  # 契约验证，返回一个符合 M1-M2 接口契约 schema 的示例响应
POST /generate        # 核心推理端点，接受 prompt，返回生成的技能文本
```

### Mock Server（Day 1 必须交付）

**在真实模型加载前，必须先提供 `services/m1_mock_server.py`**，返回硬编码的合法格式响应，使 B 同学可以立即开始联调：

```python
{
  "hero_name": "测试武将",
  "skills": [
    {"name": "测试技", "type": "锁定技", "desc": "【测试技】——锁定技，你摸牌阶段多摸一张牌。"}
  ],
  "status": "mock"
}
```

---

## 编码行为约束

### 测试优先
- ❌ 禁止在没有对应 pytest 用例的情况下提交功能代码
- 测试文件统一放在 `tests/test_m1_*.py`
- 最低覆盖要求：推理函数、数据清洗函数、格式转换函数各有独立测试

### 输出格式约束
- 所有生成结果必须符合 `workflow/interfaces/M1-M2-contract.md` 中定义的 JSON schema
- 技能名必须用【】包裹
- 技能描述必须包含触发时机（如"出牌阶段"、"回合开始阶段"）
- 禁止输出未定义的机制词汇

### 显存与算力容灾
- 模型加载时检测显存，若 < 16GB 自动启用 4bit 量化（bitsandbytes）
- 若量化后仍无法加载，切换为 1.5B/3B 蒸馏模型，并在日志中明确记录降级
- 微调效果评估：在 W4 结束前，如果微调输出格式错误率 > 30%，立即启动 Few-Shot 兜底方案

---

## 反馈行为约束

### 降级时的通知义务
当你自主执行降级决策时（如量化模型、切换蒸馏模型、启动 Few-Shot 兜底），必须：
1. 更新 context.md 第 10 章节的"降级决策记录"
2. 更新 `workflow/service-registry.md` 状态
3. git commit 并在 commit message 中标注 `[DEGRADED]`
4. 如果降级影响 `/generate` 的输出格式，在 context.md 第 10 章节填写"接口变更请求"

### 发现上游问题时
1. 先运行上游的 `/health` 和 `/contract-check` 验证
2. 若验证失败，在上游成员的 `status.md` 添加跨模块问题上报
3. 同时切换到 mock server 继续自身开发（不阻塞自己）
4. 在自己的 `status.md` 标注 🟡 并说明依赖

### 每次收工前
1. 更新 `workflow/stage{N}/A同学/status.md`（状态灯 + 今日完成 + 明日计划）
2. 如有进度变化，更新 context.md 第 8 章节进度追踪
3. `git commit && git push`

---

## 服务端口注册表（调用其他模块时使用）

| 模块 | 端口 | 主要端点 |
|------|------|---------|
| M1（我） | 8001 | `/generate`, `/health`, `/contract-check` |
| M2 RAG服务 | 8002 | `/search`, `/health`, `/contract-check` |
| M3 Agent服务 | 8003 | `/run`, `/state`, `/health` |
| M4 前端 | 8501 | Streamlit UI |

---

## 与他人的协作接口

### 我提供给 B同学（M2 接收）
- **交付物**：模型推理服务 `http://localhost:8001`
- **验收方式**：B 调用 `GET /contract-check`，验证响应格式符合契约
- **时间节点**：W1 结束前提供 mock 版本，W3 结束前提供真实版本

### 我需要验证 D同学 给了我什么（M4 → M1）
- D 向我传递用户确认后的特征词列表（人物设定 JSON）
- 我接收前需验证：字段包含 `hero_name`、`features`（列表）、`confirmed: true`

### 我辅助 D同学（M4 正则校验）
- 我需要提供标准术语词汇表：`data/valid_terms.json`
- 包含：合法机制词、合法技能类型、合法阶段名称

---

## 阶段性验收自查

在提交任何阶段成果前，运行以下检查：

```bash
# 1. 单元测试全通过
pytest tests/test_m1_*.py -v

# 2. 服务健康检查
curl http://localhost:8001/health

# 3. 契约格式验证
curl http://localhost:8001/contract-check | python -c "
import json, sys; d=json.load(sys.stdin)
assert 'skills' in d and isinstance(d['skills'], list)
assert 'qualitative_eval' in d
print('M1 契约验证通过')
"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chaojiwudidashuaige492)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chaojiwudidashuaige492)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
