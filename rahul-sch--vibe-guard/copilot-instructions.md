## vibe-guard

> CLAUDE.md — Project Guidelines

CLAUDE.md — Project Guidelines
Token Optimization Rules

NO EXPLANATIONS
Do not explain what you did. Do not say “Here’s the code.” Only output the result.

DIFFS ONLY
When editing a file, output a unified diff or a sed / search-replace block.
Never rewrite or reprint entire files unless explicitly asked.

NO FLUFF
No apologies. No acknowledgements. No “Sure.” No “Got it.” Just execute.

NO CONTEXT SCANNING
Only read files that are explicitly named.
Never explore the repository on your own.

MINIMAL OUTPUT
Output tokens are expensive.
Prefer the smallest possible diffs and the narrowest possible changes.

Phase Execution Rules

ONE PHASE AT A TIME
You must only work on the current phase defined in the VibeGuard plan.

PHASE LOCKING
You may not touch a future phase until the current phase is fully complete.

COMMIT REQUIRED
At the end of each phase:

All tests must pass

A Git commit must exist

Commit format:

phase-X: <short description>

After committing, you must stop and wait.

Security Scanner Rules

This project is a security product.
If detection logic is weak, vague, or unsafe, you must improve it.

Regex rules must be:

High-confidence

Low false-positive

Exploit-oriented

No demo rules. No placeholders.

AI Cost Control

Never re-read files unless necessary.
Use Repomix (repomix.xml) as the source-of-truth map when present.

Always prefer:

File-scoped edits

Function-level diffs

Surgical patches

If a change affects many files, ask for confirmation first.

Project Context

Project: VibeGuard

Type: Regex-first security scanner CLI

Stack: TypeScript, Node.js

Purpose: Detect insecure AI-generated code (secrets, injections, supply-chain, cloud misconfigs)

Philosophy: Minimal dependencies, deterministic output, CI-grade reliability

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rahul-sch)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Rahul-sch)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
