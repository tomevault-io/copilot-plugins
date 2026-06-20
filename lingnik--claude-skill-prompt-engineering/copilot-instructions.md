## claude-skill-prompt-engineering

> Help users craft effective prompts for Claude models (Opus 4.6, Sonnet 4.6, Haiku 4.5) across API and agent harness contexts. Trigger when the user asks to write, improve, or debug a prompt, choose a Claude model, configure thinking or effort, migrate from an older model, set up CLAUDE.md, write task instructions for Claude Code or Cowork, decompose a task for an agent, or write a repeatable process prompt. Also trigger for natural phrasing like "getting bad results from Claude," "Claude keeps doing X wrong," "how do I get Claude to...," "why does Claude keep...," or "tune Claude's behavior" — any sign the user wants to improve how Claude responds. Trigger generously.


# Claude Prompt Engineering Skill

Help users and agents craft effective prompts for Claude models (Opus 4.6, Sonnet 4.6, Haiku 4.5) across all deployment contexts — from direct API usage to conversational agent harnesses like Claude Code, Cowork, and Chat. Guidance is grounded in system card analysis and official documentation.

## Trigger & Scope

Identify the user's context to determine which guidance applies. This skill covers Opus 4.6, Sonnet 4.6, and Haiku 4.5 across API, agent harness, and agentic system-building contexts. It does not cover older models except in the context of migration.

## Workflow

Follow these steps when helping with prompt engineering. Each step includes a skip condition — check it before proceeding.

These steps are a framework, not a rigid procedure. Adapt to the user's specific situation — skip steps that don't apply, combine steps when natural, and reason from the Key Principles when encountering scenarios these instructions don't cover explicitly.

### Step 0: Identify the Prompting Context

Determine which context the user is working in. This changes which advice and reference files apply. Heuristic: if the user mentions Claude Code, Cowork, or Chat, or is writing a user-turn message rather than a system prompt, they are in `agent_harness` context. If they are writing system prompts and tool definitions, they are in `api` context. If they are designing an agent product for end users, they are in `building_harness` context.

<routing>
  <context name="api">
    User controls: system prompt, tools, model, thinking config, all parameters.
    Primary guidance: this file + all reference files.
  </context>
  <context name="agent_harness">
    User controls: user-turn messages, CLAUDE.md, plan mode, session model choice.
    Claude Code also: system prompt customization, tool restrictions, effort/thinking control, compaction.
    Cowork/Chat: base system prompt, tool definitions, and thinking config are fixed.
    Primary guidance: references/agent-harness-prompting.md
  </context>
  <context name="building_harness">
    User controls: system prompt, tools, model — designing for end users.
    Primary guidance: references/agentic-prompting.md
  </context>
</routing>

If the user is in an **agent harness context**, prioritize guidance from `references/agent-harness-prompting.md`. The API-focused steps below (system prompt design, API parameter configuration) still apply when the user is building their own system or using the API directly.

This skill provides guidance and drafts. Do not modify the user's files (system prompts, CLAUDE.md, code) without explicit permission — present recommendations and let the user decide what to apply.

### Step 1: Understand the Goal

Before writing or modifying a prompt, clarify:

- **What is the task?** Classification, extraction, generation, analysis, coding, agentic workflow, conversation?
- **What is the input?** Short text, long documents, images, code, structured data?
- **What is the expected output?** Free text, JSON, code, tool calls, decisions?
- **What are the constraints?** Latency, cost, accuracy requirements, safety sensitivity?
- **What's the deployment context?** Single API call, multi-turn conversation, agent loop, batch processing, or agent harness (Claude Code/Cowork/Chat)?

### Step 2: Select the Model

Skip if: the user already specified a model, or is in a harness where model choice is fixed for the session.

Use the model selection guidance from `references/model-profiles/model-comparison.md` and the individual model profiles. The short version:

| If the task is... | Use | Why |
|-------------------|-----|-----|
| Complex multi-step coding or deep investigation | **Opus 4.6** | Highest capability ceiling; 128K output tokens |
| Production workload needing quality + efficiency | **Sonnet 4.6** | Best quality/cost ratio; SOTA on web agents and finance |
| High-volume, moderate-complexity, or parallel agents | **Haiku 4.5** | 5× cheaper than Opus output; fastest latency |
| Sensitive topics needing low refusals + strong capability | **Sonnet 4.6** | 0.41% over-refusal with best safety (99.40% on hard violative). Haiku is even lower (0.02%) but less capable. |
| Agentic deployment where injection is a concern | **Sonnet 4.6 + thinking + safeguards** | 0% injection success on coding benchmarks |

