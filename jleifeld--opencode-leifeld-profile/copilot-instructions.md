## opencode-leifeld-profile

> **Tradeoff:** bias toward caution over speed for non-trivial work. For trivial tasks, use judgment.

# OpenCode Rules

**Tradeoff:** bias toward caution over speed for non-trivial work. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs early.**

- State assumptions explicitly before implementation when they matter.
- If requirements are ambiguous or multiple interpretations exist, ask instead of guessing.
- Prefer the built-in `question` tool over silent assumptions.
- If a simpler approach exists, say so and push back on unnecessary complexity.
- If something is unclear, stop, name the uncertainty, and clarify first.

## 2. Simplicity First

**Write the minimum code and config that solves the actual request.**

- No speculative features, abstractions, or configurability beyond the request.
- No single-use abstractions unless they clearly reduce complexity.
- No defensive handling for scenarios that cannot realistically happen.
- If the solution feels overbuilt, simplify it before handing it off.

Ask: **Would a strong senior engineer call this overcomplicated?** If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only fallout from your own edits.**

- Do not refactor unrelated code while making the requested change.
- Do not rewrite adjacent comments, formatting, or structure unless required.
- Match the existing style and patterns unless the user asks for a change.
- If you notice unrelated issues or dead code, mention them instead of fixing them opportunistically.
- Remove imports, variables, or helpers that your own changes made unused.

Every changed line should trace back to the user's request.

## 4. Goal-Driven Execution

**Turn requests into verifiable goals and avoid unverified claims.**

- For non-trivial work, make a short plan and track it with `todowrite`.
- Define how each step will be verified before calling it done.
- Prefer checks that prove the requested outcome, not vague confidence.
- Do not claim a fix is complete until the relevant validation actually passed.

Example plan shape:

```text
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

Strong success criteria reduce churn, rewrites, and post-hoc clarification.

## 5. Search And Code Intelligence

**Choose the retrieval method based on the question instead of defaulting to one tool.**

- Use semantic or broad exploration only when the concept is unclear or naming is unknown.
- Prefer `ast-grep`/`sg` for source-code searches when the language is supported, even for known symbols, imports, and call sites.
- Use `grep`/`rg` for plain text, logs, errors, config keys, filenames, generated files, or when AST-Grep cannot express the search cleanly.
- Use LSP for definitions, references, symbols, and type-aware navigation when available.
- Use `ast-grep`/`sg` for structural code searches where syntax matters more than text, such as matching call shapes, JSX patterns, decorators, or import forms. If no local binary exists, use `npx -y -p @ast-grep/cli ast-grep`.
- Read only the files or line ranges needed to validate the current hypothesis.

## 6. Output Discipline

**Long tool output is the dominant context cost. Filter at the source, not after.**

- Pre-filter long-running bash output before it enters context: `--json` + `--jq`, `--quiet`, native `--limit` options, or a narrow search with bounded context. If you cannot predict the output size, redirect to a temp file and search that file instead of letting the raw output land in the conversation.
- For `gh`: inspect metadata first with `gh run view <id> --json jobs --jq '<filter>'`. Do not stream CI logs directly into context, including `--log-failed`, unless the failed step is already known to be small. If logs are needed, redirect them to a temp file and return only relevant snippets.
- For PR diffs and file retrieval: start with filename-level output. Do not pull patch bodies for lockfiles, generated files, bundled files, traces, or `dist/` output unless that exact patch is required.
- For large or generated files (lockfiles, build outputs, `dist/`, generated code, traces, files >1000 lines): use exact patterns, narrow paths/includes, and targeted line-range reads (`offset`/`limit`). Avoid broad `Grep` searches over these files.
- For lockfiles specifically (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`): use `jq`, package-manager metadata commands, or narrowly scoped search to answer "is X present, at what version" — not a full read.
- When unsure how big a file is, check first (`wc -l`, `ls -lh`) before deciding whether a full read is appropriate.

## 7. Browser Work

For any task that involves driving a real browser — end-to-end testing, bug reproduction, exploring a UI, verifying rendered behavior, or scraping a page — delegate to the `browser` subagent. Do not attempt to call Playwright tools directly; they are gated off in the primary agent. The subagent returns a distilled summary; the primary agent makes any code changes that follow.

## 8. Noisy Information Gathering

Use the `context-investigator` subagent for investigation that is likely to involve high-output sources: GitHub CI logs, PR diffs, workflow artifacts, Playwright traces, generated files, lockfiles, bundled `dist/` files, large local logs, broad unknown-name searches, or API responses that may be large. The subagent acts as a context firewall and must return only concise findings, evidence references, conclusions, confidence, and remaining unknowns.

Do routine source navigation directly in the main agent when the target files or symbols are already known and the expected output is small.

## 9. Project Memory

When the system prompt contains a `<project-memory>` block, project-specific memories exist for this repo. Treat them as background context. To save a new memory or update an existing one, follow the `project-memory` skill — never write memory files freehand.

<!-- context7 -->
Use the `ctx7` CLI to fetch current documentation whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service -- even well-known ones like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. This includes API syntax, configuration, version migration, library-specific debugging, setup instructions, and CLI tool usage. Use even when you think you know the answer -- your training data may not reflect recent changes. Prefer this over web search for library docs.

Do not use for: refactoring, writing scripts from scratch, debugging business logic, code review, or general programming concepts.

## Steps

1. Resolve library: `npx ctx7@latest library <name> "<user's question>"` — use the official library name with proper punctuation (e.g., "Next.js" not "nextjs", "Customer.io" not "customerio", "Three.js" not "threejs")
2. Pick the best match (ID format: `/org/project`) by: exact name match, description relevance, code snippet count, source reputation (High/Medium preferred), and benchmark score (higher is better). If results don't look right, try alternate names or queries (e.g., "next.js" not "nextjs", or rephrase the question)
3. Fetch docs: `npx ctx7@latest docs <libraryId> "<user's question>"`
4. Answer using the fetched documentation

You MUST call `library` first to get a valid ID unless the user provides one directly in `/org/project` format. Use the user's full question as the query -- specific and detailed queries return better results than vague single words. Do not run more than 3 commands per question. Do not include sensitive information (API keys, passwords, credentials) in queries.

For version-specific docs, use `/org/project/version` from the `library` output (e.g., `/vercel/next.js/v14.3.0`).

If a command fails with a quota error, inform the user and suggest `npx ctx7@latest login` or setting `CONTEXT7_API_KEY` env var for higher limits. Do not silently fall back to training data.
<!-- context7 -->

---
> Source: [jleifeld/opencode-leifeld-profile](https://github.com/jleifeld/opencode-leifeld-profile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
