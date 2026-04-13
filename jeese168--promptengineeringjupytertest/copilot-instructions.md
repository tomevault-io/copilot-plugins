## promptengineeringjupytertest

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a prompt engineering educational project consisting of Jupyter notebooks demonstrating various LLM interaction techniques. The primary focus is experimenting with the Qwen3 model series via Ollama, along with external APIs like Moonshot AI's Kimi model.

The notebooks are designed as hands-on experiments for learning prompt engineering principles, testing model capabilities, and comparing different model sizes.

## Development Environment

**Primary Environment**: Google Colab with GPU acceleration (T4/L4/A100)
- Python 3.11
- Jupyter Notebooks (.ipynb files)
- Virtual environment: `.venv/` (for local development)

**Key Dependencies**:
- `ollama` - Local LLM orchestration (installed via shell script in notebooks)
- `requests` - HTTP client for API calls
- `openai` - Python client (for Kimi API compatibility)
- `python-dotenv` - Environment variable management

## Working with Notebooks

### Notebook Structure

Notebooks are self-contained and follow a consistent pattern:

1. **Environment Setup Cell**: Installs Ollama and starts the service
   ```python
   # Typical first cell
   !curl -fsSL https://ollama.com/install.sh | sh
   !nohup ollama serve > ollama.log 2>&1 &
   ```

2. **Model Download Cell**: Pulls required Qwen3 models
   ```python
   !ollama pull qwen3:8b
   # For L4/A100 GPUs: qwen3:14b, qwen3:32b
   ```

3. **Core Function Definitions**: Reusable helper functions for model interaction
   - `get_completion(prompt, model="qwen3:8b", temperature=0.7)` - Basic inference
   - `ollama_chat_qwen(messages, ...)` - Advanced chat interface with streaming
   - `compare_prompts(good_prompt, bad_prompt, title)` - Side-by-side comparison

4. **Experiment Cells**: Demonstrate specific techniques with expected outputs

### Running Notebooks

**In Google Colab**:
- Open `.ipynb` files directly in Colab
- Execute cells sequentially from top to bottom
- First execution installs dependencies and downloads models (~5-20GB depending on model)
- Subsequent executions reuse installed components

**Locally**:
- Requires local Ollama installation: `curl -fsSL https://ollama.com/install.sh | sh`
- Start Jupyter: `jupyter notebook`
- Note: GPU not required but significantly improves performance

### Notebook Series

**Main Learning Path**:
- `00_intro.ipynb` - Introduction to prompt engineering concepts
- `01_principles.ipynb` - Core principles (clarity, specificity, thinking time)
- `02_shot_prompting.ipynb` - Zero-shot, one-shot, few-shot strategies
- `03_output_format.ipynb` - Controlling output format (XML/JSON/HTML)
- `04_delimiters.ipynb` - Using delimiters to prevent prompt injection
- `05_thought_chains.ipynb` - Chain-of-thought reasoning techniques
- `06_react.ipynb` - ReAct (Reasoning + Acting) patterns

**Experimental Notebooks** (`temptest/`):
- `kimi_prompt01.ipynb` - Kimi API testing, context retention experiments
- `local_models_experiment.ipynb` - Local model comparison
- `qwen3_intelligence_benchmark.ipynb` - Multi-model intelligence comparison (8B vs 14B vs 32B)

## Key Architectural Patterns

### 1. Streaming Response Pattern

All notebooks use **real-time streaming** for better UX:

```python
response = requests.post(url, json=payload, stream=True)
for line in response.iter_lines():
    data = json.loads(line.decode('utf-8'))
    content = data['message']['content']
    print(content, end='', flush=True)  # Real-time display
```

This provides ChatGPT-like token-by-token output instead of waiting for complete responses.

### 2. Model Size Optimization

Notebooks are designed to work across different GPU tiers:

**T4 GPU (16GB)**: qwen3:8b, qwen3:14b
**L4/A100 (24GB+)**: qwen3:32b

