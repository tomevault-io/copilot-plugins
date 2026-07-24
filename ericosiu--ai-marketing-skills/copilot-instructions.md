## ai-marketing-skills

> Evaluate any AI-powered feature: chat agents, content generators, classifiers, summarizers, code generators, or any system that takes input and produces AI output. Defines what "good" looks like through conversation, generates test cases, runs them, and scores results.

# /eval — AI Output Evaluation Suite

Evaluate any AI-powered feature: chat agents, content generators, classifiers, summarizers, code generators, or any system that takes input and produces AI output. Defines what "good" looks like through conversation, generates test cases, runs them, and scores results.

## When to use
- After changing prompts, models, or system instructions
- Before deploying any AI feature to production
- Weekly to catch quality drift
- When defining quality standards for a new AI feature

## How to invoke

```
/eval                          # Start fresh or run existing config
/eval --run                    # Skip setup, run existing eval.config.json
/eval --verbose                # Show full outputs during run
/eval --baseline               # Save results as baseline for future comparisons
```

## Instructions for Claude

When the user invokes `/eval`:

### Step 1: Check for existing config

```bash
ls eval.config.json 2>/dev/null && echo "CONFIG_EXISTS" || echo "NO_CONFIG"
```

If `CONFIG_EXISTS` and user did NOT pass `--run`: ask "You have an existing eval config with N scenarios. Want to run it, update it, or start fresh?"
If `CONFIG_EXISTS` and user passed `--run`: skip to Step 4.
If `NO_CONFIG`: proceed to Step 2.

### Step 2: Understand what you're evaluating

Ask the user these questions ONE AT A TIME via AskUserQuestion. The first question determines the flow for the rest.

**Q1: "What type of AI output are you evaluating?"**
Options:
- A) Chat agent / conversational AI (multi-turn: user sends messages, agent responds)
- B) Content generator (single input, long-form output: blog posts, emails, plans)
- C) Classifier / scorer (input data, output label or score)
- D) Summarizer / extractor (input document, output summary or structured data)
- E) Something else (describe it)

**Q2: "Describe what it does in one sentence."**
Example: "It generates SEO-optimized blog post outlines from a keyword."

**Q3 (varies by type):**

For CHAT AGENTS (A):
- "What's the API endpoint? How do I send a message and get a response?"
- "What should a good conversation look like? List 3-5 things the agent should always do."
- "What should it never do?"
- "Who are ideal users vs. who should be turned away?"

For CONTENT GENERATORS (B):
- "What's the API endpoint or function? What input does it take?"
- "What makes the output GOOD? (length, tone, structure, accuracy, keywords)"
- "What makes the output BAD? (hallucinations, wrong tone, too short/long, missing sections)"
- "Show me one example of good output if you have it."

For CLASSIFIERS (C):
- "What's the API endpoint or function?"
- "What are the possible output labels/scores?"
- "Do you have labeled test data (known correct answers)?"
- "What's the cost of a false positive vs. false negative?"

For SUMMARIZERS (D):
- "What's the API endpoint or function?"
- "What should the summary include? What should it exclude?"
- "What's the max length?"
- "Should it preserve specific details (names, numbers, dates)?"

For OTHER (E):
- "Walk me through: what goes in, what comes out?"
- "How do you know when the output is good vs. bad?"
- "What are the failure modes you're worried about?"

### Step 3: Generate eval config

Based on the user's answers, generate test cases appropriate to the type:

**Chat agents:** 10-20 multi-turn conversation scenarios (qualified users, unqualified users, edge cases, product knowledge tests, hostile users, capability boundaries)

**Content generators:** 10-15 input variations testing different topics, edge cases, and quality dimensions (accuracy, tone, length, structure, keyword inclusion)

**Classifiers:** 20-30 test inputs with known correct labels, covering each class, edge cases, and adversarial inputs

**Summarizers:** 10-15 test documents of varying length and complexity, checking for completeness, accuracy, length compliance, and hallucination

**For all types, generate criteria based on:**
- Things the user said make output GOOD -> `contains`, `regex`, `max_length` checks
- Things the user said make output BAD -> `not_contains` checks
- Type-specific quality checks (see criterion types below)

Write the config to `eval.config.json`. Show the user and ask: "Does this cover what matters? Want to add or change anything?"

### Step 4: Run the evals

```bash
npx tsx .claude/skills/eval/run-eval.ts [--config eval.config.json] [--verbose] [--baseline]
```

The runner handles all types. For chat agents, it sends messages sequentially and evaluates the full conversation. For single-input types, it sends one request per scenario and evaluates the output.

### Step 5: Report results

1. Summary table (scenario x criterion, pass/fail)
2. Overall score: "X/Y criteria passed (Z%)"
3. If baseline exists: "Score changed from A% to B%"
4. Regressions: scenarios that got worse since last run
5. Top 3 failures with diagnosis
6. Recommendation: fix or ship

### Step 6: Iterate

If failures exist:
1. Read the failing output from eval-results.json
2. Diagnose root cause (prompt issue, missing data, model limitation)
3. Suggest a fix
4. After fix: re-run to verify

## Config format

```json
{
  "name": "My AI Feature Eval",
  "type": "chat | content | classifier | summarizer | custom",
  "endpoint": "https://my-api.com/endpoint",
  "method": "POST",
  "headers": {},
  "request_template": {},
  "response_field": "response",
  "threshold": 80,
  "good_behaviors": [],
  "bad_behaviors": [],
  "scenarios": []
}
```

The `type` field determines how scenarios are executed:
- `chat`: multi-turn (sends messages sequentially, maintains history)
- `content | classifier | summarizer | custom`: single-turn (one request per scenario)

## Criterion types

| Type | Works for | Description |
|------|-----------|-------------|
| `contains` | All | Output contains a string (case-insensitive) |
| `not_contains` | All | Output does NOT contain a string |
| `regex` | All | Output matches a regex pattern |
| `max_length` | All | Output is under N characters |
| `min_length` | Content | Output is at least N characters |
| `max_sentences` | Chat, Content | Output is under N sentences |
| `response_time` | All | API responds within N milliseconds |
| `json_valid` | Classifier, Custom | Output is valid JSON |
| `json_field_equals` | Classifier | A JSON field equals an expected value |
| `no_hallucination` | Content, Summarizer | Output doesn't contain claims not in the input |
| `preserves_names` | Summarizer | Key names from input appear in output |
| `preserves_numbers` | Summarizer | Key numbers from input appear in output |

---
> Source: [ericosiu/ai-marketing-skills](https://github.com/ericosiu/ai-marketing-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
