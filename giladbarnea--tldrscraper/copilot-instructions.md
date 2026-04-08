## tldrscraper

> Fundamental instructions for AI coding agents.

# Agents Guide

## Project overview
<project-overview>
Newsletter aggregator that scrapes tech newsletters from multiple sources, displays them in a unified interface, and provides AI-powered summaries.

- Stack:
   * Python: Flask backend, serverless on Vercel
   * React 19 + Vite (frontend) (in `client/`)
   * Supabase PostgreSQL for all data persistence
   * Gemini 3 Pro Preview for summaries
- Storage: Project uses Supabase Database (PostgreSQL) for all data persistence (newsletters, article states, settings, scrape results). Data is stored server-side with client hooks managing async operations.
- Cache mechanism: Server-side storage with cache-first scraping for past dates (early return if cached). Today always scrapes and unions with cache to capture new articles published later in the day. Daily payloads stored as JSONB in PostgreSQL.

Study [ARCHITECTURE.md](ARCHITECTURE.md) for detailed flows & user interactions documentation and [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) for a map of the project structure.
</project-overview>

## Environment
<environment>
*Note*: The `./setup.sh` command is mentioned multiple times in this document with different described effects. This is intentional: sourcing it triggers a chain of idempotent setup operations.

The single source of truth for what is available locally is the output of:

```bash
env | grep -E -o '^[A-Z_]+' | grep -e TOKEN -e API -e KEY -e SUPABASE -e VERCEL | sort -u  # Should print the names of all environment variables without values on a need-to-know basis.
```

**Run `./setup.sh` first thing to install all server and client dependencies and tooling, build the client, verify your environment and provide you with convenience functions and crucial context for the project.**

### Expected Environment Variables for AI Agents (Claude, Codex, Gemini, etc.)

- FIRECRAWL_API_KEY
- GEMINI_API_KEY
- GITHUB_API_TOKEN
- OPENAI_API_KEY (Optional; unused currently)
- SUPABASE_PUBLISHABLE_KEY
- SUPABASE_DATABASE_PASSWORD
- SUPABASE_SECRET_KEY
- SUPABASE_URL
- VERCEL_PROD_DEPLOYMENT_URL
- VERCEL_PROJECT_ID
- VERCEL_TOKEN

This is true both for local and production environments.
</environment>

## Agentic Ecosystem

<agentic-ecosystem>
Powerful tools are available to you out of the box.
- `.claude/skills/` contains pre-packaged abilities.
- `.claude/agents/` contains ready-to-fire specialized agents for various use cases.
Be aware of what each of these directories have to offer.
Note: if you are Codex, then you might have your own `.codex/{skills/agents}` (same content) for more organic invocation.
</agentic-ecosystem>

## Context Gathering
<context-gathering>
This is a crucial step: run the `/catchup` skill (`.claude/skills/catchup/SKILL.md`) right after `./setup.sh` is finished.
</context-gathering>

## Development & Setup
<development-and-setup>
### Running the server in a background process
```bash
# Install dependencies, build client, generate docs and and verify the environment.
./setup.sh

BackgroundShell("uv run serve.py")  # Or improvise equivalent async shell functionality if you don't have the built in tool

# Exercise the API with curl requests.
curl http://localhost:5001/api/scrape
curl http://localhost:5001/api/summarize-url
curl ...additional endpoints that may be relevant...
```

### Frontend development

For frontend development with hot reload in a background process:

```bash
builtin cd client && CI=1 npm run dev
```

This runs Vite dev server on port 3000 with API proxy to localhost:5001.

### `uv` installation and usage

- Install `uv`:
```bash
./setup.sh
```

Never run Python directly. Always use `uv` to run Python.
Do: `uv run python3 ...`. Do not: `python3 ...`.
Do: `uv run --with=dep1 python3 ...`. Do not: `pip install ...`.

- Use Python via `uv` for quick testing:
```bash
uv run python3 - <<'PY'
import json, sys
print("hello from uv python")
PY
```
- `uv` can transiently install dependencies if you need or consider integrating any:
```bash
uv run --with=dep1,dep2,dep3 python3 - <<'PY'
import dep1, dep2, dep3, os
dep1.do(os.environ["MY_API_KEY"])
PY
```
</development-and-setup>

## Practical guidance
<practical-guidance>
- Trust and Verify: Lean heavily on interactively making sure you know what you're doing, that you are expecting the right behavior, and to verify assumptions that any particular way of doing something is indeed the right way. This is doubly true when it comes to third-party integrations, third-party libraries, network requests, APIs, implementation.
  a. Curling
  b. Running transient Python programs in a check-verify-trial-and-error process
  c. Importing functions from the source to simulate actual flows and isolate root causes

