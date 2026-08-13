## test-verification

> 代码修改后强制全量测试验证，严格限制mock仅用于测试类


# 测试验证与Mock限制

每次生成或修改代码后，必须执行以下流程：

1. 运行全部测试命令（`pytest tests/ -v` 或 `npm test`），确认 100% 通过。
2. 验证功能正确性与 AI 能力（ReAct 编排、prompt 解析、真实 LLM 调用链）。
3. **仅允许**在 `tests/` 目录下的测试类、方法或 support/ 脚本中书写 mock（例如 scripted_llm.py）。
4. **严禁**在 core/、apps/、docs/ 等非测试代码中出现任何 mock、stub、fake 实现或条件绕过。

若测试失败，必须定位真实问题并修复，严禁通过新增 mock 使测试“通过”。

**BAD 示例**：
```python
# core/agents/llm_client.py 中
if test_mode:
    return mock_response  # 禁止
```

**GOOD 示例**：
- 修改 `core/llm/master_react.py` 后立即执行：
  ```bash
  pytest tests/unit/ -q && pytest tests/ -q
  ```
- 仅在 `tests/support/scripted_llm.py` 中扩展 LLM 脚本响应。
- 测试因 prompt 变更失败 → 同步更新 `docs/superpowers/reference/prompt-architecture.md` 并修复实际逻辑。

所有变更提交前必须附带“全量测试通过 + 无非测试mock”的确认。

---
> Source: [GodyuFF/SuperVideoGenerator](https://github.com/GodyuFF/SuperVideoGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
