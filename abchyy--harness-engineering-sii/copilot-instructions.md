## harness-engineering-sii

> - Import 白名单：Python 标准库、NumPy、`harness_base`。禁止 SciPy、scikit-learn 及其他第三方包。

# 硬约束（每次任务前必读）

- Import 白名单：Python 标准库、NumPy、`harness_base`。禁止 SciPy、scikit-learn 及其他第三方包。
- 禁止文件读写、自行发起网络请求和创建子进程；LLM 只能通过注入的 `self.call_llm` 调用。
- 单次 Prompt 必须不超过 `max_prompt_tokens - safety_margin`；调用前用 `count_messages_tokens` 实测，不得估算。被动截断警告必须为 0。
- `predict()` 不得修改任何影响预测结果的状态。一次性、幂等、加锁保护且结果确定的构建除外。
- 所有 JSON 序列化使用 `ensure_ascii=False, separators=(",", ":")`。
- 检索使用 `dict` 倒排索引，不使用 SciPy 稀疏矩阵或 NumPy 稠密语料矩阵。
- 可调超参数总数不超过 3 个；新增参数必须说明必要性并提供消融数据。
- 单条样本的 LLM 调用次数硬上限为 2；禁止 `while True` 和无界重试。
- `predict()` 任何路径都必须返回训练标签集合内的字符串，退出前执行成员校验。
- 禁止硬编码任何 DEV 标签、领域词或答案映射。

# 工作方式

- 一次只改一个主要变量；先跑子集，再报告改动前后的准确率、token、延迟和异常数字。
- 没有实测数据支撑的改动不进入最终候选。
- 每次改动后用两句话解释“为什么采用它”和“为什么不是另一种做法”。
- 输出使用简洁、准确的语言。

---
> Source: [Abchyy/harness-engineering-sii](https://github.com/Abchyy/harness-engineering-sii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
