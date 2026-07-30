## effgen

> effGen is a large framework, but it follows a small set of consistent

# API Conventions

effGen is a large framework, but it follows a small set of consistent
conventions. Learn these once and the rest of the surface is predictable.

## Importing

The common entry points live at the top level:

```python
from effgen import Agent, AgentConfig, load_model, create_agent, list_presets
from effgen import tool, Tool          # low-boilerplate tool authoring
from effgen.tools.builtin import Calculator, WebSearch  # built-in tools
```

`import effgen` is lazy — names resolve on first access, so importing the
package is cheap even though the public surface is large.

## Creating an agent

There are two equivalent paths; pick whichever reads better for you:

```python
# Preset + model id (shortest):
agent = create_agent("math", "gpt-5-nano")

# Explicit config (full control):
agent = Agent(AgentConfig(name="my-agent", model="gpt-5-nano", tools=[...]))
```

A `model` is always **required** — effGen never silently picks a paid cloud
model. Pass a model id (string) or a loaded model instance. To choose a default
once, set the `EFFGEN_DEFAULT_MODEL` environment variable.

## Models and providers

Model ids are strings. When an id is unambiguous it routes automatically;
otherwise prefix it with the provider:

```
"gpt-5-nano"                     # routes to OpenAI
"openai:gpt-5-nano"              # explicit provider prefix
"Qwen/Qwen2.5-1.5B-Instruct"     # local (Transformers/vLLM)
```

You can also pass `provider=` to `AgentConfig` / `load_model`, or `--provider`
on the CLI. A wrong id fails closed with a "did you mean…/available now…" hint —
run `effgen models list` to browse and `effgen doctor` to see which providers
are usable.

## Results

Every `agent.run(...)` returns an `AgentResponse`:

```python
result = agent.run("What is 17% of 250?")

print(result)            # the answer (str) — __str__ returns result.output
result.output            # the answer string
result.text              # read-only alias of .output
result.content           # read-only alias of .output
result.success           # bool — never True with an empty answer
result.tokens_used       # int
result.execution_time    # float seconds
result.to_dict()         # full structured detail (trace, cost, metadata)
```

On failure, `success` is `False`, the message is clear and redacted, and
`result.metadata["error"]` is a structured `{type, category, provider, model,
message, retryable}` dict — identical whether the failure came from the direct
or the tool path.

## Streaming

`agent.stream(task)` yields successive **answer-text** `str` chunks; joining them
reconstructs the (sanitized) answer. The iterator ending is the "done" signal; a
provider failure raises a typed error rather than silently ending the stream.

```python
for chunk in agent.stream("Write a haiku about the sea"):
    print(chunk, end="", flush=True)
```

This holds for **tool-using agents** too: the default text stream is the answer
only — the internal ReAct scaffolding (`Thought:` / `Action:` / `Observation:` /
`Final Answer:`) is never part of the text payload. To observe the steps as they
happen, either pass the `on_thought` / `on_tool_call` / `on_observation`
callbacks, or opt into typed events:

```python
for event in agent.stream(task, include_events=True):
    if event.kind == "answer":
        print(event.text, end="", flush=True)
    elif event.kind == "tool_call":
        print(f"\n[calling {event.tool}]")
```

`include_events=True` yields `StreamEvent` objects with a `kind` of `answer`,
`thought`, `tool_call`, `observation`, `status`, or `usage`; concatenating the
`answer` events still reconstructs the final answer. For the best tool-use
quality on capable models, `agent.run(task)` (which uses native function-calling
where available) is recommended over streaming.

### Usage after a stream

The last event of an `include_events=True` stream is a `usage` event carrying
what the run cost, and the same dict is on `agent.last_stream_usage` after any
stream — including text mode — so a streamed turn can be tallied without running
the prompt a second time:

```python
chunks = list(agent.stream(task))
usage = agent.last_stream_usage
print(usage["total_tokens"], usage["cost_usd"], usage["ttft_ms"])
```

The keys are `prompt_tokens`, `completion_tokens`, `total_tokens`, `cost_usd`
(`None` for a model with no published price), `latency_ms`, `ttft_ms` (time to
the first answer token), `model_calls` (above one on a tool-using run), and
`estimated` — `True` when the token counts were counted locally because the
backend reported none, as local engines do.

Over the OpenAI-compatible server the same numbers arrive on the final
`stream_options.include_usage` chunk, whose `effgen` object carries `cost_usd`
alongside the standard `usage` block.

## Tools

The recommended way to author a tool is the `@tool` decorator (it wraps the full
`BaseTool` machinery for you):

```python
from effgen import tool

@tool
def word_count(text: str) -> int:
    """Count the words in a piece of text."""
    return len(text.split())
```

The decorated object is a real tool instance — drop it into
`AgentConfig(tools=[...])` and it works with provider-native function-calling
too. `Tool.from_function(fn)` is the non-decorator equivalent. For rich
validation or lifecycle hooks, subclass `BaseTool` directly (see
[Custom Tools](../tutorials/custom-tools.md)). Multi-action built-in tools take
a canonical `operation` selector and accept common synonyms (`action`, and
natural verbs) so the obvious call works.

## Errors

User-facing errors are typed and actionable: a one-line cause plus how to fix
it. Unknown preset, model, or tool names raise typed errors with a fuzzy "did
you mean?" suggestion instead of a bare `KeyError`.

## Type hints

effGen ships a `py.typed` marker, so your editor and `mypy`/`pyright` see effGen's
annotations on the public surface (`from effgen import ...`). The public surface
is checked two ways in CI: a deterministic gate ensures every advertised name
resolves with an introspectable signature, and an advisory `mypy` lane
type-checks the public modules. Internal modules carry best-effort annotations
that may still tighten over time; rely on the documented public surface.

## Counts (tools, presets, providers, models)

The headline counts drift as the framework grows, so they are **generated**, not
hand-maintained. Get the exact current numbers from the live package:

```bash
python scripts/gen_counts.py           # human-readable table
python scripts/gen_counts.py --json    # machine-readable
```

This is the single source of truth for "how many tools/presets/providers/models
does effGen ship".

---
> Source: [ctrl-gaurav/effGen](https://github.com/ctrl-gaurav/effGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
