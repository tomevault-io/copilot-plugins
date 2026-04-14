## the-l-ai-ons-den

> 1. **[README.md](README.md)** - System overview.

# Gemini Integration for The L-AI-ons' Den

## Required Reading (In Order)

1. **[README.md](README.md)** - System overview.
2. **[AI-instructions.md](AI-instructions.md)** - Core workflow, file management, and agent logic.
3. **This file** - Gemini CLI specific instructions.

## Gemini-Specific Workflow

Gemini's strength is its large context window, allowing for comprehensive, single-prompt operations for both review and fix application.

### Review and Aggregation in a Single Step

Instead of running agents separately, instruct Gemini to assume all relevant personas, perform the review, and act as the Judge to aggregate the findings in one go.

### Comprehensive Fix Application

When the user initiates fix mode, use a single, detailed prompt that references all findings and outlines the implementation phases defined in `AI-instructions.md`.

```bash
# Example fix prompt for session 6
gemini "
Apply comprehensive improvements to @sessions/6/document.html based on all findings in @sessions/6/agent_results/

Follow the standard implementation phases:
1. Emergency fixes
2. Safety additions
3. Grammar corrections
4. Readability improvements
5. Structural enhancements
6. Final validation and reporting

Preserve technical accuracy while implementing all improvements.
Document every change in sessions/6/fixes_applied.md with a before/after comparison.
"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ra13s)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ra13s)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
