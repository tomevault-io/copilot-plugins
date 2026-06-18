## track4world-platform

> When working with large codebases or long sessions, proactively compact context and avoid including images/large files in conversation. Split large tasks into smaller, independent sub-tasks to avoid hitting request size limits.

## General Guidelines

When working with large codebases or long sessions, proactively compact context and avoid including images/large files in conversation. Split large tasks into smaller, independent sub-tasks to avoid hitting request size limits.

Do not include raw images or large binary files in context. If referencing visual outputs, describe them textually or reference file paths instead.

## Critical Rules

NEVER fabricate or estimate experimental data. Always verify numbers against source files before including them in any document, table, or chart.

## File Handling

When working with large files (>10MB), read only the specific sections needed rather than loading entire files. Use grep, head, tail, or awk to extract relevant data.

## Workflow

Always confirm the full requirements before starting multi-step tasks. Re-read the user's constraints after each step to avoid drift.

## Python / Training

This project uses Python with PyTorch (mixed precision training). When writing training code, always cast BCE loss inputs to float32 before calling binary_cross_entropy. Use `tensor.float()` explicitly.


Do NOT read entire large files. Use head/tail/grep to inspect specific sections. Do not include images in context. If we hit size limits, summarize progress and tell me what to carry to a new session.



Before we end, write a handover doc to ./HANDOVER.md with: completed tasks, pending tasks, current errors, and file paths touched. I'll use this to start the next session.

---
> Source: [137Qpalzm/track4world-platform](https://github.com/137Qpalzm/track4world-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