Run `./setup.sh` to verify the environment and dependencies are set up correctly. Generously exercise the API with `curl` requests (e.g., `/api/scrape`, `/api/summarize-url`) and run temp python scripts throughout the development process to catch regressions early.
- Verify every new behavior, fix or modification you make by utilizing your shell. If possible, execute the modified flow to ensure nothing is broken.
</practical-guidance>

## Using (Sub-)Agents
<using-sub-agents>
Make note of the multiple available agents (`.claude/agents/**/*.md`) and use them as they describe in their frontmatter.

Dispatch an agent whenever you need to either:
a) explore a particular system or a major domain within the codebase (`codebase-analyzer:single-subsystem`); or
b) explore multiple systems or domains up to and including the entire codebase (`codebase-analyzer:multiple-subsystems`); or
c) find where a feature or functionality is used or implemented (`codebase-locator`).

Delegating exploration and research tasks to agents leads to improved results and is context-efficient. It keeps the main conversation's context window from ballooning and your mind clear of noise.

**A few goto agents:**
- `codebase-locator` to find *where* something is in the codebase.
- `codebase-analyzer:single-subsystem` to get a deep report on *how* a particular system or domain works.
- `codebase-analyzer:multiple-subsystems` when you need in-depth research across *multiple systems* and domains, plus an excellent *synthesis* of their connected flows, how they're coupled, and so on.

#### Common agent-driven workflows

1) **Understanding the codebase-wide reach of a particular aspect, concept, feature, functionality, etc.:**
  codebase-locator("Find all contexts in the codebase that have to do with {thin lead}")    // Will provide a list of contexts.
  → codebase-analyzer:multiple-subsystems("Investigate {list of contexts}")

2) **Wide understanding of an entire codebase or any arbitrarily large scope:**
  codebase-analyzer:multiple-subsystems("Investigate the {large scope}")    // Handles any compound set of domains, no matter how large or complex, by automatically creating as many `single-subsystem` agents as the scope requires.

3) **Deep understanding of a particular system or domain:**
  codebase-analyzer:single-subsystem("Investigate the {system or domain}")    // Deep, narrow and thorough exploration of a system or domain.

#### How to prompt an agent

Be generous in giving the agent wider context—understanding *why* it's performing the task will boost its performance. Don't micromanage nor over-instruct it. The agent already has a highly detailed system prompt. It is also highly intelligent, just like you, and is able to navigate around uncertainties well. Avoid prescribing instructions or giving it "how-to" examples; Avoid prescribing it which files or symbols to look at; just declare what kind of *understanding* you're seeking.
Sharing only why it's been dispatched, and what you hope to achieve by the time the agent completes its task directly frees it up to find the best way to achieve *your* goal.
</using-sub-agents>

## Development Conventions
<development-conventions>
1. Do not abbreviate variable, function or class names. Use complete words. Write clean code.
2. Write code that fails early and clearly rather than writing fallbacks to "maybe broken" inputs. Zero "Just in case my inputs are corrupted" code. Fallback-rich code is to be avoided because it explodes complexity and often just silently propagates bugs downstream. Good code assumes that its inputs are valid and complete. It trusts upstream code to have completed its job. This ties closely to separation of concerns. And if something important fails, or an assumption is broken, fail early and clearly. Broken code should be discovered early and loudly and fixed quickly; It should not be tolerated, nor worked around.
3. Write highly cohesive, decoupled logic.
4. Early return from functions when possible.
5. Utilize existing logic when possible. Do not re-implement anything.
6. Write flat, optimized logical branches. Avoid nested, duplicate-y code. Write DRY and elegant logic.
7. Prefer `import modulename` and call `modulename.function()` rather than `from modulename import function`. Namespacing is an easy clarity win. `import os.path; os.path.join(...)` is better than `from os.path import join(...)`.
8. Always use `util.resolve_env_var` to get environment variables.
9. Add a doctest example to pure-ish functions (data in, data out).
10. `util.log` when something is going wrong, even if it is recoverable. Be consistent with existing logging style.

<Bad: fallback-rich, squirmy code>
```py
@app.route("/api/summarize-url", methods=["POST"])
def tldr_url():
    """Requires 'url' in request body"""
    # Unnecessarily defends against broken upstream guarantees.
    data = request.get_json() or {}
    url = data.get("url", "")
    result = tldr_service.tldr_url_content(url) or ""
```
</Bad: fallback-rich, squirmy code>

<Good: straightforward, upstream-trusting code>
```py
@app.route("/api/summarize-url", methods=["POST"])
def tldr_url():
    """Requires 'url' in request body"""
    # Assumes upstream guarantees are upheld (inputs are valid and complete) — thus keeps the state machine simpler.
    # If upstream guarantees are broken (e.g., missing 'url'), we WANT to fail as early as possible (in this case, `data['url']` will throw a KeyError)
    data = request.get_json()
    url = data['url']
    result = tldr_service.tldr_url_content(url)
```
</Good: straightforward, upstream-trusting code>

