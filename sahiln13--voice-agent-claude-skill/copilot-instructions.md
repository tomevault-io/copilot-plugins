## voice-agent-claude-skill

> Optimize Vapi voice agent prompts through systematic evaluation and surgical improvements. Use when creating evals, analyzing call transcripts, optimizing prompts based on feedback, or making targeted prompt edits.


<essential_principles>

<principle_1>
**Evals Are Your Source of Truth**

Never optimize blindly. Every prompt change must be driven by eval data:
- Build eval suites BEFORE making changes
- Run evals AFTER every change
- Track pass rates across versions
- Regressions are unacceptable without explicit tradeoff approval
</principle_1>

<principle_2>
**Surgical Edits Over Rewrites**

Voice prompts are complex systems. Wholesale rewrites introduce regressions:
- Change ONE section at a time
- Preserve working patterns exactly
- Add specificity, don't remove constraints
- Test the specific behavior you're changing
</principle_2>

<principle_3>
**Voice Is Not Text**

Voice agents have unique constraints:
- 1-3 sentences per turn maximum
- Pronunciation guides for addresses, numbers, acronyms
- Pacing/pause instructions matter
- Tool execution timing is critical (speak THEN execute)
- Channel detection (voice vs SMS) changes behavior
</principle_3>

<principle_4>
**Prompt Structure Hierarchy**

Vapi prompts follow this structure (order matters for model attention):
1. **Dynamic Variables** - Time/date context
2. **Channel Detection** - Voice vs SMS branching
3. **Identity** - Role, personality, constraints
4. **Task** - Workflow, opening, core logic
5. **Context** - Business info, hours, FAQs
6. **Rules** - Boundaries, error handling
7. **Examples** - Input/output pairs by scenario
</principle_4>

<principle_5>
**Eval Types for Voice Agents**

Different behaviors need different eval strategies:
- **Factual accuracy**: Does response contain correct info?
- **Behavioral compliance**: Did agent follow workflow rules?
- **Tool usage**: Did agent call the right tool at the right time?
- **Fallback handling**: Did agent escalate correctly when unsure?
- **Tone/brevity**: Did response match voice constraints?
</principle_5>

</essential_principles>

<intake>
What would you like to do?

1. **Create evals** - Build an evaluation suite to test voice agent behavior
2. **Optimize from evals** - Analyze failing evals and improve the prompt systematically
3. **Surgical edit** - Make a targeted change to fix a specific behavior
4. **Audit prompt** - Review an existing prompt for anti-patterns and opportunities
5. **Something else** - Describe what you need

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "create evals", "build evals", "evaluation", "test suite" | `workflows/create-evals.md` |
| 2, "optimize", "failing", "improve", "from evals" | `workflows/optimize-from-evals.md` |
| 3, "surgical", "edit", "fix", "specific", "targeted" | `workflows/surgical-prompt-edit.md` |
| 4, "audit", "review", "check", "analyze" | `workflows/audit-prompt.md` |
| 5, other | Clarify intent, then route |

**After reading the workflow, follow it exactly.**
</routing>

<reference_index>
All domain knowledge in `references/`:

**Prompt Patterns:** prompt-structure.md, voice-constraints.md
**Evaluation:** eval-criteria.md, rubric-types.md
**Optimization:** common-failures.md, fix-patterns.md
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| create-evals.md | Build comprehensive eval suites for voice agent behavior |
| optimize-from-evals.md | Systematically improve prompts based on eval failures |
| surgical-prompt-edit.md | Make targeted, minimal changes to fix specific behaviors |
| audit-prompt.md | Review prompts for anti-patterns and improvement opportunities |
</workflows_index>

<quick_commands>
**Common patterns this skill handles:**

- "My agent is giving wrong hours" → surgical-edit workflow
- "Agent transfers when it shouldn't" → create evals → optimize
- "Build tests for my new prompt" → create-evals workflow
- "Review this prompt for issues" → audit-prompt workflow
- "Agent says too much" → surgical-edit (brevity constraint)
</quick_commands>

<success_criteria>
This skill succeeds when:
- Eval suites comprehensively cover agent behavior
- Prompt changes are driven by data, not intuition
- Edits are minimal and targeted
- Pass rates improve without regressions
- Voice-specific constraints are preserved
</success_criteria>

---
> Source: [sahiln13/voice-agent-claude-skill](https://github.com/sahiln13/voice-agent-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
