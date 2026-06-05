## line-faq-bot-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **LINE FAQ Bot** — a chatbot that answers frequently asked questions via the LINE messaging platform. The project is in early stages; no application code exists yet.

Primary language for documentation and agent instructions: **Japanese**.

## Custom Agents

Four specialized subagents are configured in `.claude/agents/`:

| Agent | Role |
|---|---|
| `planner` | 要件整理・MVP設計・機能優先順位の整理 |
| `reviewer` | コード品質・セキュリティ・保守性レビュー |
| `bug-fixer` | エラー解析・原因特定・デバッグ |
| `doc-writer` | README・技術ドキュメント作成（Haiku モデル使用） |

Use these agents via the Agent tool when tasks match their roles (e.g., use `planner` before starting a new feature, `reviewer` before finalizing code).

## Design Principles (from agent configs)

- Prioritize **simplicity** and **maintainability** over clever abstractions
- Keep the codebase understandable to beginners
- Optimize for **mobile usability** (LINE is primarily a mobile app)
- Avoid over-engineering; separate MVP features from nice-to-haves

---
> Source: [chiyo-labs/line-faq-bot-demo](https://github.com/chiyo-labs/line-faq-bot-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
