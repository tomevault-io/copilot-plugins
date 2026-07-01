## fame-knowledge-agent-gateway

> 中文优先。English follows.

# AGENTS.md

中文优先。English follows.

## 一句话

FAME Knowledge Agent Gateway 是 Agent 的外部知识、工程记忆和工具治理插件。它不是普通 RAG。Agent 必须按路线、作用域、ContextPack、FAME 参数和 Tool Gateway 工作。

## 先选版本

- 中文开源版：`versions/chinese-open/`
- English open edition：`versions/english/`

## 最小循环

```text
用户目标 -> 语言树归一 -> subject/route_id
-> ContextPack -> ProposedAction
-> Tool Gateway / ApprovedAction
-> ToolResultSummary -> 工程记忆 overlay
```

## 必读入口

中文：

- `versions/chinese-open/QUICKSTART_AGENT.md`
- `versions/chinese-open/docs/07_Agent协议与最小接入.md`
- `versions/chinese-open/docs/agent-protocol/`
- `versions/chinese-open/docs/agent-protocol/09_fame-and-knowledge-extension.md`
- `versions/chinese-open/docs/04_Agent接入工作流.md`
- `versions/chinese-open/docs/08_常见Agent接入片段.md`

English:

- `versions/english/QUICKSTART_AGENT.md`
- `versions/english/docs/agent-protocol.md`
- `versions/english/docs/agent-protocol/`
- `versions/english/docs/agent-protocol/07_fame-and-knowledge-extension.md`
- `versions/english/docs/common-agent-snippets.md`

## 快速命令

```bash
npm run route:chinese-open -- --goal "<用户目标>" --compact
npm run connect:chinese-open -- --agent codex
npm run route:english -- --goal "<user goal>" --compact
npm run connect:english -- --agent codex
```

## 不允许

- 不允许全量加载知识网。
- 不允许把长日志、完整工具输出或完整图谱塞进上下文。
- 不允许绕过 ProposedAction / ApprovedAction 执行风险工具。
- 不允许覆盖用户改动。
- 不允许把工程 overlay 自动写入核心知识网。
- 不允许无证据宣称完成。

## Workbench

Workbench 默认打开 `Agent 外脑`。这里看安全阻断、动作契约审批流、失败签名热力区、动态免疫网、上下文健康和写回队列。

`Agent Runtime` 视图提供本机 Agent 接入向导。它只做消息交接和用户确认，不自动改写本机 Agent 配置。

## English

FAME Knowledge Agent Gateway is an external knowledge, project-memory and tool-governance plugin for agents. It is not plain RAG. Agents should operate through route ids, scoped ContextPacks, FAME parameters and Tool Gateway rules.

Minimal loop:

```text
Goal -> Language Tree -> subject/route_id
-> ContextPack -> ProposedAction
-> Tool Gateway / ApprovedAction
-> ToolResultSummary -> project-memory overlay
```

Never full-load the knowledge net, overwrite user changes, run destructive tools without scope checks, or claim completion without evidence.

---
> Source: [superalp1985/fame-knowledge-agent-gateway](https://github.com/superalp1985/fame-knowledge-agent-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