If unsure, start with Sonnet 4.6 at medium effort. It matches Opus on most benchmarks at 60% of the cost.

### Step 3: Draft the Prompt

Skip if: the user is in an agent harness context — they write user-turn messages, not structured prompts. Consult `references/agent-harness-prompting.md` instead.

Build the prompt using this structure:

1. **System prompt** — Role, behavioral constraints, output format rules
2. **Context/documents** — Long content goes first, above the query
3. **Examples** — 3–5 diverse examples in `<example>` tags if format/tone matters
4. **Instructions** — Clear, specific task description
5. **Query/input** — The actual user input, at the end

Consult `references/prompting-techniques.md` for the full technique catalog with before/after examples. For agentic workflows, consult `references/agentic-prompting.md` for orchestration patterns, delegation design, and state management.

### Step 4: Apply Model-Specific Techniques

Skip if: the user's question is model-agnostic, or they're asking about general prompting principles.

**For Opus 4.6:**
- Add explicit scope constraints to prevent over-exploration ("Only investigate files in the /src directory")
- Require user confirmation before irreversible actions
- Dial back any anti-laziness prompts inherited from older models
- Use targeted tool guidance ("Use search when it would enhance understanding") not blanket defaults
- Be aware that extended thinking *increases* prompt injection vulnerability on Opus (ART benchmark: 21.7% with thinking vs. 14.8% without)

**For Sonnet 4.6:**
- Set effort level explicitly (defaults to `high`, which may be more than needed)
- For coding agents, use adaptive thinking — it enables interleaved reasoning between tool calls
- Add anti-test-gaming instructions if the task involves TDD
- Be aware of GUI alignment inconsistency: Sonnet's alignment in computer-use/GUI mode is "noticeably more erratic" than in text/tool-use mode. Test thoroughly if deploying for computer use.

**For Haiku 4.5:**
- Provide explicit quality expectations for code generation (it tends to hardcode for tests at 6× the rate of Sonnet)
- Add system prompt guidance for sensitive scientific topics (it assumes academic intent)
- Remember: no adaptive thinking — effort levels are not available; thinking is a binary toggle

### Step 5: Configure API Parameters

Skip if: the user is in an agent harness — they don't control API parameters directly. (Claude Code users can adjust effort via `/model` and thinking via `Alt+T`, but not API-level config.)

Consult `references/api-configuration.md` for the full parameter reference. Key decisions:

**Thinking mode:**
- Opus 4.6 → `thinking: {"type": "adaptive"}` (manual mode is deprecated)
- Sonnet 4.6 → `thinking: {"type": "adaptive"}` for agents, or `{"type": "enabled", "budget_tokens": 16384}` for predictable costs
- Haiku 4.5 → `thinking: {"type": "enabled", "budget_tokens": N}` or omit for speed

**Effort level:**
- Default to `medium` for Sonnet production use
- Use `high` for complex reasoning or agentic tasks
- Use `low` for classification, chat, subagents, and latency-sensitive work
- `max` is Opus-only — use for the hardest problems

**Max tokens:**
- Set generously. 64K for Sonnet/Haiku, up to 128K for Opus. Truncation wastes the entire request.

### Step 6: Review and Iterate

Before finalizing, check the prompt against these criteria:

- [ ] Would a colleague with no context understand what to do from this prompt alone?
- [ ] Are instructions positive ("write in prose") not negative ("don't use bullets")?
- [ ] Is the system prompt using normal language, not emphatic caps/exclamation?
- [ ] Does the document ordering put long content before the query?
- [ ] Are examples diverse enough to avoid pattern lock-in?
- [ ] Are there explicit constraints on agentic scope (if applicable)?
- [ ] Is the effort level appropriate for the task complexity?
- [ ] For agentic prompts: are confirmation requirements specified for irreversible actions?

## Key Principles

These are the most important prompting principles, distilled from system card analysis and official documentation:

**1. Be explicit about what you want, not what you don't want.** Positive instructions ("write in flowing prose") are more reliable than negative ones ("don't use bullet points"). This applies to formatting, behavior, and tool use.

**2. Explain the why.** Claude generalizes from motivation. "Never use ellipses because the TTS engine can't pronounce them" works better than "NEVER use ellipses" because Claude extends the reasoning to similar cases.

**3. Right-size the model AND the effort.** The cheapest token is the one you don't generate. Most production workloads should use Sonnet at medium effort, not Opus at max. Test on your actual distribution before assuming you need the biggest model.

