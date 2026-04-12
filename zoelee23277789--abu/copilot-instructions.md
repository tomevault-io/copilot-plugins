## abu

> The nano-browser-use project uses Laminar for evaluating AI agent performance. Evaluations test the browser agent's ability to complete tasks and provide structured feedback on performance.

# Evaluation Workflow for nano-browser-use

## Overview
The nano-browser-use project uses Laminar for evaluating AI agent performance. Evaluations test the browser agent's ability to complete tasks and provide structured feedback on performance.

## Project Structure

### Evaluation Files
- **Evaluation Scripts**: Located in [nano_browser_use/evals/](mdc:nano_browser_use/evals/)
- **Datasets**: Located in [eval_data/](mdc:eval_data/)
- **Logs**: Generated in [logs/](mdc:logs/) with organized subdirectories

### Key Files
- [nano_browser_use/evals/basic_eval.py](mdc:nano_browser_use/evals/basic_eval.py) - Example evaluation implementation
- [eval_data/sample_dataset.json](mdc:eval_data/sample_dataset.json) - Example dataset format

## Evaluation Components

### 1. Dataset Format
Datasets are JSON files with the following structure:
```json
[
    {
        "data": {
            "query": "user query string",
            "index": 0
        },
        "target": "expected result"
    }
]
```

### 2. Evaluation Script Structure
Every evaluation script must include:

#### Required Imports
```python
import asyncio
import json
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from lmnr import evaluate
from pydantic import BaseModel, Field
from nano_browser_use.agent.service import Agent
from nano_browser_use.utils import initialize_laminar
```

#### Configuration Constants
```python
load_dotenv()
initialize_laminar()

NAME = "eval-name-v1"  # Unique evaluation name
GROUP_NAME = "eval-group"  # Group for organizing related evals
AGENT_MODEL = "gpt-4.1-mini"  # Model for the agent to test
SCORER_MODEL = "gpt-4.1"  # Model for evaluation/scoring
DATASET_PATH = "eval_data/your_dataset.json"
```

#### Pydantic Model for Results
```python
class YourEval(BaseModel):
    evaluation_feedback: str = Field(description="Detailed feedback prompt")
    evaluation_score: int = Field(description="Scoring prompt")
    is_correct: bool = Field(description="Correctness check prompt")
```

#### LLM Setup
```python
llm = ChatOpenAI(model=SCORER_MODEL, temperature=0.0)
structured_llm = llm.with_structured_output(YourEval, include_raw=True)
```

#### Executor Function
```python
async def run_browser_agent(data: dict) -> dict[str, str | int]:
    agent = Agent(
        query=data["query"],
        model_name=AGENT_MODEL,
        max_steps=20,
        temperature=0.0,
        headless=True,  # Use True for evaluations
        log_stdout=False,
        screenshot=True,
        log_file=f"logs/{GROUP_NAME}/{NAME}/query_{data['index']}.log",
        capture_logs_in_memory=True,  # Enable in-memory capture for immediate access
    )
    await agent.run()
    return {
        "log_file_path": agent.log_file,
        "input_tokens": agent.input_tokens_count,
        "output_tokens": agent.output_tokens_count,
        "total_tokens": agent.total_tokens_count,
        "trace": agent.logger.get_logs_as_string(),
    }
```

