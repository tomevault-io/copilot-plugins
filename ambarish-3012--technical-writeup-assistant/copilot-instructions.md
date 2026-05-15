## technical-writeup-assistant

> Project: Technical Write-up Assistant

Project: Technical Write-up Assistant

Purpose
Generate architect-level technical articles that can be published on Medium and repurposed across platforms.

Core Principles

• Outline must always be generated before writing the full article.
• Article sections should be regenerable independently.
• Prompts must remain separate from application logic.
• Publishing must always require explicit user approval.
• Diagrams should support interactive editing and export.
• Articles must be exportable in Medium-ready Markdown.
• The system should remain modular and extensible.

Architecture Expectations

• Local-first application.
• Clean modular architecture.
• Separation between UI, orchestration, and AI services.
• Avoid monolithic scripts.
• Prefer tool-based architecture (writer, diagram generator, publisher).

Knowledge Strategy

The system must support multiple knowledge modes:

1. Model knowledge (baseline generation)
2. Public/open-source technical references
3. Optional local knowledge base

The system must still produce high-quality articles even if no local knowledge base exists.

Cost Efficiency

• Prefer staged generation workflows.
• Avoid regenerating the entire article unnecessarily.
• Allow section-level refinement.

---
> Source: [ambarish-3012/technical-writeup-assistant](https://github.com/ambarish-3012/technical-writeup-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
