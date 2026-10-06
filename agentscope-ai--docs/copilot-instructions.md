## docs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AgentScope 2.0 is a production-ready multi-agent framework for building LLM-empowered agent applications. This repository contains the documentation for the AgentScope framework, built with Mintlify. 

The goal of this documentation is to help developers understand "how to use" the AgentScope 2.0 framework. Explaining "how AgentScope works or implements a feature internally" is secondary — your goal is to enable readers to use each part of the framework based on the documentation.

## Writing Style Guidelines

Follow these core principles when writing documentation:

### The Source Code Helps
- AgentScope is open-sourced in https://github.com/agentscope-ai/agentscope
- Read the source code helps you better understand and translation

### Language and Style
- Use clear, concise, precise, and direct language appropriate for technical audiences
- Address the reader as "developer" in instructions and procedures (use "开发者" in Chinese documentation)
- Use active voice over passive voice
- Maintain consistent terminology throughout all documentation
- Minimize em dashes (`—` in English, `——` in Chinese) in both languages. Prefer commas, colons, parentheses, or splitting into two sentences. When a bullet list pairs each item with a dash-joined explanation (e.g. "单个 `Msg` 或 `Msg` 列表——开始一次新的回复"), turn the list into a table instead. Em dashes as empty-cell placeholders in tables are fine

### Content Organization
- Lead with the most important information (inverted pyramid structure)
- Use progressive disclosure: basic concepts before advanced ones
- Break complex procedures into numbered steps
- Include prerequisites and context before instructions
- Provide expected outcomes for each major step
- Always precede tables, images, diagrams, and code blocks with an explanatory sentence or paragraph that describes what the content shows or demonstrates
- Avoid long, dense paragraphs — they make documentation hard to read and understand; break content down with code, bullet points, steps, tables, etc.

A representative page structure:

1. The first paragraph introduces the feature very concisely (usually one sentence): what it does — so readers know what this section covers. Do not use descriptions like "This section introduces ..."; instead, introduce the feature directly, e.g. "When xxx (scenario), you can achieve xxx by setting the xxx field of xxx"
Note: do not copy this example sentence verbatim — in the English version, use more natural, idiomatic English phrasing.
2. The second paragraph introduces the concepts/features, also serving as an overview and lead-in; it can incorporate tables, diagrams, code, etc.
3. Introduce each part in separate sections
  - For important interfaces, explain how to use them in detail
  - For abstractions, introduce the implementations currently supported in the repo

## Mintlify Component Usage

### Callout Components
- `<Note>` - Supplementary information that supports the main content
- `<Tip>` - Expert advice or best practices
- `<Warning>` - Critical information about potential issues
- `<Info>` - Background information or context
- `<Check>` - Success confirmations or positive indicators

### Code Components
- **Single code block**: Include language specification when possible
  ```javascript config.js
  const apiConfig = {
    baseURL: 'https://api.example.com',
    timeout: 5000
  };
  ```

- **Code groups**: Use `<CodeGroup>` for multiple implementation options
  ````
  <CodeGroup>
  ```javascript Node.js
  const response = await fetch('/api/endpoint');
  ```

  ```python Python
  import requests
  response = requests.get('/api/endpoint')
  ```
  </CodeGroup>
  ````

- **API examples**: Use `<RequestExample>` and `<ResponseExample>` for API documentation

### Structural Components
- **Steps**: Use `<Steps>` for sequential instructions
- **Tabs**: Use `<Tabs>` for platform-specific content
- **Accordions**: Use `<AccordionGroup>` for expandable content
- **Cards**: Use `<Card>` and `<CardGroup>` to highlight important information
  - The `<Card>` component must have `title`, `icon`, `href`, and `cta` fields

### API Documentation
- **Parameters**: Use `<ParamField>` to document API parameters
- **Responses**: Use `<ResponseField>` to document API response properties
- **Nested objects**: Use `<Expandable>` for hierarchical information

## Page Structure

Every documentation page must begin with YAML frontmatter:
```yaml
---
title: "Clear, specific, keyword-rich title"
description: "Concise description explaining page purpose and value"
---
```