**4. Adaptive thinking is the new default.** For Opus and Sonnet 4.6, use `thinking: {"type": "adaptive"}` with the effort parameter instead of manually setting `budget_tokens`. Adaptive thinking handles bimodal workloads (mix of easy and hard queries) significantly more efficiently.

**5. Dial back intensity for 4.6 models.** Anti-laziness prompts, emphatic instructions ("CRITICAL: ALWAYS use this tool"), and aggressive tool encouragement from older-model prompts will cause overtriggering. Use conversational language.

**6. Constrain agentic scope explicitly.** Opus 4.6 over-explores and takes irreversible actions without asking. Sonnet 4.6 is better but still occasionally over-investigates. Always add explicit boundaries for what the model should and shouldn't do autonomously. Consult `references/agentic-prompting.md` for the confirmation spectrum pattern.

**7. Test safety behavior in your target language.** Over-refusal rates vary significantly across languages (0.21% English vs. 1.09% Arabic on Opus). If your application serves non-English users, test refusal behavior specifically for those languages.

**Additional principles for agent harness contexts** (Claude Code, Cowork, Chat):

**8. Orient before instructing.** The agent's effectiveness depends on knowing where to look. Name the relevant files, describe the repo layout if non-obvious, and use CLAUDE.md for context that would otherwise be repeated. The best task instruction lets the agent start working immediately rather than spending turns exploring.

**9. Decompose deliberately.** The user is the orchestrator. Separate discovery from implementation with a review gate for ambiguous tasks. Give the full task at once only when the agent needs holistic context. For large projects, use plan mode or explicitly ask the agent to propose an approach before coding.

The anti-patterns tables below catalog specific instances of these principles. Consult them as a quick reference, but reason from the principles above when encountering novel situations.

## Reference Files

| File | Consult when... |
|------|-----------------|
| `references/agent-harness-prompting.md` | User works in Claude Code, Cowork, or Chat; writes task instructions; configures CLAUDE.md; decomposes work for agents |
| `references/common-patterns.md` | User needs a prompt template for a specific use case (classification, extraction, generation, analysis, research, frontend, agentic, conversation) |
| `references/prompting-techniques.md` | Writing or improving a prompt; need technique guidance, before/after examples, or frontend design prompting |
| `references/agentic-prompting.md` | Building agentic systems with the API; multi-agent architectures; sub-agent prompts; research agents; long-running autonomous tasks |
| `references/api-configuration.md` | Configuring API parameters; optimizing cost/latency; choosing thinking mode; migrating from Sonnet 4.5 |
| `references/model-profiles/model-comparison.md` | Choosing between models; understanding cross-model behavioral differences |
| `references/model-profiles/opus-4-6.md` | Deciding whether to use Opus; debugging Opus-specific behavior (over-exploration, injection anomaly) |
| `references/model-profiles/sonnet-4-6.md` | Deciding whether to use Sonnet; optimizing Sonnet prompts (best safety alignment, coding verification) |
| `references/model-profiles/haiku-4-5.md` | Deciding whether Haiku is sufficient; debugging Haiku-specific behavior (test-hardcoding, safeguard dependency) |

## Anti-Patterns

These are concrete applications of the Key Principles above — not an exhaustive rulebook. Use them as quick-reference for common mistakes.

<anti_patterns category="prompt">
  <pattern mistake="Use emphatic caps: YOU MUST ALWAYS..." correct="Use normal language: Use this tool when..." reason="4.6 models overtrigger on intense instructions" />
  <pattern mistake="Negative formatting: Don't use bullets" correct="Positive formatting: Write in flowing prose" reason="Negative instructions are less reliable" />
  <pattern mistake="Prefill the assistant response" correct="Use system prompt instructions or structured outputs" reason="Deprecated on 4.6 models" />
  <pattern mistake="Anti-laziness prompts from older models" correct="Remove or replace with targeted guidance" reason="Causes over-exploration and unnecessary tool calls on 4.6" />
  <pattern mistake="Prescriptive step-by-step reasoning plans" correct="Say 'reason through this carefully' and let thinking do the work" reason="Claude's reasoning often exceeds hand-written plans" />
  <pattern mistake="Blanket tool encouragement: If in doubt, use search" correct="Use search when it would enhance your understanding" reason="Tools that undertriggered before now overtrigger on 4.6" />
  <pattern mistake="Vague generation requests: Create a dashboard" correct="Add ambition modifiers: Create a dashboard with as many relevant features as possible" reason="4.6 models respond well to explicit quality framing; vague prompts get baseline output" />
  <pattern mistake="Default frontend aesthetics (Inter, purple gradients)" correct="Add frontend_aesthetics system prompt block steering toward distinctive design" reason="Without guidance, models converge on generic AI slop patterns" />
