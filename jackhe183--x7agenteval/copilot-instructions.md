## x7agenteval

> 请使用第一性原理思考。在动手之前，先确认你理解了每个模块的职责边界：

# X7AgentBench Developer Guidance

请使用第一性原理思考。在动手之前，先确认你理解了每个模块的职责边界：
- CustomerAgent 的首问不能调模型
- StopAgent 的低置信度判断必须强制转为继续
- LogFetcher 在对话结束后才调用，不在对话 loop 里
- EvalAgent 无记忆，每次独立评分
- 所有 API Key 和 URL 从环境变量读，不能硬编码

代码风格要求：
- 每个 class 和 method 必须有 docstring，说明职责和关键约束
- 不要在非 Config 的文件里出现硬编码的字符串参数
- 异常处理原则：Agent 内部的 LLM 调用失败，降级返回默认值；X7 接口失败，返回错误字符串；LogFetcher 失败，返回空列表。三者都不能 raise 导致整个批量任务中断

如果路径不清晰，停下来问我，不要猜。

---
> Source: [jackhe183/X7AgentEval](https://github.com/jackhe183/X7AgentEval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
