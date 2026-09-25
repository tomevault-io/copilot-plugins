## codexcomp

> - codexcomp 是 Codex CLI 与上游 Responses API 间的环回代理。通过顶层 `openai_base_url` 接入；bind 保持环回，Authorization 原样转发且不检查、记录或持久化。

# 项目契约

- codexcomp 是 Codex CLI 与上游 Responses API 间的环回代理。通过顶层 `openai_base_url` 接入；bind 保持环回，Authorization 原样转发且不检查、记录或持久化。
- 上游始终无状态 SSE POST，每轮完整重放 input；不转发 `previous_response_id` 或 `generate`，不直连上游 WebSocket。首轮零 reasoning 是合法回答，不额外重试。
- 干净轮次不丢失或重排 output items；代理拥有 sequence_number/output_index 和终止事件，非 reasoning 暂存到轮次确认后释放。EOF、流错误和续写打开失败产生 `response.incomplete`，首轮拒绝产生 `response.failed`，不伪造完成。保留逐轮与累计成本 metadata。
- 修改 fold 运行 `uv run python test_fold.py`，修改 WebSocket 路径运行 `uv run python test_ws.py`；均为 assert 脚本。eval 命令消耗真实 token/quota，按明确任务需要执行。
- 用户可见行为同时维护 README.md 与 README.zh-CN.md；保留 neteroster/CodexCont 致谢，LICENSE 保持纯 MIT。跟踪任务见 `docs/agents/issue-tracker.md`。
- 发布候选先步进版本并进入 master，等同一 SHA 的普通 CI 成功后推匹配 annotated tag；远端 tag 不移动或复用。v0.1.0–v0.3.8 的历史 lightweight tag 保持原样。PyPI 发布成功后创建 GitHub Release。
- 本地服务更新只在明确存在活跃 uv-tool/service 部署且任务覆盖时执行；包安装本身不注册服务。运行与架构说明见 README。

---
> Source: [dzshzx/codexcomp](https://github.com/dzshzx/codexcomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
