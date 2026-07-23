## nanoclaw

> Four pillars. Every line of code must serve at least one. If it serves none, delete it.

# agents.md - Rules for AI Agents Working on nanoClaw

## Core Principles

Four pillars. Every line of code must serve at least one. If it serves none, delete it.

1. **SECURE** - The user trusts us with shell access to their machine. Every input is hostile until proven safe. Defense in depth: if one layer fails, the next catches it. No security shortcuts, no "we'll fix it later".

2. **LIGHTWEIGHT** - Target: ~3000 lines total. Every file has a line budget (see below). No abstractions without 2+ concrete uses. No wrapper classes that just delegate. If stdlib does it, don't import a package.

3. **FAST** - The user is waiting in Telegram. Target: first response token in <2 seconds for simple queries. Shared connections, async everything, minimal context window usage. Don't send 50 messages of history when 15 is enough. Don't call LLM to extract memories if the message is "thanks".

4. **EFFECTIVE** - The agent must actually solve problems. Bias toward action: call tools, don't describe what tools you could call. Get it done in fewer iterations. A 1-iteration answer is better than a 4-iteration answer if the result is the same.

## Code Size

Target: ~3000 lines total. This is a guideline, not a hard limit per file.

The rule is simple: **every line must earn its place.** If a function can be 10 lines instead of 20 with the same clarity and safety, use 10. But never sacrifice error handling, security checks, or readability to save lines.

If a file feels bloated (300+ lines), that's a signal to split it. But don't split prematurely -- a clean 280-line file is better than three 100-line files with awkward imports.

## Project Language

Everything in English: code, comments, commits, docs, README, variable names, error messages, CLI output. No exceptions.

## Architecture

- The source of truth for architecture is `nanoclaw-spec-v2.md`. Read it before writing any code.
- If you change architecture decisions (add a module, change a dependency, rename something), update the spec file too.
- Keep `ARCHITECTURE.md` in the repo root with a current high-level overview: modules, data flow, dependencies. Update it when structure changes.

## Code Style

