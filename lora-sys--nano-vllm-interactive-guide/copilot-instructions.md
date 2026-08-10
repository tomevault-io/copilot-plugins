## nano-vllm-interactive-guide

> 维护一本面向初学者与开发者的 nano-vLLM 交互式源码教程。首要目标是教学正确性与可验证性，不是堆砌术语或追求炫技动画。

# AGENTS.md

## 项目目标

维护一本面向初学者与开发者的 nano-vLLM 交互式源码教程。首要目标是教学正确性与可验证性，不是堆砌术语或追求炫技动画。

## 修改规则

1. 上游相关事实先检查 `GeeeekExplorer/nano-vllm` 当前主分支。
2. 不把 HTML 概念实验描述成真实模型或真实 GPU 执行。
3. 每个 `docs/guide/*.md` 必须包含一个 `<HtmlLab>` 和一个 `<ExerciseCard>`。
4. 新增 lab 放在 `docs/public/labs/`，不得依赖外部 CDN，确保 GitHub Pages 离线构建。
5. 浏览器 API 只在 Vue `onMounted` 后使用，保持 VitePress SSR 构建可用。
6. 修改后运行 `npm run check`。
7. 性能结论必须记录硬件、模型、依赖版本、输入输出分布、warmup 和重复次数。

## 写作风格

- 中文为主，首次出现的英文术语给出解释。
- 先直觉、再数据流、再源码定位、最后边界与误区。
- 避免把 vLLM 生产实现的所有能力错误投射到 nano-vLLM。
- 代码片段尽量短，强调关键状态变化。

---
> Source: [lora-sys/nano-vllm-interactive-guide](https://github.com/lora-sys/nano-vllm-interactive-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
