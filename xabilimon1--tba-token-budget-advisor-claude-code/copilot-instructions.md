## tba-token-budget-advisor-claude-code

> >


# Token Budget Advisor

A skill that intercepts the response flow so the user can make an informed
choice about how much depth / how many tokens to spend BEFORE the answer is
generated.

## Workflow

### Step 1: Analyze the prompt

Run the estimator on the user's prompt. The script lives at
`scripts/token_estimator.py` inside this skill's directory.

```bash
python3 <SKILL_DIR>/scripts/token_estimator.py --text "USER_PROMPT"
```

For long prompts or prompts containing quotes, use a temp file:

```bash
cat > /tmp/_tba_prompt.txt << 'PROMPT_EOF'
USER_PROMPT
PROMPT_EOF
python3 <SKILL_DIR>/scripts/token_estimator.py --file /tmp/_tba_prompt.txt
```

Replace `<SKILL_DIR>` with the real install path (usually
`.claude/skills/token-budget-advisor` or whatever appears in your config).

The script returns JSON with: `input_tokens`, `detected_language`,
`detected_type`, `complexity`, `response_estimates` (per level
25/50/75/100), and `total_estimates`.

### Step 2: Present the options to the user

Show the information clearly BEFORE answering the real prompt. See
`references/calibration.md` for what each level includes and omits.

Recommended format:

```
Prompt analysis
---------------
Input: ~X tokens | Type: [type] | Complexity: [level]

Choose your depth level:

[1] Essential   (25%)  -> ~Y tokens   Direct answer only
[2] Moderate    (50%)  -> ~Z tokens   Answer + context + 1 example
[3] Detailed    (75%)  -> ~W tokens   Full answer with alternatives
[4] Exhaustive (100%)  -> ~V tokens   Everything, no limits

Heuristic estimate (~85-90% accuracy, +/- 15%).
```

### Step 3: Wait for the choice

Ask the user which level they prefer. In Claude Code's terminal, render the
options as plain text and wait for their reply.

### Step 4: Respond at the chosen level

| Level | Length | What to include |
|-------|--------|-----------------|
| 25% Essential | 2-4 sentences max | Just the direct answer. No preamble. |
| 50% Moderate | 1-3 paragraphs | Answer + minimal context + 1 example if relevant. |
| 75% Detailed | Structured response | Multiple examples, pros/cons, alternatives. |
| 100% Exhaustive | No limit | Everything: full analysis, complete code, every perspective. |

## Shortcuts

If the user already states a level, do not ask -- just answer at that level:

- "at 25%" / "al 25%" / "short version" / "tldr" / "summary"            -> 25%
- "at 50%" / "al 50%" / "moderate" / "normal"                            -> 50%
- "at 75%" / "al 75%" / "detailed" / "complete"                          -> 75%
- "at 100%" / "al 100%" / "exhaustive" / "everything" / "no limit"       -> 100%

If the user picked a level in a previous message of the same session, keep
that level for subsequent responses without asking again -- unless they
ask to change it.

---
> Source: [Xabilimon1/TBA-Token-Budget-Advisor-Claude-Code](https://github.com/Xabilimon1/TBA-Token-Budget-Advisor-Claude-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
