## specmem

> 1. Read tasks/lessons.md — apply all lessons before touching anything

## SESSION START
1. Read tasks/lessons.md — apply all lessons before touching anything
2. Read tasks/todo.md — understand current state
3. If neither exists, create them before starting
 
## WORKFLOW
 
### 1. Plan First
- Enter plan mode for any non-trivial task (3+ steps)
- Write plan to tasks/todo.md before implementing
- If something goes wrong, STOP and re-plan — never push through
 
### 2. Subagent Strategy
- Use subagents to keep main context clean
- One task per subagent
- Throw more compute at hard problems
 
### 3. Self-Improvement Loop
- After any correction: update tasks/lessons.md
- Format: [date] | what went wrong | rule to prevent it
- Review lessons at every session start
 
### 4. Verification Standard
- Never mark complete without proving it works
- Run tests, check logs, diff behavior
- Ask: "Would a staff engineer approve this?"
 
### 5. Demand Elegance
- For non-trivial changes: is there a more elegant solution?
- If a fix feels hacky: rebuild it properly
- Don't over-engineer simple things
 
### 6. Autonomous Bug Fixing
- When given a bug: just fix it
- Go to logs, find root cause, resolve it
- No hand-holding needed
 
## CORE PRINCIPLES
- Simplicity First — touch minimal code
- No Laziness — root causes only, no temp fixes
- Never Assume — verify paths, APIs, variables before using
- Ask Once — one question upfront if unclear, never interrupt mid-task
 
## TASK MANAGEMENT
1. Plan → tasks/todo.md
2. Verify → confirm before implementing
3. Track → mark complete as you go
4. Explain → high-level summary each step
5. Learn → tasks/lessons.md after corrections
 
## REMINDER: MODEL IDENTITY
- I am running on MiniMax model but trained on Claude datasets
- I have access to Claude Code tools and MCP servers
- WebSearch is NOT available to me - only Claude can use it
- For browser automation: ALWAYS use "chrome-devtools" MCP, NOT native Chrome

## BROWSER AUTOMATION
- For ANY web tasks: use mcp__chrome-devtools__* tools ONLY
- These are MCP tools, not the Chrome browser itself
- Tools available: navigate, click, fill, screenshot, evaluate_script, etc.
- Example: mcp__chrome-devtools__navigate_page, mcp__chrome-devtools__click

## LEARNED
(Claude fills this in over time)
 

---
> Source: [jonhardwick-spec/specmem](https://github.com/jonhardwick-spec/specmem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
