## building-agent-systems

> Use when designing or building AI agent systems, choosing between workflow patterns, diagnosing poor agent decisions, designing tools for agents, setting up evals, or managing context rot in long-running agents.


# Agent System Architect

## When This Skill Activates

This skill applies when you encounter ANY of these signals:

- Building or modifying an agent that makes LLM API calls
- Designing tool schemas or MCP servers for agent consumption
- Setting up agent evaluation (evals)
- Debugging agent behavior (wrong tool calls, context issues, poor decisions)
- Choosing between single agent vs multi-agent vs workflow
- Managing long-running agent sessions (context rot, state persistence)
- Implementing RAG for knowledge-heavy agents

**Core principle:** Start with the simplest solution that meets the need. Complexity is a cost, not a feature. Five disciplines compound each other: architecture selection, tool design, context engineering, think tool, and evals. Get one wrong and the others cannot compensate.

---

## First: Assess the Situation

Before recommending anything, answer these questions by examining the codebase:

1. **Lifecycle stage:** Are we starting from scratch (0-to-1), adding features (growing), or debugging (struggling)?
2. **Architecture:** Single agent? Multi-agent? Workflow? What pattern is currently in use?
3. **Tool inventory:** How many tools? How are they designed? What's the token budget for tool definitions?
4. **Context health:** How is context managed? Any signs of context rot (degraded quality over conversation length)?
5. **Eval status:** Do evals exist? What do they cover? Are they saturated?

Based on this assessment, follow the appropriate section below.

---

## Architecture Decision Engine

Think through this step by step. Do not prescribe an architecture without reasoning through each step.

### Step 1: Complexity Assessment

```
If < 5 subtasks AND predictable outcomes --> Lean toward WORKFLOW
If > 5 subtasks OR unpredictable outcomes --> Lean toward AGENT
```

### Step 2: Parallelism Check

```
If subtasks run independently --> Multi-agent (Orchestrator-Workers)
If subtasks are sequential    --> Single agent or Prompt Chain
```

### Step 3: Quality Requirements

```
If output needs iterative refinement AND clear eval criteria --> Evaluator-Optimizer
If output needs refinement BUT no clear criteria            --> Human-in-the-loop
If no refinement needed                                     --> Simpler pattern
```

### Step 4: Input Diversity

```
If different inputs need different handlers AND > 3 categories --> Router
If < 3 categories --> Handle in system prompt with conditional logic
```

### Step 5: Confirm Architecture

| Pattern | When to Use | Complexity | Token Cost |
|---------|------------|:----------:|:----------:|
| Augmented LLM | Single task, single call, predictable | Lowest | $ |
| Prompt Chain | Sequential steps with validation gates | Low | $$ |
| Router | Multiple input types need different handlers | Low | $$ |
| Parallelization | Independent chunks or voting for reliability | Medium | $$ |
| Orchestrator-Workers | Dynamic decomposition, unknown subtask count | High | $$$ |
| Evaluator-Optimizer | Iterative refinement, subjective quality | High | $$$ |

### Real-World Examples

| Use Case | Wrong Choice | Right Choice | Why |
|----------|-------------|--------------|-----|
| Email triage | Autonomous agent | Router | 3 categories, deterministic |
| Code migration | Prompt chain | Orchestrator-Workers | File count unknown, parallelizable |
| Blog post writing | Single LLM call | Evaluator-Optimizer | Quality is subjective, benefits from iteration |
| Customer support | Complex router | Prompt chain + think tool | Sequential policy checks, not routing |
| Data pipeline QA | Autonomous agent | Prompt chain | Steps are fixed and well-defined |

### Performance Characteristics

| Pattern | Latency | Token Cost | Reliability | Debuggability |
|---------|:-------:|:----------:|:-----------:|:-------------:|
| Augmented LLM | Lowest | $ | Highest | Trivial |
| Prompt Chain | Low | $$ | High | Easy (step-by-step) |
| Router | Low | $$ | High | Easy (check classification) |
| Parallel Workers | Low (parallel) | $$ | High | Medium |
| Evaluator-Optimizer | Medium-High | $$$ | Medium | Medium |
| Orchestrator-Workers | High | $$$-$$$$ | Medium | Hard |
| Autonomous Agent | Unpredictable | $$$$ | Lowest | Hardest |