- No em dashes (—), en dashes (–), or unicode arrows (→, ←, ↔) anywhere in source code, comments, docstrings, or string literals. Use standard ASCII: `--`, `->`, `<-`, `=>`.
- No smart quotes (" " ' '). Use straight quotes only (" and ').
- No emoji in source code or logs. Emoji are allowed ONLY in user-facing Telegram messages and CLI output where explicitly specified in the spec.
- Type hints on all function signatures. Use `from __future__ import annotations` where needed.
- Docstrings on every public class and function. One-liner is fine for simple ones.
- Max line length: 100 characters.
- Use `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants.
- Imports: stdlib first, then third-party, then local. Separated by blank lines.
- No wildcard imports (`from x import *`).
- Prefer explicit over implicit. No magic. If something is unclear, add a comment.

## Git Commits

- Never mention Claude, AI, LLM, Copilot, or any AI tool in commit messages. Write commits as if a human developer wrote the code.
- Commit message format: `type: short description` where type is one of: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`.
- Examples: `feat: add shell sandbox with three-tier filtering`, `fix: handle timeout in web_fetch`, `docs: update architecture diagram`.
- Keep commits atomic. One logical change per commit. Don't lump unrelated changes together.
- Don't commit generated files, `__pycache__`, `.pyc`, `config.json`, or workspace contents.

## Dependencies

- Minimize dependencies. Every new package needs justification.
- Pin versions in pyproject.toml with minimum bounds (`>=`), not exact pins (`==`).
- No heavy frameworks: no langchain, llamaindex, fastapi, flask, django, react.
- Before adding a dependency, check if stdlib can do it. `sqlite3`, `json`, `asyncio`, `pathlib`, `re`, `datetime`, `importlib` are all stdlib.
- If you add a dependency, add it to pyproject.toml AND mention it in ARCHITECTURE.md.

## Error Handling

- Every `async with`, `await`, and external call must have proper try/except.
- Never catch bare `Exception` and silently pass unless it's explicitly a best-effort operation (like background memory extraction).
- User-facing errors should be clear and actionable: "Web search failed: invalid API key. Run `nanoclaw init` to reconfigure." not "Error occurred."
- Log errors with `logger.error()` including the exception. Don't just print.
- All tools must return a string result, never raise unhandled exceptions to the agent loop.

## Security

- Security code is the most critical part of the project. Be extra careful.
- Never log API keys, tokens, or passwords. Truncate or mask them if needed for debugging.
- Never add API keys or secrets to default config, examples, tests, or README.
- All file paths go through `FileGuard.validate_path()`. No exceptions, no shortcuts.
- All shell commands go through `ShellSandbox`. No direct `subprocess` or `os.system` calls anywhere else.
- All tool outputs go through `PromptGuard.sanitize_tool_output()` before being added to LLM context.
- Test security code with adversarial inputs: path traversal, command injection, prompt injection.

## Testing

- Write tests for security modules first (sandbox, file guard, prompt guard).
- Use `pytest` and `pytest-asyncio`.
- Tests should be runnable without API keys or network (mock external calls).
- Test file: `tests/test_<module_name>.py` mirroring `nanoclaw/<module_name>.py`.
- Each security layer needs at least: 3 positive cases (allowed), 3 negative cases (blocked).

## File Organization

- Follow the file structure from the spec exactly. Don't add extra directories or reorganize without updating the spec.
- Each file should have a single clear responsibility.
- If a file exceeds 300 lines, consider splitting it. Exception: `dashboard/index.html` and test files.
- `__init__.py` files should be minimal: version info and key re-exports only.

## Performance

- All I/O is async. No blocking calls in the event loop.
- **Shared aiohttp session.** Create ONE `aiohttp.ClientSession` at startup, reuse everywhere (LLM calls, web_search, web_fetch, skills). Don't create a new session per request. Close on shutdown.
- SQLite operations should be wrapped in async (use `asyncio.to_thread` for DB calls or `aiosqlite`).
- Truncate large strings before storing in DB or sending to LLM. Limits defined in spec.
- **Context window discipline.** Don't stuff everything into the prompt:
  - History: last 15 messages, not 50 (50 is the DB storage limit, not context limit)
  - Memories: top 5 most relevant, not 10
  - Tool results: truncate at 4000 chars, not 10000
  - System prompt: under 500 tokens
  - If total context approaches 8000 tokens, trim history first
- **Lazy imports.** Optional dependencies (chromadb, croniter) must be imported inside the function that uses them, not at module top level. This keeps startup under 1 second.
- **Smart memory extraction.** Don't call LLM to extract memories on every message. Only trigger when message matches keyword patterns (already in spec). Skip for: greetings, single-word replies, questions, follow-ups.
- **Connection timeouts.** LLM: 30s. Web fetch: 15s. Brave search: 10s. Shell: 30s. Never wait longer.
- **No polling waste.** Telegram polling interval: 1s when recent activity, back off to 3s after 60s idle. Don't hammer Telegram API.

## Documentation

- `README.md` is the primary user-facing doc. Keep it practical: install, configure, use.
- Code comments explain WHY, not WHAT. The code shows what; comments show reasoning.
- If you make a non-obvious design decision, add a comment starting with `# NOTE:` explaining why.
- Don't leave TODO comments without a concrete description of what needs to be done.

## What NOT to Do

- Don't add features not in the spec without explicit approval.
- Don't refactor working code just to make it "cleaner" without a functional reason.
- Don't add logging to every single line. Log at entry/exit of important operations and on errors.
- Don't create abstract base classes or interfaces unless there are at least 2 concrete implementations.
- Don't use dataclasses when pydantic BaseModel is already in the project (pick one, we use pydantic).
- Don't add retry logic everywhere. One retry on LLM calls is fine. Tools should fail fast.
- Don't over-engineer. This is a ~3000 line project. Keep it simple.

## Effectiveness

The agent must solve problems, not talk about solving them. These rules ensure that.

- **System prompt drives behavior.** The system prompt tells the LLM to act, not describe. Every revision must keep "bias toward action" and "minimize iterations" as the first two rules.
- **Tool schemas matter.** Write tool descriptions that guide the LLM to pick the right tool. Bad: "Can be used to search the web." Good: "Search the internet for current information. Returns top 5 results with titles, URLs, and snippets."
- **One iteration is best.** If the user says "what time is it in Tokyo", the agent should call get_time in iteration 1 and return the result in iteration 2. Two iterations total. Not: think about it, search the web, then call the tool.
- **Fail fast, fail clear.** If a tool fails, return a useful error immediately. Don't retry 3 times, don't try a different approach unless the user asks.
- **Memory makes the agent smarter over time.** Save facts that will be useful later: user preferences, project names, tech stack, timezone. Don't save: "user said hi", "user asked about weather", trivial exchanges.

## Token Efficiency

This is nanoClaw's key competitive advantage. OpenClaw bleeds tokens. We don't.

**The problem with OpenClaw:** Every message sends ALL 100+ skill descriptions to the LLM (~5000-8000 tokens just for tool schemas). Full conversation history with no windowing. No truncation of tool outputs. No budget tracking. Users report $10-30/day on active use.

**Our target:** Average message costs under $0.01. Heavy tool-use messages under $0.05. Daily active use under $1-2.

### Strategy 1: Dynamic Tool Injection

Don't send all tools every time. The LLM doesn't need weather, github, and timezone tools when the user asks "summarize this URL".

```python
# In context.py -- select only relevant tools for this message
def select_tools(self, user_message: str, all_tools: list) -> list:
    """
    Send core tools always (5 tools, ~600 tokens).
    Add skills only when message hints at them (~200 tokens each).
    
    Core (always): web_search, web_fetch, shell_exec, file_read, file_write
    Conditional: memory_save, memory_search, spawn_task, file_list
    Skills: only when keyword match
    """
    core_tools = ['web_search', 'web_fetch', 'shell_exec', 'file_read', 'file_write']
    
    selected = [t for t in all_tools if t['function']['name'] in core_tools]
    
    msg_lower = user_message.lower()
    
    # Add memory tools if message is personal or references memory
    if any(w in msg_lower for w in ['remember', 'my ', 'i am', 'i work', 'i like',
                                      'recall', 'forgot', 'you know']):
        selected.extend(t for t in all_tools 
                       if t['function']['name'] in ('memory_save', 'memory_search'))
    
    # Add spawn if message implies long task
    if any(w in msg_lower for w in ['research', 'analyze', 'compare', 'background',
                                      'deep dive', 'report on', 'investigate']):
        selected.extend(t for t in all_tools 
                       if t['function']['name'] == 'spawn_task')
    
    # Add skills by keyword matching
    skill_triggers = {
        'weather': ['weather', 'temperature', 'rain', 'forecast'],
        'github': ['github', 'repo', 'repository', 'pull request', 'issue', 'pr'],
        'news': ['news', 'headlines', 'latest'],
        'get_time': ['time in', 'timezone', 'what time'],
        'summarize_url': ['summarize', 'summary', 'tldr'],
    }
    for tool_name, triggers in skill_triggers.items():
        if any(t in msg_lower for t in triggers):
            selected.extend(t for t in all_tools 
                          if t['function']['name'] == tool_name)
    
    # Always include file_list if file operations are in selected
    if any(t['function']['name'] in ('file_read', 'file_write') for t in selected):
        selected.extend(t for t in all_tools 
                       if t['function']['name'] == 'file_list' 
                       and t not in selected)
    
    return selected
```

**Token savings:** Typical message sends 5-7 tools (~800 tokens) instead of 14+ (~2500 tokens). Saves ~1700 tokens per LLM call. Over a 3-iteration conversation, that's ~5000 tokens saved.

### Strategy 2: Smart History Windowing

```python
# In context.py
def build_history(self, session_id: str, current_message: str) -> list:
    """
    Adaptive history window. Not fixed -- based on what's useful.
    
    Rules:
    - Last 4 messages: always include (immediate context)
    - Messages 5-15: include only if they contain tool calls or long responses
      (these are the "important" turns)
    - Messages 16+: drop entirely (use memory for older context)
    - Truncate any single message over 1000 chars to 1000 chars
    """
```

**Comparison:**
- OpenClaw: sends ~50 messages, no truncation = 10,000-30,000 tokens of history
- nanoClaw: sends 4-15 messages, truncated = 1,500-4,000 tokens of history

### Strategy 3: Tool Output Compression

```python
# In agent.py, after tool execution
def compress_tool_output(self, tool_name: str, raw_output: str) -> str:
    """
    Truncate tool outputs aggressively. The LLM doesn't need 4000 chars
    of web page text if 1500 chars covers the relevant content.
    
    Limits per tool:
    - web_search: 2000 chars (5 results with snippets)
    - web_fetch: 4000 chars (article content)
    - shell_exec: 2000 chars (command output)
    - file_read: 4000 chars (file content)
    - file_list: 1000 chars (directory listing)
    - memory_search: 1000 chars (memory facts)
    - skills: 1000 chars (weather, github info)
    """
    limits = {
        'web_search': 2000, 'web_fetch': 4000, 'shell_exec': 2000,
        'file_read': 4000, 'file_list': 1000, 'memory_search': 1000,
    }
    limit = limits.get(tool_name, 1500)  # default 1500 for skills
    
    if len(raw_output) > limit:
        return raw_output[:limit] + f"\n...[truncated at {limit} chars]"
    return raw_output
```

### Strategy 4: Compact System Prompt

```
Target: system prompt under 400 tokens.

Current spec prompt: ~300 tokens (good).

Rules:
- No redundant phrases. "Be concise" is 2 tokens. Don't spend 20 tokens saying the same thing.
- Security rules use numbered list, not prose.
- No tool descriptions in system prompt (tools have their own schemas).
- Memory facts are bullet points, not sentences.
```

### Strategy 5: Skip Unnecessary LLM Calls

```python
# In agent.py
async def run(self, user_message: str, ...):
    # Skip memory extraction for trivial messages (saves 1 full LLM call)
    skip_memory = (
        len(user_message) < 20
        or user_message.lower().strip() in [
            'thanks', 'thank you', 'ok', 'okay', 'got it', 'cool',
            'yes', 'no', 'sure', 'hi', 'hello', 'hey', 'bye'
        ]
        or not any(t in user_message.lower() for t in self.memory_triggers)
    )
    
    # ... after getting response ...
    if not skip_memory:
        asyncio.create_task(self._extract_memories(user_message, final_response))
```

**This alone saves ~500-1000 tokens on 60-70% of messages** (greetings, short replies, follow-ups).

### Strategy 6: Token Tracking and Reporting

```python
# SessionBudget tracks cumulative cost
class SessionBudget:
    async def get_cost_estimate(self, session) -> dict:
        """Estimate cost based on model pricing."""
        # OpenRouter provides pricing per model
        return {
            "tokens_used": session.total_tokens,
            "estimated_cost_usd": session.total_tokens * self.cost_per_token,
            "budget_remaining_pct": (1 - session.total_tokens / self.max_tokens) * 100
        }

# Dashboard shows daily/weekly cost estimates
# CLI: nanoclaw status shows token usage
```

### Token Budget Summary

| Component | OpenClaw (est.) | nanoClaw | Savings |
|-----------|----------------|----------|---------|
| System prompt | ~800 tokens | ~300 tokens | 62% |
| Tool schemas | ~5000 tokens (100+ tools) | ~800 tokens (5-7 tools) | 84% |
| History | ~15000 tokens (50 msgs) | ~3000 tokens (4-15 msgs) | 80% |
| Tool outputs | ~8000 tokens (no truncation) | ~2000 tokens (compressed) | 75% |
| Memory extraction | every message | 30-40% of messages | 60-70% |
| **Total per 3-iteration conversation** | **~30000 tokens** | **~6000-8000 tokens** | **73-80%** |
| **Estimated cost (Claude Sonnet)** | **~$0.05-0.10** | **~$0.01-0.02** | **75-80%** |
| **Daily cost (active user, 50 msgs)** | **$3-5** | **$0.50-1.00** | **75-80%** |

## Agent Execution Patterns

nanoClaw uses two execution patterns, chosen automatically based on task complexity.

### Pattern 1: ReAct (default, ALL messages)

```
User message -> LLM (with tools) -> tool_calls[] -> parallel execute -> LLM -> response
```

Standard Reason-Act-Observe loop. The LLM sees the user message, decides which tools to call, we execute them (in parallel if multiple), feed results back, LLM responds.

Key optimization: modern LLMs (Claude, GPT-4o) can return **multiple tool_calls in a single response**. We execute them all in parallel with `asyncio.gather`. This means "search the web AND check my files" is one iteration, not two.

For complex tasks, the LLM naturally plans before acting. Claude and GPT-4o already think step-by-step on hard problems without being told. We don't need a separate planning step.

### Pattern 2: Automatic Escalation (when ReAct struggles)

If ReAct hits 4 iterations without a final answer, we inject a nudge:

```
"You've been working on this for several steps. Pause and plan:
what remaining steps do you need? Which can run in parallel?
Execute efficiently."
```

This uses the NEXT iteration's LLM call (which would happen anyway), so it costs zero extra tokens. The LLM steps back, organizes remaining work, and finishes faster.

Escalation fires at most once per session.

### Why Not Other Patterns

- **Separate Plan-then-Execute**: Costs an extra LLM call (~300-500 tokens) on EVERY complex task. Our escalation achieves the same result at zero cost -- and only when actually needed.
- **ReWOO** (plan all tools upfront without intermediate observation): Too rigid. If step 2's result changes what step 3 should do, ReWOO can't adapt.
- **Tree of Thought**: Expensive (multiple parallel LLM calls exploring branches). Overkill for a personal assistant.
- **Reflexion** (self-critique loop): Adds an extra LLM call per iteration for self-evaluation. Token cost too high for the benefit.
- **LATS** (Language Agent Tree Search): Research-grade, way too complex and expensive.

ReAct with parallel tools + automatic escalation covers 99% of personal assistant use cases.

### How the Agent Picks a Pattern

**Not keyword matching.** Keyword heuristics are brittle: "compare two numbers" triggers plan mode for no reason, "find me the best laptop" doesn't trigger it but should.

**Not an LLM classifier call.** That costs 300-500 tokens on EVERY message just to decide how to work. Defeats the purpose.

**The actual approach: ReAct-first with automatic escalation.**

```
Always start with ReAct.
If the LLM naturally returns a plan in text -> follow it (free planning).
If ReAct hits 4 iterations without finishing -> pause, switch to plan mode.
```

This means:
- Simple tasks (90%+): ReAct, done in 1-2 iterations. Zero overhead.
- Medium tasks: LLM calls 2-3 tools across iterations. ReAct handles it fine.
- Complex tasks: ReAct starts, burns a few iterations, we detect it's struggling, pause and ask the LLM to make an explicit plan for the remaining work.

The LLM already knows how to plan. Claude and GPT-4o will naturally think before acting on hard tasks -- they'll output reasoning text alongside tool calls. We don't need a separate planning step for that. We only intervene when ReAct is clearly struggling (iteration count climbing without a final answer).

### Escalation Logic

```python
# In _run_react, inside the iteration loop:

# If we've used 4+ iterations and still calling tools, escalate to plan mode
if iteration >= 4 and llm_response.tool_calls and not llm_response.content:
    logger.info(f"Escalating to plan mode after {iteration} iterations")
    
    # Inject a planning nudge into the conversation
    messages.append({
        "role": "user",
        "content": (
            "You've been working on this for several steps. "
            "Please pause and make a brief plan: "
            "what remaining steps do you need, and which can run in parallel? "
            "Then execute the plan efficiently."
        )
    })
    # Continue the loop -- LLM will now plan before acting
    # This costs 0 extra LLM calls (it's the next iteration anyway)
    continue
```

This is elegant because:
- **Zero overhead on simple tasks.** No planning call, no keyword matching.
- **Free planning on medium tasks.** The LLM plans naturally in its reasoning.
- **Automatic rescue on complex tasks.** After 4 iterations, we nudge the LLM to step back and plan. This uses the next iteration's LLM call (which would happen anyway), so it costs nothing extra.
- **The nudge is a user message, not a system hack.** The LLM treats it naturally.

### What About Explicit Plan Requests?

If the user literally says "make a plan first" or "step by step", we respect that:

```python
def _user_wants_plan(self, message: str) -> bool:
    """Only trigger on EXPLICIT plan requests from the user."""
    explicit = ['make a plan', 'plan first', 'step by step', 'create a plan']
    return any(p in message.lower() for p in explicit)
```

This is a narrow trigger -- only when the user explicitly asks for a plan. Everything else starts with ReAct.

### Summary

| Scenario | What happens | Extra cost |
|----------|-------------|------------|
| "What's the weather?" | ReAct, 1 tool, done | None |
| "Search for X and save to file" | ReAct, 2 tools parallel, done | None |
| "Research competitors and write report" | ReAct starts, LLM naturally plans, 3-4 iterations | None |
| "Analyze 10 companies" (complex) | ReAct runs 4 iterations, escalation nudge, LLM plans remaining work | None (nudge uses existing iteration) |
| "Make a plan for my project" | User explicitly requested plan, plan-first mode | 1 extra LLM call |

## Runtime Efficiency

Token efficiency is about cost. Runtime efficiency is about speed. Both matter equally.

### Parallel Tool Execution

When the LLM returns multiple tool_calls in one response, execute them in parallel with `asyncio.gather`, not sequentially.

```python
# BAD -- sequential, each tool waits for previous
for tool_call in llm_response.tool_calls:
    result = await execute_tool(tool_call)  # 2s each = 6s for 3 tools

# GOOD -- parallel, all tools run at once
results = await asyncio.gather(*[
    execute_tool(tc) for tc in llm_response.tool_calls
], return_exceptions=True)  # 2s total for 3 tools
```

Example: user says "check weather in Tbilisi, get latest AI news, and show my GitHub repos". LLM returns 3 tool calls. Sequential: 6s. Parallel: 2s.

Shell commands needing confirmation still block on user input. `gather` handles this naturally -- other tools run while one waits.

### Session Cache

Cache expensive operations within a session. Same URL fetched twice? Return cached.

```python
class SessionCache:
    """In-memory cache per session. Cleared when session ends."""
    
    def __init__(self, ttl_seconds: int = 300):
        self._cache = {}  # key -> (result, timestamp)
        self.ttl = ttl_seconds
    
    def get(self, key: str) -> str | None:
        if key in self._cache:
            result, ts = self._cache[key]
            if time.time() - ts < self.ttl:
                return result
            del self._cache[key]
        return None
    
    def set(self, key: str, value: str):
        self._cache[key] = (value, time.time())
```

What to cache: web_fetch (by URL), web_search (by query), memory_search (by query), file_read (by path, invalidate on write).
What NEVER to cache: shell_exec (side effects), file_write, memory_save.

### Connection Pooling

One shared aiohttp session for the entire application lifecycle.

```python
class ConnectionPool:
    _session: aiohttp.ClientSession | None = None
    
    @classmethod
    async def get_session(cls) -> aiohttp.ClientSession:
        if cls._session is None or cls._session.closed:
            connector = aiohttp.TCPConnector(
                limit=20,            # max concurrent connections
                limit_per_host=5,    # respect per-host rate limits
                ttl_dns_cache=300,   # DNS cache 5 min
                keepalive_timeout=30,
            )
            cls._session = aiohttp.ClientSession(
                connector=connector,
                timeout=aiohttp.ClientTimeout(total=30),
            )
        return cls._session
    
    @classmethod
    async def close(cls):
        if cls._session and not cls._session.closed:
            await cls._session.close()
```

All HTTP consumers (LLM API, web_search, web_fetch, skills) use this single pool. Close on shutdown.

### Async Database

SQLite is synchronous. Don't block the event loop.

```python
# Wrap all DB calls with asyncio.to_thread
async def get_history(self, session_id, limit=15):
    return await asyncio.to_thread(self._get_history_sync, session_id, limit)

# Enable WAL mode for concurrent reads during writes:
conn.execute("PRAGMA journal_mode=WAL")
conn.execute("PRAGMA synchronous=NORMAL")
```

Start with `asyncio.to_thread` (no extra dependency). Switch to `aiosqlite` only if profiling shows DB is a bottleneck.

### Startup Speed

Target: accepting Telegram messages within 2 seconds of `nanoclaw serve`.

- Lazy imports: chromadb, croniter imported inside functions, not at module level
- Pre-compile all regex patterns at module level (compile once, match many)
- SQLite WAL mode (set on first connection)
- Don't validate config on every import -- only on `serve` and `init`

### Efficient Memory Search

```python
def _fts_query(self, query: str) -> str:
    """Convert natural language to FTS5 query.
    'weather preferences' -> 'weather OR preferences'"""
    words = query.split()
    return " OR ".join(w for w in words if len(w) > 2)
```

Use FTS5 rank function. Only return memories that actually match -- don't pad with random facts.

### Iteration Discipline

```
1-2 iterations: simple Q&A, single tool -- ideal
3-4 iterations: research with multiple tools -- normal
5+ iterations: log warning, agent may be struggling
10+ iterations: force stop, return partial results
```

### Response Time Targets

| Scenario | Target | How |
|----------|--------|-----|
| Simple question, no tools | <2s | 1 LLM call, compact context |
| Single tool call | <4s | LLM + tool + LLM |
| Multi-tool parallel (3 tools) | <5s | LLM + parallel tools + LLM |
| Research (5+ iterations) | <15s | Parallel + cached repeats |
| Background task (spawn) | <2s ack | Immediate response, async work |

---
> Source: [grishahq/nanoClaw](https://github.com/grishahq/nanoClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