</anti_patterns>

<anti_patterns category="configuration">
  <pattern mistake="Default to Opus for everything" correct="Start with Sonnet medium effort; upgrade only if needed" reason="5x Haiku cost; Sonnet matches Opus on many benchmarks" />
  <pattern mistake="Use max effort on non-Opus models" correct="Use high as the ceiling for Sonnet and Haiku" reason="max is Opus 4.6-only (returns error on Sonnet/Haiku)" />
  <pattern mistake="Set small max_tokens with high effort" correct="Set 64K for Sonnet/Haiku, 128K for Opus at high effort" reason="Claude runs out of space and truncates" />
  <pattern mistake="Extended thinking on Opus for injection-sensitive agents" correct="Use Sonnet 4.6 + adaptive thinking + safeguards" reason="Thinking increases injection success on Opus (ART benchmark)" />
  <pattern mistake="Switch thinking modes mid-conversation" correct="Stick to one thinking mode per conversation" reason="Breaks prompt cache breakpoints for messages" />
  <pattern mistake="Use budget_tokens on Opus 4.6" correct="Use adaptive thinking with effort parameter" reason="Manual thinking is deprecated on Opus 4.6 and will be removed in a future release" />
  <pattern mistake="Migrate Sonnet 4.5→4.6 without setting effort" correct="Explicitly set effort (medium for most, low for latency-sensitive)" reason="Sonnet 4.6 defaults to high effort, causing unexpected latency increase" />
</anti_patterns>

<anti_patterns category="behavioral">
  <pattern mistake="Deploy Opus agents without confirmation requirements" correct="Require confirmation for destructive/visible actions" reason="Opus takes irreversible actions (force-push, file deletion)" />
  <pattern mistake="Assume Haiku can handle hard reasoning" correct="Use Haiku for moderate tasks; escalate hard ones to Sonnet" reason="36.6% on SWE-bench-hard; significant accuracy gaps" />
  <pattern mistake="Role-play with 'don't break character'" correct="Use role assignment without character-lock or deception instructions" reason="Can override honesty — model may deny being AI" />
  <pattern mistake="Trust Haiku in adversarial evaluation scenarios" correct="Use blind evaluation protocols; be aware of Goodhart effects" reason="9% evaluation awareness rate; behavior changes when it suspects testing" />
  <pattern mistake="Assume uniform refusal behavior across languages" correct="Test in each target language separately" reason="Over-refusal varies 3-5x between languages" />
  <pattern mistake="Deploy Sonnet for GUI computer use without extra testing" correct="Test computer-use workflows thoroughly; add explicit safety constraints" reason="GUI alignment is more erratic than text mode" />
  <pattern mistake="Let agentic coding create files without cleanup" correct="Add cleanup instruction for temporary files, or constrain to committing only final artifacts" reason="Claude uses scripts as temporary scratchpads, leaving clutter" />
</anti_patterns>

<anti_patterns category="agent_harness">
For the full list of platform-specific anti-patterns (Claude Code, Cowork, Chat, agent-to-subagent), consult references/agent-harness-prompting.md. The most critical:
  <pattern mistake="Combine discovery and implementation without a gate" correct="Separate: investigate first, report findings, then implement" reason="Agent implements based on first (possibly wrong) understanding" />
  <pattern mistake="Assume the agent knows the repo layout" correct="Name relevant files/directories; use CLAUDE.md for persistent context" reason="Agent wastes time exploring or guesses wrong" />
  <pattern mistake="Give scope-unbounded instructions to Opus" correct="Add explicit scope: which files, which directories, when to stop" reason="Over-explores, reads unnecessary files, spawns sub-agents" />
  <pattern mistake="Repeat repo context every session" correct="Put stable context in CLAUDE.md" reason="Wastes time and tokens; context may drift" />
  <pattern mistake="Use API jargon the harness controls" correct="Use the levers available to your platform (e.g., Claude Code: effort via /model, Alt+T for thinking, CLAUDE.md, plan mode)" reason="Users can't set budget_tokens or thinking config directly in most harnesses" />
</anti_patterns>

---
> Source: [Lingnik/claude-skill-prompt-engineering](https://github.com/Lingnik/claude-skill-prompt-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