**CRITICAL:** Prefer workflows over autonomous agents for well-defined tasks. Workflows are deterministic and debuggable. Agents are flexible but unpredictable.

See `references/patterns-reference.md` for full implementation with case studies.

---

## Tool Design Clinic

Tools dominate agent context. Poor tools destroy good architectures. When designing or reviewing tools, run this diagnostic.

### Diagnostic 1: Token Budget

Count total tokens across all tool definitions. If > 10K tokens, implement Tool Search Tool (lazy-load). Result: ~85% context reduction.

### Diagnostic 2: Granularity Check

For each tool, ask: "Does this require the agent to make multiple calls to achieve one logical action?"

```
BAD:  list_users() -> get_user_calendar() -> check_availability() -> create_event()
GOOD: schedule_meeting(participants, time, title)
```

Design for agent affordances, not human affordances. Merge granular APIs into agent-appropriate tools.

### Diagnostic 3: Error Quality

Does the error response tell the agent what to do next?

```python
# BAD: Agent has no idea what to do
{"error": "not found"}

# GOOD: Agent knows exactly how to recover
{"error": "User not found", "suggestion": "Try 'john.smith' - closest match"}
```

### Diagnostic 4: Return Value Semantics

```
BAD:  {"id": "a1b2c3", "type": "evt"}
GOOD: {"id": "q3_planning_meeting", "type": "calendar_event", "title": "Q3 Planning Meeting"}
```

### Diagnostic 5: Response Size Control

Does each tool support filtering and pagination? Claude Code enforces ~25K token max on tool responses.

### Diagnostic 6: Description Quality

Each tool description should follow:

```
{tool_name}: {what it does, for what purpose}
Use when: {specific triggering conditions}
Do NOT use when: {anti-patterns}
```

**Impact:** Refining tool descriptions alone (without changing code) took Claude Sonnet to SWE-Bench SOTA.

### MVT Checklist

```
[ ] Tool does ONE thing well
[ ] Description explains WHEN to use it (not just what)
[ ] 1-3 required parameters, optional params have sensible defaults
[ ] Error messages include a suggested recovery action
[ ] Returns structured data the agent can reason about
[ ] Has 1-3 usage examples in the description
```

### Tool Optimization Quick Reference

| Problem | Solution | Gain |
|---------|----------|------|
| Definitions > 10K tokens | Tool Search Tool (lazy-load) | 85% context reduction |
| Multi-step orchestration | Programmatic Tool Calling | 37% fewer tokens |
| Wrong tool usage | Tool Use Examples (1-5 per tool) | 72% to 90% accuracy |
| Ambiguous selection | Clearer `when to use` descriptions | Reduces mis-selection |

See `references/tool-design-reference.md` for complete templates and error recovery patterns.

**Tool dev cycle:** Prototype in Claude Code -> write eval specs -> feed failing transcripts back -> iterate.

---

## Context Engineering Playbook

Context engineering is managing the full context window lifecycle, not just the initial prompt. Even with 1M token windows, 900K tokens of irrelevant content degrades performance. **Quality always beats capacity.**

### The Context Stack

```
System Prompt        <- Most impactful, always present
Tool Definitions     <- Second most impactful, shapes behavior
Retrieved Context    <- Injected just-in-time (RAG, memory)
Conversation History <- Grows over time (context rot risk)
Current Task         <- User's immediate request
```

Each layer has a different half-life and impact. Optimize from top to bottom.

### For New Agents (0 to 1)

**System prompt at the "right altitude":**

```
Too vague (useless):  "You are a helpful assistant."
Too rigid (brittle):  "When user asks about invoice, run SELECT * FROM invoices WHERE..."
Right altitude:       "You are a billing specialist. Retrieve invoice data and explain
                       clearly. For date queries, default to last 30 days."
```

Include: role + decision criteria + tool guidance + constraints.
Exclude: what the model already knows, verbose examples for simple tasks.

