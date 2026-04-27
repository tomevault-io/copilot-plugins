## llm-universal-proxy

> - 所有子代理在任何情况下都必须使用 `xhigh` reasoning effort。

# 子代理约定

## 通用

- 所有子代理在任何情况下都必须使用 `xhigh` reasoning effort。

## explorer

- 默认使用 `gpt-5.4` + `xhigh`。
- 只有任务特别简单，或者基本不需要逻辑分析时，才可以使用 `gpt-5.4-mini` + `xhigh`。

## worker

- 默认使用 `gpt-5.4` + `xhigh`。
- 只有任务非常简单时，才可以使用 `gpt-5.4-mini` + `xhigh`。

---
> Source: [lzjever/llm-universal-proxy](https://github.com/lzjever/llm-universal-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