#### Evaluator Function
```python
async def agent_output_evaluation(
    outputs: dict, target: str | None = None
) -> dict[str, int | str]:
    log_str = outputs["trace"]

    # Build evaluation prompts
    EVAL_SYSTEM_PROMPT = "Your evaluation system prompt"
    user_message_content = f"### Trace\n{log_str}\n\n### Ground Truth\n{target}"

    messages = [
        {"role": "system", "content": EVAL_SYSTEM_PROMPT},
        {"role": "user", "content": user_message_content},
    ]

    # Run evaluation
    evaluation = await structured_llm.ainvoke(messages)
    model = evaluation["parsed"]
    model_raw = evaluation["raw"]

    # Append evaluation to log
    with open(outputs["log_file_path"], "a") as f:
        f.write(f"\n\nEvaluation:\n{model.evaluation_feedback}")

    return {
        "is_correct": int(model.is_correct),
        "evaluation_score": float(model.evaluation_score),
        "input_tokens": int(outputs["input_tokens"]),
        "output_tokens": int(outputs["output_tokens"]),
        "total_tokens": int(outputs["total_tokens"]),
        "scorer_input_tokens": int(model_raw.usage_metadata.get("input_tokens", 0)),
        "scorer_output_tokens": int(model_raw.usage_metadata.get("output_tokens", 0)),
        "scorer_total_tokens": int(model_raw.usage_metadata.get("total_tokens", 0)),
    }
```

#### Evaluation Execution
```python
evaluate(
    data=json.load(open(DATASET_PATH)),
    executor=run_browser_agent,
    evaluators={"agent_output_evaluation": agent_output_evaluation},
    name=NAME,
    group_name=GROUP_NAME,
    metadata={"agent_model": AGENT_MODEL, "scorer_model": SCORER_MODEL},
)
```

## Running Evaluations

### Prerequisites
1. **Activate Virtual Environment**: Always activate before running evals
   ```bash
   source .venv/bin/activate
   ```

2. **Environment Variables**: Ensure `.env` file contains required API keys
   ```bash
   OPENAI_API_KEY=your_key_here
   ```

### Execution Command
```bash
source .venv/bin/activate && lmnr eval nano_browser_use/evals/your_eval.py
```

### Example Command
```bash
source .venv/bin/activate && lmnr eval nano_browser_use/evals/basic_eval.py
```

## Evaluation Best Practices

### 1. Naming Conventions
- Use descriptive names: `basic_eval-v1`, `navigation_eval-v2`
- Include version numbers for tracking changes
- Use consistent group names for related evaluations

### 2. Dataset Design
- Include diverse test cases covering different scenarios
- Provide clear, unambiguous queries
- Include expected results for automated scoring
- Use realistic, real-world tasks

### 3. Evaluation Prompts
- Be specific about evaluation criteria
- Use structured scoring (0-10 scale recommended)
- Include both qualitative feedback and quantitative scores
- Consider multiple evaluation dimensions (correctness, efficiency, robustness)

### 4. Logging and Debugging
- Always use `headless=True` for evaluations
- Enable screenshots for visual debugging
- Check generated logs in `logs/{GROUP_NAME}/{NAME}/`
- Review evaluation feedback appended to log files

### 5. Performance Considerations
- Set appropriate `max_steps` to prevent infinite loops
- Monitor token usage for cost optimization
- Use `temperature=0.0` for consistent results
- Consider batching evaluations for efficiency
- Use different models for agent execution vs evaluation scoring (e.g., `gpt-4.1-mini` for agent, `gpt-4.1` for scoring)

## Troubleshooting

### Common Issues
1. **Import Errors**: Ensure virtual environment is activated
2. **API Key Errors**: Check `.env` file configuration
3. **Dataset Path Errors**: Verify relative paths from project root
4. **Log Directory Errors**: Ensure `logs/` directory exists

### Debugging Steps
1. Check log files in `logs/{GROUP_NAME}/{NAME}/`
2. Review evaluation feedback in log files
3. Verify dataset format matches expected structure
4. Test individual components (executor, evaluator) separately

## Evaluation Metrics

### Standard Metrics
- **Correctness**: Binary success/failure
- **Score**: 0-10 quality assessment
- **Token Usage**: Input/output/total tokens
- **Scorer Tokens**: Evaluation LLM usage

### Custom Metrics
Add additional metrics to evaluator return dictionary as needed for specific evaluation goals.

## Integration with Laminar

The evaluation system integrates with Laminar for:
- Automated evaluation execution
- Result tracking and comparison
- Performance monitoring
- Experiment management

Use Laminar's web interface to view evaluation results and compare different runs.
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZoeLee23277789)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ZoeLee23277789)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