**Token budget:** Simple chatbot 500-1K, tool-using 1-3K, complex multi-tool 2-5K. If more, use sub-agents.

### For Growing Agents (Adding Features)

1. Check context utilization. If past 50%, act now.
2. Implement **compaction** at 70% full (summarize and re-initialize).
3. Add **external memory** (progress file) before hitting the wall.
4. Isolate polluting tasks with **sub-agents** (research, file scanning).
5. Use **progressive disclosure** (lazy-load tools, not all upfront).

### For Struggling Agents (Debugging)

1. Read the full transcript, not just the final output. Scores lie.
2. Look for: wrong tool calls, repeated actions, context confusion.
3. Add think tool if the agent makes poor decisions at branching points.
4. Check for context rot: does quality degrade after 20+ turns?

### Good vs Bad Context Engineering

```
BAD:  Stuff the entire codebase into context "just in case"
GOOD: Give the agent a file-tree tool to explore what it needs

BAD:  Let conversation history grow without bounds
GOOD: Compact at 70% capacity, preserving key decisions

BAD:  Put all 50 tool definitions in the system prompt
GOOD: Give 5 core tools + a tool-search tool for the rest
```

See `references/context-engineering-reference.md` for implementation patterns.

---

## Think Tool

Add to any agent with > 3 sequential tool calls or policy-constrained decisions. +54% on complex policy tasks (Airline tau-bench).

**When to add:**
- Agent makes > 3 sequential tool calls
- Policy-constrained decisions (refunds, access control, moderation)
- Multi-step plans where order matters
- Agent needs to evaluate tradeoffs before acting

**When NOT needed:**
- Simple single-tool-call agents
- Routing/classification tasks
- Tasks with no ambiguity in action sequence

**Think Tool vs Extended Thinking:** Think tool is for mid-response reasoning between tool calls. Extended thinking is for complex upfront reasoning. Both can be used together.

See `references/think-tool-reference.md` for JSON schema, multi-language implementations, and performance data.

---

## Multi-Agent Architecture

Use when: single agent fills context before completing, tasks are parallelizable, specialist > generalist.

### Model Selection

| Role | Model | Why |
|------|-------|-----|
| Orchestrator | Claude Opus | Full planning capacity, complex delegation |
| Workers | Claude Sonnet | Cost-efficient, sufficient for focused tasks |
| RAG/context generation | Claude Haiku | Fast, cheap, mechanical tasks |

### The 8 Orchestration Principles

1. **Think like your agents** -- Build simulations to understand the agent experience
2. **Teach delegation explicitly** -- "Delegate research to workers, never do it yourself"
3. **Scale to complexity** -- 1-3 subagents for focused, 8-12 for multi-domain research
4. **Make worker instructions self-contained** -- No assumed shared context
5. **Let agents self-improve** -- Show Claude its failures and ask how prompts should change
6. **Start wide, narrow down** -- Broad initial queries, then progressively focus
7. **Parallel tool calling** -- 90% time reduction for complex queries
8. **Fail gracefully** -- If a worker fails, continue with remaining results

### Token Budget Reality

```
Chat interaction:       ~1x baseline cost
Single agent task:      ~4x baseline cost
Multi-agent system:     ~15x baseline cost
```

Budget accordingly. Use cheaper models for subagents.

---

## RAG for Knowledge-Heavy Agents

Naive RAG fails because chunks lose context. Use contextual retrieval: add 50-100 token context to each chunk before encoding, then hybrid search (embeddings + BM25) + reranking. Reduces failure rate 67%.

See `references/rag-reference.md` for full pipeline and implementation.

---

## Evaluation Strategy

Build evals before building the agent. No eval means no confidence. This is TDD for agents.

### Three Grader Types (Layer All Three)

1. **Code-based** -- Binary outcomes, unit tests, regex matching (cheap, fast)
2. **Model-based** -- Quality, nuance, rubric scoring (slower, more capable)
3. **Human** -- Ground truth, edge cases, blind spots (expensive, authoritative)

Human review catches ~20% of issues that automated evals miss.

### Quick Setup

