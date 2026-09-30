## typesafe-cli

> TypeSafe System One: small typed judgments (noul / choice / score) your code can combine. Not a text generator. JSON in, JSON out.

# typesafe-cli

TypeSafe System One: small typed judgments (noul / choice / score) your code can combine. Not a text generator. JSON in, JSON out.

Use this CLI instead of the TypeSafe agent skill. Live docs: https://docs.typesafe.ai/llms.txt

## Commands

```bash
typesafe-cli noul "Does this convey urgency?" --state "..."
typesafe-cli choice "Which team?" --state "..." --option billing="Payments" --option technical="Bugs" --option sales
typesafe-cli score "How frustrated?" --state "..." --level "Calm" --level "Frustrated" --level "Very angry"
typesafe-cli eval questions.json --state "..."          # many questions, one call (preferred)
typesafe-cli eval request.json                          # full {state, questions, model?} body
typesafe-cli eval - --state-file ticket.json            # questions on stdin
typesafe-cli models
```

`--state-file FILE|-` for JSON or raw text. `--id` names the question (defaults: `noul` / `choice` / `score`). `--dry-run` prints the request and skips the API. `--model` defaults to `jev-latest`.

## When to use

Reach for TypeSafe when software needs programmable common sense: route, rank, extract a closed-set value, verify a claim, score a dimension. Keep rules, math, lookups, and side effects in code. Do not use it to generate prose, code, or explanations.

Ask independent questions over the **same state in one `eval`**. They run in parallel, cannot see each other, and batching is much cheaper/faster than one call per question. A second call is only warranted when an earlier answer is needed to fetch evidence or build the next options.

## Primitives

| Need | Command | Returns |
| --- | --- | --- |
| Whether a condition holds | `noul` | `noul` 0..1 (P(yes)). No separate confidence. Near 0.5 means yes/no are similar, not "medium intensity". One noul per label when several may apply. |
| One of a defined set | `choice` | `choice`, `probabilities`, `confidence` |
| Degree on ordered levels | `score` | `score` (can land between levels), `legend`, `probabilities`, `confidence` |

Choice/Score `confidence` is distribution concentration, not permission to act. Thresholds belong in your code, on your data.

## How to write questions

- **State** is the evidence: source text, identities, policies, current facts. Prefer named JSON fields when context has several parts. Reference nested paths in instructions with backticks: `` `ticket.messages[0].text` ``.
- **Instructions** carry the full meaning. Question ids are for your code and are **not** sent to the model.
- **Criteria** define the possible answers. Choice: option → rubric (`null` if the key is enough). Score: ordered level descriptions that stand on their own (min 2). Noul: optional `{true, false}` glosses (`--yes` / `--no`).
- One narrow, coherent judgment per question. Split independent dimensions. Include a no-match option when nothing may fit. The model cannot choose a value you omitted.
- Strings work for simple questions. For contrasts, exclusions, or examples, put structured objects in an `eval` questions file.

## eval input

Questions map:

```json
{
  "is_urgent": { "type": "noul", "instructions": "Does this convey urgency?" },
  "department": {
    "type": "choice",
    "instructions": "Which team should handle this?",
    "criteria": { "billing": "Payments", "technical": "Bugs", "sales": "Pricing" }
  }
}
```

Or a full request body with `state`, `questions`, and optional `model`.

## Output

```json
{"ok": true, "data": {"model": "jev-latest", "answers": {...}, "usage": {"input_tokens": 312, "output_tokens": 48}}}
{"ok": false, "error": "...", "code": "CONFIG|PARSE|BAD_USAGE|HTTP|ERROR"}
```

Noul answer: `{"type":"noul","noul":0.92}`. Choice adds `choice`, `probabilities`, `confidence`. Score adds `score`, `legend`, `probabilities`, `confidence`.

## Auth

`.env` with `TYPESAFE_API_KEY` (mode 600). Optional: `TYPESAFE_BASE_URL`, `TYPESAFE_DEFAULT_MODEL`.

## Offline check

```bash
ruby test/test_typesafe.rb
```

---
> Source: [gxbvc/typesafe-cli](https://github.com/gxbvc/typesafe-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