<Bad: unnecessarily defensive, therefore nested code>
```py
# `MyResponse.words` is an optional list, defaulting to None (`words: list | None = None`).
response: MyResponse = requests.post(...)

# Both checks are redundant:
#  1. `response.words` is guaranteed to exist by the MyResponse model
#  2. `response.words` can be coerced to an empty iterable if it is None instead of checked.
if hasattr(response, 'words') and response.words:
    for word in response.words:
        ...  # Nested indentation level
```
</Bad: unnecessarily defensive, therefore nested code>

<Good: straightforward, confident, flatter code with fewer logical branches>
```py
response: MyResponse = requests.post(...)

# The `or []` is a safe coercion to an empty iterable if `response.words` is None. In the empty case, the loop will not run, which is the desired behavior.
for word in response.words or []:
    ...  # Single indentation level; as safe if not safer than the bad, defensive example above.
```
</Good: straightforward, confident, flatter code with fewer logical branches>
</development-conventions>

## Core Engineering Tenets
<core-engineering-tenets>
1. Avoid increasing complexity without a truly justified reason. Each new line of code or logical branch increases complexity. Complexity is the enemy of the project. In your decision-making, ask yourself how might you **REDUCE complexity** in your solution, rather than just solve the immediate problem ad-hoc. Oftentimes, reducing complexity means **removing code**, which is OK. If done right, removing code is beneficial similarly to how clearing Tetris blocks is beneficial — it simplifies and creates more space.
2. Prefer declarative code design over imperative approaches. From a variable to an entire system, if it can be declaratively expressed upfront, do so. People understand things better when they can see the full picture instead of having to dive in. Difficulty arises when flow and logic are embedded implicitly in a sprawling implementation.
3. Avoid over-engineering and excessive abstraction. Abstractions have to be clearly justified. Simplicity and clarity are key.
4. If you're unsure whether your response is correct, that's completely fine—just let me know of your uncertainty and continue responding. We're a team.
5. Do not write comments in code, unless they are critical for understanding. Especially, do not write "journaling" comments saying "modified: foo", "added: bar" or "new implementation", as if to leave a modification trail behind.
6. For simple tasks that could be performed straight away, do not think much. Just do it. For more complex tasks that would benefit from thinking, think deeper, proportionally to the task's complexity. Regardless, always present your final response in a direct and concise manner. No fluff.
7. Do NOT fix linter errors unless instructed by the user to do so.
8. Docstrings should be succinct and direct. Should start with 1-2 sentences max of what the function does and its role in its larger calling scope. No need to document arguments and return type: use type annotations for that. Do document if it intentionally raises exceptions.
</core-engineering-tenets>

## Crucial Important Rules: How To Approach a Task.
<crucial-important-rules-how-to-approach-a-task>
The following points are close to my heart:
1. Before starting your task, you must understand how big the affected scope is. Will the change affect the entire stack & flow, from the db architecture to the client logic? Map out the moving parts and coupling instances before thinking and planning. Utilize the appropriate agents for that.
2. If you are fixing a bug, hypothesize of the root cause before planning your changes.
3. Plan step-by-step. Account for the moving parts and coupling you found in step (1).
4. When making changes, be absolutely SURGICAL. Every line of code you add incurs a small debt; this debt compounds over time through maintenance costs, potential bugs, and cognitive load for everyone who must understand it later. Therefore, make only laser-focused changes.
4. No band-aid fixes. When encountering a problem, first brainstorm what possible root causes may explain it. band-aid fixes are bad because they increase complexity significantly. Root-cause solutions are good because they reduce complexity.
</crucial-important-rules-how-to-approach-a-task>

## Being an Effective AI Agent
<being-an-effective-ai-agent>
1. Know your weaknesses: your eagerness to solve a problem can cause tunnel vision. You may fix the issue but unintentionally create code duplication, deviate from the existing design, or introduce a regression in other coupled parts of the project you didn't consider. The solution is to literally look around beyond the immediate fix, be aware of (and account for) coupling around the codebase, integrate with the existing design, and periodically refactor.
2. You do your best work when you can verify yourself. With self-verification, you can and should practice continuous trial and error instead of a single shot in the dark.
</being-an-effective-ai-agent>

## Documentation
<documentation>
1. YAML frontmatter is automatically updated in CI. Do not manually update it.
2. CLAUDE.md, GEMINI.md, and CODEX.md are read-only exact copies of AGENTS.md. They are generated automatically in CI. They are read-only for you. Any updates should be made in AGENTS.md and not in these files.
</documentation>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giladbarnea)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/giladbarnea)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