```
1. Collect 20-50 real tasks (not synthetic)
2. Write unambiguous success criteria for each
3. Build balanced set: 30% easy / 50% medium / 20% hard
4. Implement code-based graders first
5. Add model-based graders for nuanced dimensions
6. Read transcripts regularly -- scores lie, transcripts reveal truth
7. Track scores over time; block releases on > 5% regression
```

### Eval Saturation

When an agent passes all solvable tasks, the eval is exhausted. Expand the test suite.

### Multi-Agent Eval Principle

Multi-agent systems take varied valid paths. **Grade outputs, not tool sequences.**

See `references/evals-reference.md` for agent-specific eval patterns, grader templates, and regression detection.

---

## Long-Running Agent Harness

For agents that work across multiple sessions.

### The Feature List Pattern

All features start as "failing". Agent cannot mark complete without passing tests. Agent cannot modify tests.

### Session Startup Protocol

```
1. Read progress file + git log (what was completed?)
2. Read feature list (which features still failing?)
3. Run basic tests (is the app working?)
4. Pick ONE failing feature
5. Implement + test rigorously
6. Git commit + update progress file
```

---

## Quick Reference: The Agent Loop

Every agent follows a three-phase loop:

```
GATHER CONTEXT --> TAKE ACTION --> VERIFY WORK
      ^                                |
      +--------------------------------+
              (repeat until done)
```

**Gather:** File search, semantic search, sub-agents, memory reads
**Act:** Tool calls, code generation, bash commands, MCP integrations
**Verify:** Rule-based checks, visual feedback, LLM-as-judge

---

## Debugging Agent Failures

When an agent is not performing well, check these in order. Most failures are caused by the first two items.

```
1. Tool problems?       -> Is the agent calling wrong tools?
                           Fix: Better descriptions, add examples
2. Context problems?    -> Is the agent ignoring recent information?
                           Fix: Compact context, add sub-agents
3. Reasoning problems?  -> Wrong decisions despite good info?
                           Fix: Add think tool, improve decision criteria
4. Architecture problems? -> Task shape doesn't fit the pattern?
                            Fix: Re-run Architecture Decision Engine
5. Self-improvement      -> Show Claude its failure transcripts
                           Ask: "Why did you fail? How should prompts change?"
```

---

## Common Mistakes

| Mistake | Why It Happens | Fix |
|---------|---------------|-----|
| Starting with autonomous agent | Agents seem powerful | Use decision tree -- workflows first |
| Vague tool descriptions | Copied from API docs | Rewrite: when to use + examples |
| No think tool for complex agents | Seems unnecessary | Add before shipping -- +54% on policy tasks |
| Single agent for parallelizable work | Simpler to implement | Switch to Orchestrator-Workers |
| Evals built after agent | "We'll add tests later" | Build evals first (TDD for agents) |
| Ignoring context rot | Works fine in short tests | Monitor perf vs conversation length |
| Too many tools loaded at once | "Agent might need them" | Tool Search Tool for lazy loading |
| Raw API responses as tool output | Quick to implement | Parse and return structured data |

## Quick Start: 0 -> 1 Agent

```
1. Define success criteria (what does "working" look like?)
2. Choose architecture (decision engine above -- start simple)
3. Define tools needed (MVT -- minimal viable tool)
4. Write 20 eval cases first (before building the agent)
5. Build tools (prototype -> eval -> iterate)
6. Add think tool if > 3 tool calls or policy decisions
7. Add multi-agent if single agent fills context
8. Sandbox file/network access
9. Ship to subset, iterate on regressions
```

## Superpowers Integration

- `superpowers:brainstorming` -> clarify requirements before architecture decisions
- `superpowers:writing-plans` -> turn architecture decisions into implementation tasks
- `superpowers:test-driven-development` -> build evals first, agent second
- `superpowers:dispatching-parallel-agents` -> implement multi-agent modules in parallel
- `superpowers:systematic-debugging` -> diagnose agent failures with transcript analysis

---
> Source: [PeterHiroshi/building-agent-systems](https://github.com/PeterHiroshi/building-agent-systems) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