Model switching is handled by the `model` parameter. Look for `#TODO 用l4的时候解除注释` comments to enable larger models.

### 3. Ollama Configuration

Standard parameters across notebooks:
- `num_ctx: 8192` - Context window size
- `num_predict: 4096` - Maximum output tokens
- `temperature: 0.7` - Default randomness
- `repeat_penalty: 1.07` - Avoid repetition
- `top_p: 0.9` - Nucleus sampling

### 4. Experiment Design Pattern

Educational notebooks follow a **good vs bad prompt** comparison structure:

```python
compare_prompts(
    good_prompt="Clear, specific instructions with delimiters",
    bad_prompt="Vague instructions without structure",
    title="Strategy 1.1: Using Delimiters"
)
```

This shows learners the **impact** of prompt engineering techniques through direct comparison.

## Environment Configuration

### API Keys

For notebooks using external APIs (e.g., Kimi):

1. Create `.env` file in project root:
   ```
   MOONSHOT_API_KEY="your_api_key_here"
   ```

2. Load in notebook:
   ```python
   from dotenv import load_dotenv
   load_dotenv()
   ```

### Ollama Server Management

**Starting Ollama** (done in notebooks):
```python
ollama_server = subprocess.Popen(["ollama", "serve"], ...)
time.sleep(5)  # Wait for startup
```

**Checking Status**:
```bash
curl http://localhost:11434/api/tags
```

**Stopping** (if needed):
```python
ollama_server.terminate()  # Variable from startup cell
```

## Model Selection Guide

From README.md Qwen3 comparison table:

| Model | Parameters | Size (Q4_K_M) | Min VRAM | Colab GPU | Use Case |
|-------|-----------|---------------|----------|-----------|----------|
| qwen3:0.6b | 0.6B | ~400MB | 2GB | T4 ✅ | Lightweight testing |
| qwen3:8b | 8B | ~5.2GB | 6GB | T4 ✅ | **Most recommended** |
| qwen3:14b | 14B | ~8.5GB | 10GB | T4 ✅ | High quality inference |
| qwen3:32b | 32B | ~19GB | 20GB | L4 ✅ | Professional applications |

**Default choice**: `qwen3:8b` - Best balance of quality and performance on T4.

## Common Tasks

### Adding a New Experiment Notebook

1. Copy the structure from `01_principles.ipynb`
2. Include standard cells:
   - Environment setup (Ollama installation)
   - Model download
   - Core function definitions
   - Experiment cells with clear markdown explanations

3. Follow naming convention: `##_topic_name.ipynb` (numbered sequence)

### Testing Prompt Engineering Techniques

Use the `compare_prompts()` pattern:
- Design a **good prompt** that follows best practices
- Design a **bad prompt** that demonstrates common mistakes
- Let the comparison show the impact

### Debugging Model Responses

Enable debug mode in helper functions:
```python
response = ollama_chat_qwen(messages, debug=True)
```

This shows:
- API parameters used
- Response timing
- Token counts
- Thinking process (if enabled)

### Switching Models

Change model parameter in function calls:
```python
# Default
get_completion(prompt, model="qwen3:8b")

# Larger model (requires L4+ GPU)
get_completion(prompt, model="qwen3:32b")
```

## Important Notes

- **No build process**: Each notebook is self-contained
- **GPU memory**: Monitor with `nvidia-smi` when running locally
- **Colab sessions**: Free tier has 12-hour limit, save work frequently
- **Model downloads**: First run downloads models (can take 5-15 minutes)
- **Thinking mode**: Generally disabled (`enable_thinking=False`) to avoid content truncation
- **Context limits**: Qwen3 supports up to 8192 context window (configured via `num_ctx`)

## Related Documentation

- **README.md**: Qwen3 model specifications and Colab deployment guide
- **GEMINI.md**: Project overview and development conventions
- **吴恩达 x OpenAI 提示词工程课程**: Source material for learning path (referenced in notebooks)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeese168) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
