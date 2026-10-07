## frontend-knip-exports

> 前端 export / knip：只导出被外部使用的符号，改完须过 knip


# 前端导出与 knip

修改 `frontend/`（尤其新增 `utils`、hooks、公共组件）时：

1. **默认不 `export`**：模块内辅助函数/类型保持私有；仅当其它文件真实 import 时才导出。
2. **禁止「顺便导出」**：不要为「以后可能用」或对称 API 而 export 未使用的 `patchX` / `listX` / `addX` / `removeX`。
3. **交付前跑 knip**：`cd frontend && npm run knip`，修到无 Unused exports / Unused dependencies 再结束（pre-commit husky 会拦）。
4. **与 `.qoder/rules/agent.md` 第 8 条一致**：禁止默认 `--no-verify` 跳过。

典型修法：去掉多余 `export`，或删除完全未引用的函数。

---
> Source: [aiqachat/tkeapi](https://github.com/aiqachat/tkeapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
