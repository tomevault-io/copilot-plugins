## ai901-cert-buddy-claude

> Guidance for **Claude Code** when working in this repository.

# CLAUDE.md

Guidance for **Claude Code** when working in this repository.

## What this is

**AI-901 Cert Buddy (Claude Code edition).** An agentic study companion for **Exam AI-901: Microsoft Azure AI Fundamentals**. It generates exam-realistic practice questions, brief hands-on **Microsoft Foundry** labs, and personalized study plans, all grounded in **Microsoft Learn** through the **Microsoft Learn MCP server**.

This project is a **Claude Code port** of Tim Warner's original GitHub Copilot "Cert Buddy" (`az104-cert-buddy`, `ai901`). The point of the port is to show that the same enterprise workflow carries across both agent platforms, one primitive at a time. See `README.md` for the full primitive map.

There is no application to build. The artifacts are agent definitions, skills, slash commands, hooks, and MCP configuration.

## Where things live (Claude Code primitives)

| Primitive | Location |
| --- | --- |
| Memory (this file) | `CLAUDE.md` |
| Subagents | `.claude/agents/` |
| Slash commands | `.claude/commands/` |
| Agent Skills | `.claude/skills/` |
| Hooks | `.claude/settings.json` + `.claude/hooks/` |
| MCP servers | `.mcp.json` |
| Plugin packaging | `.claude-plugin/` |

## The grounding contract (non-negotiable)

1. **Microsoft Learn first.** Ground every fact about Microsoft Foundry, the Foundry SDK, Azure AI services (Language, Speech, Vision), Azure Content Understanding, Azure OpenAI in Microsoft Foundry, and responsible AI in **Microsoft Learn**, accessed through the MCP server `ai901buddy-mslearn`.
2. Use `microsoft_docs_search` for breadth, then `microsoft_docs_fetch` for full-page detail, and `microsoft_code_sample_search` when SDK, CLI, or Bicep syntax accuracy matters.
3. **Cite a real Microsoft Learn URL** for every question and every lab. Never invent a URL. If you cannot ground a claim, say so and stop.
4. Pick objectives from `.claude/skills/_references/ai901-objective-domain.md` (a verbatim Microsoft Learn sync).

## AI-901 canonical facts

- **Exam:** AI-901, Microsoft Azure AI Fundamentals.
- **Skills measured date:** April 15, 2026. **Pass score:** 700.
- **Domains:** Identify AI concepts and capabilities (**40-45%**); Implement AI solutions by using Microsoft Foundry (**55-60%**).
- **Audience:** beginning of a career in AI solution development; conceptual Azure AI knowledge; Python syntax fluency; familiar with Azure resources.

## Hard rules the skills depend on

- **Two-phase question delivery.** Present the scenario, stem, and choices ONLY. Stop. Wait for the learner's answer. THEN reveal the correct letter, a two-sentence rationale per choice, and the Microsoft Learn URLs. Never reveal the answer in the same message as the question.
- **Answer-letter randomization.** The correct answer must be randomized across A, B, C, and D. Never default to one position.
- **Fictional-company randomization.** Use only the WWL-approved company list in `.claude/skills/_references/fictional-companies.md`. Use the entire company name on every mention. Do not default to Contoso, Ltd.
- **Keyless auth default.** All Foundry SDK and Azure SDK samples use **keyless authentication via Microsoft Entra ID** with `DefaultAzureCredential`. No API keys unless an objective explicitly requires key-based auth.
- **Style precedence.** Exam items follow the **Microsoft Worldwide Learning Exam Writing Style Guide (WWL)**; prose follows the **Microsoft Writing Style Guide (MWSG)**. When they conflict in an item, WWL wins (no contractions; key names in ALL CAPS). Full rules in `.claude/skills/_references/wwl-style-guide.md`.

## Current Microsoft product names (silently map retired names)

Always use current names, even if the user types a retired one.

| Retired / legacy name | Current name |
| --- | --- |
| Azure Active Directory (Azure AD) | Microsoft Entra ID |
| Azure AI Studio | Microsoft Foundry |
| Azure AI Foundry | Microsoft Foundry |
| Azure OpenAI Service (standalone) | Azure OpenAI in Microsoft Foundry |
| Azure Cognitive Services | Azure AI services |
| Azure Cognitive Search | Azure AI Search |
| Form Recognizer | Azure AI Document Intelligence |
| LUIS | Conversational language understanding in Azure AI Language |

If Microsoft Learn shows a different current name, prefer the Learn name.

## How the agent and skills fit together

The **ai901-cert-buddy** subagent (`.claude/agents/ai901-cert-buddy.md`) is the orchestrator. It delegates to three skills based on the request:

- **ai901-item-creator** for practice questions.
- **ai901-lab-creator** for hands-on Foundry labs.
- **ai901-study-planner** for personalized study plans.

The **grounding-verifier** subagent independently checks that generated content cites live Microsoft Learn URLs and uses no retired product names. The **azure-principal-architect** subagent is an optional "pair architect" for lab design.

## Author

Tim Warner, TechTrainerTim.com. MIT licensed.

## Imported rules

@.claude/rules/grounding.md

## Project layout (v0.2 additions)

Beyond the core `.claude/` primitives, this repository now includes:

- A custom MCP server in `mcp-server/` (server id `cert-buddy`), built on the MCP Python SDK (FastMCP). It exposes tools (`score_answer`, `validate_item`, `next_question`, and the optional `generate_item`), resources (`certbuddy://domains`, `certbuddy://progress`, `certbuddy://bank/{domain}`), and prompts (`quiz`, `exam`). It is the scoring and interaction layer; **Microsoft Learn remains the source of truth.**
- Two more hooks: `session-start.py` (**SessionStart**) and `prompt-context.py` (**UserPromptSubmit**), which keep the grounding and WWL rules in front of the model on every session and every prompt.
- A **Cert Coach** output style in `.claude/output-styles/`.
- A workspace MCP config at `.vscode/mcp.json` so the Microsoft Learn MCP is turnkey in VS Code as well as Claude Code.

## Turnkey grounding (non-negotiable)

The Microsoft Learn MCP connection **travels with this repository** in `.mcp.json` (Claude Code) and `.vscode/mcp.json` (VS Code). A clone is grounded out of the box with no API key. All AI-901 content is grounded **100 percent at Microsoft Learn first**, and every practice item follows the **Microsoft Worldwide Learning (WWL)** exam-writing rules. The agent and skills never substitute generic web search for Microsoft Learn.

---
> Source: [timothywarner-org/ai901-cert-buddy-claude](https://github.com/timothywarner-org/ai901-cert-buddy-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
