## ai-benchmark

> 测评 AI 数学能力的 Benchmark 平台。通过 OpenRouter API 调用 AI 模型解题，裁判 AI 提取答案并评分。

# AI 数学 Benchmark 系统 - 交接文档

## 项目概述

测评 AI 数学能力的 Benchmark 平台。通过 OpenRouter API 调用 AI 模型解题，裁判 AI 提取答案并评分。

---

## 技术栈

| 组件 | 方案 |
|------|------|
| 前端 | 单一 HTML 文件（KaTeX 渲染 LaTeX） |
| 后端 | Python FastAPI |
| 数据存储 | JSON 文件 |
| API | OpenRouter（API Key 已固定在后端） |
| 裁判模型 | 默认 `openai/gpt-5.2`，可切换 |

---

## 项目结构

```
数学benchmark/
├── index.html              # 前端页面（含LaTeX渲染）
├── server.py               # 后端服务（API Key已内置）
├── requirements.txt        # Python依赖: fastapi, uvicorn, httpx
├── start.command           # Mac双击启动脚本
├── AGENTS.md               # 本文档
├── data/
│   └── questions.json      # 题库（15道数学竞赛题）
└── results/
    └── eval_*.json         # 测评结果存档
```

---

## 当前题库

15道高难度数学竞赛题，涵盖：多元积分、矩阵特征值、群论、数论、代数几何、级数极限

**评分规则：**
- 第1-12, 14-15题：每题 **1分**，答案必须完全正确
- 第13题：**3分**，答案允许 ±1 容差
- 总分：**17分**
- 无部分分：对就满分，错就0分

---

## 数据结构

### 题目格式 (`data/questions.json`)
```json
{
  "id": "q001",
  "question": "题目内容（支持LaTeX）",
  "answer": 130,
  "score": 1,
  "tolerance": 0
}
```

### 评判结果
```json
{
  "question_id": "q001",
  "model_final_answer": 130,
  "correct_answer": 130,
  "tolerance": 0,
  "is_correct": true,
  "earned_score": 1,
  "reason": "答案正确: 130"
}
```

---

## API 接口

Base URL: `http://localhost:8000`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/questions` | 获取题库 |
| POST | `/api/questions` | 新增题目 |
| PUT | `/api/questions/{id}` | 修改题目 |
| DELETE | `/api/questions/{id}` | 删除题目 |
| GET | `/api/models` | 获取可用模型列表（无需参数） |
| POST | `/api/evaluate` | 开始测评 |
| GET | `/api/results` | 获取历史结果 |
| GET | `/api/results/{id}` | 获取详情 |

### POST /api/evaluate 请求体
```json
{
  "tested_models": ["openai/gpt-4o", "anthropic/claude-sonnet-4"],
  "judge_model": "openai/gpt-5.2",
  "question_ids": ["q001", "q002"]
}
```

---

## 裁判 AI Prompt

裁判AI只负责从回答中提取最终数字答案：

```
你是一个数学题答案提取专家。请从被测AI的回答中提取最终的数字答案。

【题目】{question}
【被测AI的回答】{model_response}

【任务】
1. 只提取最终答案，不是中间计算结果
2. 答案应该是一个整数或小数
3. 优先提取"答案是"、"最终答案"等标注

【输出格式】
{"extracted_answer": <数字或null>}
```

后端根据 `tolerance` 字段判断正误：
- `|extracted - correct| <= tolerance` → 正确，得满分
- 否则 → 错误，得0分

---

## 前端功能

### Tab 1: 题库管理
- 题目列表（LaTeX渲染）
- 增删改查
- JSON批量导入

### Tab 2: 开始测评
- 被测模型：列表选择 或 手动输入
- 裁判模型：列表选择 或 手动输入（默认gpt-5.2）
- 勾选题目后开始测评

### Tab 3: 测评结果
- 历史记录列表
- 多模型对比表格
- 每题详细评判（可展开查看完整回答）

---

## 启动方式

```bash
# 方式1: 双击 start.command

# 方式2: 命令行
cd 数学benchmark
pip install -r requirements.txt
python -m uvicorn server:app --host 0.0.0.0 --port 8000
# 访问 http://localhost:8000
```

---

## 注意事项

1. **API Key**: 已固定在 `server.py` 第42行，无需前端输入
2. **模型列表**: 进入测评页自动加载353+个可用模型
3. **超时**: 单题API调用超时120秒
4. **LaTeX**: 前端使用KaTeX渲染数学公式
5. **容差**: 通过题目的 `tolerance` 字段控制答案允许误差

---

## 待扩展

- [ ] 测评进度实时显示
- [ ] 并发测评（当前串行）
- [ ] 结果导出为CSV/PDF
- [ ] 题目分类标签

---
> Source: [Proton1917/ai-benchmark](https://github.com/Proton1917/ai-benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
