## pydantic-boilerplate

> Located in [main.py](mdc:main.py), the `Joke` model is a simple demonstration of using Pydantic for data validation:

# Pydantic Models

## Joke Model
Located in [main.py](mdc:main.py), the `Joke` model is a simple demonstration of using Pydantic for data validation:

```python
class Joke(BaseModel):
    title: str  # Title of the joke
    joke: str   # Content of the joke
```

This model is used with the pydantic-ai library to validate and structure the output from AI models. The system prompt instructs the AI to return data in this format, and pydantic-ai handles the parsing and validation automatically.

## Using Models with AI

The project demonstrates how to use Pydantic models as structured output types with AI models:

```python
agent = Agent(
    model=gemini_25_flash,
    system_prompt=system_prompt,
    output_type=Joke,  # Tell the agent to return a Joke object
)
```

This pattern can be extended to create more complex data validation schemas for AI responses.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxmux-xyz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
