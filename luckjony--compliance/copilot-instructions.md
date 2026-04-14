## compliance

> 你正在重构一个 Streamlit 外贸合规审计系统。请将架构从“线性脚本”升级为“多智能体协作（Agentic Workflow）”。本规则用于指导你在 Cursor 中的规划、编码与状态管理。

你正在重构一个 Streamlit 外贸合规审计系统。请将架构从“线性脚本”升级为“多智能体协作（Agentic Workflow）”。本规则用于指导你在 Cursor 中的规划、编码与状态管理。

============================================================
【目标】
从 Input -> Search -> AI Output（线性）升级为：
Router Agent（路由专家） -> SQL Search Tool（本地名单查询） + RAG Tool（法规检索） -> Reasoning Agent（总审计长） -> UI 实时刷新

============================================================
【角色定义（Agents）】

1) Router Agent（路由专家）
- 职责：判断输入属于查人/查货/查支付/混合（人货对撞）。
- 产出：route_plan（需要调用哪些工具、并行策略、UI 步骤更新计划）。

2) Search Tool（sql_search_tool：搜索工具）
- 职责：在本地数据库中做秒级名单查询（优先 DuckDB/SQLite），支持模糊匹配。
- 最低要求：先做“精确/规范化匹配”，再做“模糊匹配（阈值可配置）”。
- 产出：entity_hits（命中列表、source、program、remarks、addresses、score）。

3) RAG Tool（rag_retriever_tool：合规分析师）
- 职责：在 Chroma 向量库（legal_clauses 集合）中检索对齐的 PDF 条文。
- 输出 TopK：语义 Top3 + 关键词 Top3（BM25），合并去重后 Rerank。
- 产出：legal_evidence（前3条原文必须可渲染到右侧“法律存证区”）。

4) Reasoning Agent（总审计长）
- 职责：汇总“查人 + 查货 + 地缘 + 支付路径”证据，判断是否存在“人货对撞”风险。
- 产出：verdict（风险等级、风险性质、支付风险、货物处理建议、证据引用）。

============================================================
【工具集（Tools）】

sql_search_tool（必须封装为一个独立函数/模块）
- 输入：name（实体名称）、threshold（模糊阈值）、role（收货人/进口商/出口商）
- 过程：本地数据库查询（DuckDB/SQLite 优先）→ 必要时再走 Pandas 深查
- 输出（dict）：
  {
    "matched": bool,
    "hits": [ { "source": "...", "hit_type": "...", "matched_name": "...", "program": "...", "remarks": "...", "addresses": [...], "score": 0-100 } ],
    "fast_path": "sql" | "pandas"
  }

rag_retriever_tool（必须封装为一个独立函数/模块）
- 输入：query（产品描述/HS/目的地）、top_k（默认3）
- 过程：Chroma Top3 + BM25 Top3 → 合并去重 → Rerank → Top3
- 输出（list[dict]）：
  [ { "text": "...", "source_id": "...", "source_name": "...", "match_score": 0-1, "retrieval": "chroma|bm25" } ]

============================================================
【思维链规划（Planning 协议）】
在做任何代码修改前，你必须先输出一个：

[PLAN]
1. 从输入提取实体（consignee/importer/exporter）→ 调用 sql_search_tool（可并行）
2. 从输入提取产品描述/HS/目的地 → 调用 rag_retriever_tool（可并行）
3. 交叉对比（人货对撞）→ 生成合规建议（不得使用“禁止交易”等强制措辞）
4. 将中间结果写入 st.session_state，并在 UI 中逐步刷新（不要等全流程结束）

注意：推理过程不要泄露给用户；只输出结果与可执行建议。

============================================================
【状态管理（State）】
必须使用 st.session_state 存储每个 Agent 的中间结果，并保证 UI 可实时更新：

- session_state.router_plan
- session_state.sql_hits
- session_state.rag_evidence
- session_state.reasoning_verdict
- session_state.total_risk_score
- session_state.risk_level / risk_us / risk_geo / risk_payment / risk_goods

============================================================
【异步执行（Parallel Execution）】
当 RAG 还在检索时，允许先渲染已经完成的名单结果：
- 优先使用 asyncio；在 Streamlit 环境下可退化为 ThreadPoolExecutor。
- 不要让 UI “盯着进度条等半天”：使用 placeholder（st.empty）逐步刷新右侧证据与风险灯。

============================================================
【输出规范（合规文案）】
- 风险分级要“软表达”：使用“潜在限制/需要核实/建议评估”。
- 操作建议应面向外贸业务员：优先建议“更换结算币种/清算路径、核实最终用户/用途、补充单证、合规留痕”。
- 严禁输出“禁止交易/必须停止/不得出口”等强制性指令。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LuckJony) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
