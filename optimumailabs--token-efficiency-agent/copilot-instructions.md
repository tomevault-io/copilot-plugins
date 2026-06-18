## token-efficiency-agent

> Token Efficiency Agent (TEA). Measures and reduces wasted tokens in LLM prompts. Use it to "score this prompt", "optimise this prompt", "cut token waste", "shrink this context", or to wire token optimisation into LangChain, CrewAI, AutoGen, the OpenAI SDK, or the Anthropic SDK. Provides a measurement mode (score only) and an optimisation mode (rewrite the prompt with safe deterministic transforms plus an optional LLM compressor).


# Token Efficiency Agent (TEA)

TEA cuts the tokens an LLM prompt spends without changing what the model is
asked to do. It has two modes.

- **Measure** (`scripts/score.py`): score a prompt, do not change it.
- **Optimise** (`scripts/optimize.py` or the `tea` package): rewrite the prompt
  to spend fewer tokens, and report what changed.

The bundled `tea` package also plugs into LangChain, CrewAI, AutoGen, the
OpenAI SDK, and the Anthropic SDK, so the same optimisation runs inside a
real application, not just from this skill.

## When to use this skill

- "Score this prompt" / "Is this prompt efficient?" -> measure mode.
- "Optimise this prompt" / "Cut the token waste" / "Shrink this context" ->
  optimise mode.
- "How do I add token optimisation to my LangChain / CrewAI / AutoGen app?" ->
  point the user at the matching integration (see the table below) and show the
  three-line snippet.

Do not invoke for general questions that merely mention prompts. This skill is
for explicit measure or optimise requests.

## Measure mode

Run the scorer and report the result. Always invoke the script, never
reimplement the math.

```bash
python product/skill/scripts/score.py \
    --prompt-file /tmp/prompt.txt \
    --query "the user question" \
    --quality 0.78 --quality-supplied \
    --model gpt-4o
```

The script prints JSON with `tokens`, `score`, `cost`, `suggestions`, and
`assumptions`. Render it as a short report: token breakdown, the composite
score S(P), the per-request cost, and the top suggestions. Surface the
`assumptions` so the reader knows which defaults were used.

## Optimise mode

```bash
# Safe transforms only (whitespace, dedupe, oversized few-shot pruning).
python product/skill/scripts/optimize.py \
    --prompt-file /tmp/prompt.txt \
    --query "the user question" --model gpt-4o

# Add relevance-based context dropping (needs the query).
python product/skill/scripts/optimize.py \
    --prompt-file /tmp/prompt.txt \
    --query "the user question" --aggressive --model gpt-4o

# Optimise a chat-messages JSON file instead of a raw prompt.
python product/skill/scripts/optimize.py \
    --messages-file /tmp/chat.json --model gpt-4o
```

The script prints a JSON report (`tokens_before`, `tokens_after`,
`tokens_saved`, `reduction_pct`, `transforms`, `notes`) and writes the
optimised prompt to `--out-file` if given. Report the reduction and which
transforms fired. If `exact_tokenizer` is false, mention that token counts are
approximate because tiktoken is not installed.

What each transform does:

- **whitespace**: collapse blank-line runs and trailing spaces. Preserves code
  fences. Tiny token savings, cleaner prompt.
- **dedupe**: drop duplicate paragraphs and repeated sentences. Targets the
  common RAG failure of retrieving the same passage twice.
- **few_shot**: prune the back half of an oversized few-shot block.
- **drop_context** (aggressive only): drop context chunks whose lexical overlap
  with the query is low. Never empties the context; always keeps the best chunk.
- **compress** (package only): route the prompt through a caller-supplied LLM
  compressor. Guarded so a degenerate compressor output is rejected.

Expect 15 to 35 per cent reduction on bloated prompts from the deterministic
transforms alone. Deeper cuts need the optional LLM compressor.

## Using TEA inside an application

The `tea` package is what a developer imports into their own code. It has no
hard dependency on any framework; each adapter imports its framework lazily.

| Framework | Import | One-liner |
|---|---|---|
| OpenAI SDK | `from tea.integrations.openai_wrap import wrap_openai` | `client = wrap_openai(OpenAI())` |
| Anthropic SDK | `from tea.integrations.anthropic_wrap import wrap_anthropic` | `client = wrap_anthropic(Anthropic())` |
| LangChain | `from tea.integrations.langchain_cb import TEAOptimizer` | `chain = TEAOptimizer(model_name="gpt-4o") \| model` |
| CrewAI | `from tea.integrations.crewai_hook import optimize_agents, optimize_tasks` | `optimize_agents(agents); optimize_tasks(tasks)` |
| AutoGen | `from tea.integrations.autogen_hook import TEAMessageTransform` | add `TEAMessageTransform()` to a `TransformMessages` capability |

Direct API, no framework:

```python
import tea

result = tea.optimize(prompt, query="the user question", model="gpt-4o")
print(result.optimized)      # the shorter prompt
print(result.summary())      # what changed and how much was saved

# Chat messages:
result = tea.optimize(messages, model="gpt-4o")
cheaper_messages = result.optimized
```

Enable deeper compression by passing a compressor callable:

```python
def my_compressor(text: str, target_ratio: float) -> str:
    # call any model to shorten `text` to about target_ratio of its length
    ...

result = tea.optimize(
    prompt, query=q, model="gpt-4o",
    enable=tea.AGGRESSIVE_TRANSFORMS, compressor=my_compressor,
)
```

## Output guidelines

- Numbers come from the script or the package. Never invent them.
- If a transform saved 0 tokens, say so plainly; do not imply otherwise.
- When the optimiser drops context, it preserves the highest-overlap chunk. Tell
  the user the optimiser will never empty the context, so meaning is retained.
- If the script errors, surface the error verbatim and stop.

## Honest limits

- Relevance scoring is lexical overlap, not real attention. It is a safe proxy
  that occasionally keeps a chunk a true attention signal would drop. The
  product brief, section 3.8, describes the attention and ablation path that
  replaces this in later phases.
- Anthropic models have no public tokenizer, so their counts use the cl100k
  fallback and are approximate. Relative before/after comparisons stay valid.
- The deterministic transforms are quality-safe by design. The LLM compressor
  is opt-in and bounded, but any semantic compression carries some risk, so it
  is never on by default.

## Reference

- Product brief: [`product/token_efficiency_agent.md`](../token_efficiency_agent.md)
- Package README: [`README.md`](README.md)
- Self-test: `python -m tea._selftest` from the `product/skill` directory.

---
> Source: [Optimumailabs/Token-Efficiency-Agent](https://github.com/Optimumailabs/Token-Efficiency-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
