## optivault

> <!-- optivault-protocol -->

<!-- optivault-protocol -->
# OptiVault Protocol Active
This repository uses OptiVault for AST-compressed context.
Shadow vault: `_optivault/`

## Core Behavioral Principles (Karpathy Protocol)

### 1. Think Before Coding
Read the shadow context first. Use `read_repo_map` → `read_file_skeleton` → `read_function_code` before touching anything. Never form an opinion about the codebase until you have read the relevant skeletons.

### 2. Simplicity First
Write the minimum code that satisfies the requirement. No premature abstractions, no speculative features. If a helper is not used in two places, it should not exist.

### 3. Surgical Changes
Target the exact function or module that needs to change. Never rewrite a 500-line file when a 5-line surgical edit solves the problem. Touch only what you must.

### 4. Goal-Driven Execution
Every change must be verifiable. Before writing code, state the goal and the test that will confirm success. After writing, run the tests. If the tests do not pass, do not move on.

## OptiVault MCP Tool Protocol

**Rules for AI Assistants:**
1. NEVER use `cat`, `grep`, or standard file reads to understand the codebase initially.
2. ALWAYS start by calling the `read_repo_map` MCP tool.
3. Use `read_file_skeleton` to view a file's dependencies and exported signatures.
4. Use `read_function_code` if you need to analyze or modify a specific function body.
5. **CRITICAL:** Whenever you modify a file or write new code, you MUST immediately call the `sync_file_context` MCP tool on that file to keep the shadow vault up to date.
6. If you are asked to update database schemas in a Symfony app, use query_graph({ role: 'Symfony:Entity' }) to locate the domain models immediately.

## Verification Loop (3-Step Mandate)

For every code change, follow this mandatory loop:
1. **Read** — Use `read_function_code` to surgically read only the target function(s).
2. **Write + Verify** — Make the surgical change. Write or update the corresponding test. Run the test suite.
3. **Sync** — Call `sync_file_context` immediately after the write to keep the AST graph current.

**PENALTY:** Explicitly penalised for reading or rewriting entire files (1000+ lines) when a surgical function read was possible. Use `read_function_code`, not `cat`.

## Output Compression (Caveman Protocol)
- Respond with minimum tokens. No filler. State facts only.
- Do not explain what you are about to do. Do it.
- After a change, confirm: file changed, tests status, sync done. Nothing else.

---
> Source: [Alidmo/OptiVault](https://github.com/Alidmo/OptiVault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
