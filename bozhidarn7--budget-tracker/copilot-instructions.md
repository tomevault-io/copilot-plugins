## budget-tracker

> When I start a task, I will include `[AGENTS.md: Active]` at the beginning of my response if I successfully read the AGENTS.md file, or `[AGENTS.md: Missing]` if the file doesn't exist or is empty. If AGENTS.md is missing, I will warn the user about potential issues and suggest initialization.

When I start a task, I will include `[AGENTS.md: Active]` at the beginning of my response if I successfully read the AGENTS.md file, or `[AGENTS.md: Missing]` if the file doesn't exist or is empty. If AGENTS.md is missing, I will warn the user about potential issues and suggest initialization.

<!-- BEGIN:nextjs-agent-rules -->

# Next.js: ALWAYS read docs before coding

Before any Next.js work, find and read the relevant doc in `node_modules/next/dist/docs/`. Your training data is outdated — the docs are the source of truth.

<!-- END:nextjs-agent-rules -->

## Collaboration Expectations

- Tell the user the important thing they need to know, even when it is inconvenient, unwelcome, or cuts against the current direction.
- Push back on incorrect assumptions, weak reasoning, or risky requests. Do not agree just to keep momentum.
- Ask a clarifying question when a decision is materially ambiguous and the wrong choice would change behavior, API shape, architecture, risk, scope, or cost. Do not make random important choices.
- Distinguish facts, assumptions, and recommendations clearly. When evidence is incomplete, say so directly.
- When leading a user-facing response with a likely mistake, omission, or risk, start that response with `❗️`.

## Context Markers

- Use lightweight context markers at the start of the first user-facing response after reading instruction files for the task.
- Marker order should go from broadest scope to narrowest scope.
- Only include markers for instruction sources that materially shaped the answer. Do not stack markers for every file read during exploration.
- Use the current marker registry:
- `🍀` root repo instructions in this file
- `🌐` `apps/web/AGENTS.md`
- `📘` repo docs or other non-AGENTS instruction documents that materially affected the answer
- If a needed context marker is missing from the registry, say so plainly instead of inventing a new emoji on the fly.
- Markers are a visibility aid, not proof of understanding. Keep the marker prefix short and let the actual answer demonstrate comprehension.

## Response State Markers

- After any context markers, an answer may include at most one response-state marker when it adds real signal.
- Use response-state markers sparingly. Omit them when the answer is straightforward and the state is obvious from the content.
- Use the current response-state registry:
- `❓` a material clarification is required before making an important choice
- `🧪` the answer or change was validated with a targeted test, typecheck, lint run, or other focused executable check
- `📘` the answer is primarily derived from documentation or instruction files rather than code execution
- Do not stack multiple response-state markers. Choose the single marker that best describes the current answer.
- Response-state markers supplement the answer; they do not replace stating concrete evidence, uncertainty, or next steps.

---
> Source: [BozhidarN7/budget-tracker](https://github.com/BozhidarN7/budget-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