The `description` must read like one natural sentence, not a keyword list or a colon/dash-joined feature enumeration. Good examples (in the spirit of agno's docs, do not copy them verbatim): "Run agents and process their output.", "Start simple: a model, tools, and instructions.", "Groups of agents that collaborate to solve complex tasks.". Bad example: "Drive the reasoning-acting loop: reply, stream, structured output, observation, compression, and persistence".

The `description` must state the page's outcome or value, not enumerate its mechanisms. A mechanism list reads like a table of contents and tells readers nothing about why the feature matters. For example, for the context compression page: "汇总较早的消息，截断过大的工具结果" (bad, enumerates the two mechanisms) ⇒ "将上下文长度维护在预设的长度内" (good, states the outcome). Apply the same standard to `<Card>` blurbs.

The first paragraph of a page must introduce the page as a whole, not just the first section. When a page covers several interfaces or behaviors, open with a short overall framing (e.g. "The `Agent` class abstracts what an agent does into a small set of behaviors, each suited to a different goal:") followed by a table mapping each behavior/section to its purpose.

## Content Quality Standards

### Chinese Version
- In the Chinese version, except for a small set of proper nouns (e.g. Python, RAG, URL, base64, ReAct), avoid mixing Chinese and English as much as possible. For example, use the Chinese terms 智能体/中间件/工具/系统提示/上下文 instead of mixing in the English words "agent/middleware/tool/system prompt/context"
- The Chinese version must not be a literal, word-for-word translation of the English version. Ensure the Chinese reads naturally, and that translated terms sound natural — especially for English clauses, the result must fit Chinese reading habits. For example:
  - Translating "assistant message" as "智能体消息" is better than "助手消息", because it contrasts with user message and system message — translate terms in context
  - Good example: "Agent self-managed tool system with Python functions, MCP and skills integration." ==> "智能体工具的自主管理系统，支持自主按需装配/卸载 Python 工具、MCP 和技能。"
  - Bad example:
```English source
A workspace is the agent’s execution environment. It supplies the agent with three categories of resources — tools (built-in tools and MCPs), skills, and context offloading for compressed messages and oversized tool results — and owns the lifecycle of the resources living inside it (MCP server processes, dynamically added skills, offloaded files).
```

```Bad translation
Workspace 是 agent 的执行环境，向 agent 提供三类资源 —— 工具（内置 tool 与 MCP）、skill，以及面向压缩消息与超大工具结果的上下文 offload —— 同时管理其中资源（MCP server 进程、动态加入的 skill、offload 文件）的生命周期。
```

A better Chinese phrasing is:
```Good translation
工作区（Workspace）是智能体的运行环境，负责提供以下能力：

- **工作区操作工具**：包括命令执行（Bash/Grep/Glob）及文件读写工具（Write/Edit/Read)；
- **文件系统支持**：持久化存储技能（Skill）与卸载上下文信息；
- **资源生命周期管理**：统一管理 MCP 服务器进程，支持运行时添加/移除 MCP 服务和技能。
```

- In the Chinese version, translate "offload" as 卸载 and "offloader" as 卸载器 (first mention: 卸载器（Offloader）) — do not leave the English word "offloader" scattered through Chinese prose
- In the Chinese version, translate "(model) provider" as 模型 API (or just API when the context is clear), never as 提供商 — the docs care about which API a model class talks to, not the vendor as a company. For example, "Switching the LLM provider only changes the `model` argument" should be "切换模型 API 只需改动 `model` 参数", and a "Provider" table column should be "模型 API"
- When translating, restore the real subject of the sentence instead of copying the English surface subject. In English docs, "each provider ships / every provider has ..." often really means "AgentScope provides ... for each API" — the provider (OpenAI, DashScope, ...) does not ship AgentScope's classes. For example, for "For multi-entity conversations, each provider ships a `MultiAgentFormatter`.":
  - Bad translation: "针对多实体对话，每个提供商都提供了 `MultiAgentFormatter`。"
  - Good translation: "针对多实体对话，AgentScope 为每种 API 都提供了对应的 `MultiAgentFormatter`。"

### Section Title
- Section titles must be short and concise, with the first letter of each word capitalized
- Keep title patterns consistent whenever possible, e.g. "verb + noun"

### Code Examples
- Must include a very concise title
- Use comments in the code to explain what parameters do — never provide bare code without any comments
- Use `<CodeGroup>` to provide code examples of the same feature in different scenarios
- Include complete, runnable examples that users can copy and execute
- Include expected outputs when possible
- Specify language and include filename when relevant
- Never include real API keys or secrets

### Documentation Standards
- Keep all terminology consistent across the English and Chinese versions — avoid referring to the same concept with different terms in different sections
- Every table, image, flowchart, and sequence diagram must be preceded by at least one very concise sentence introducing the content below
- Use specific, actionable link text
- Ensure proper heading hierarchy starting with H2
- Write for scannability with clear headings and lists

### Creating New Documentation Pages
- Add the new .mdx file in the appropriate directory
- Update the `docs.json` navigation configuration to include the new page
- Ensure the file starts with proper YAML frontmatter
- Apply appropriate Mintlify components to enhance readability

---
> Source: [agentscope-ai/docs](https://github.com/agentscope-ai/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
