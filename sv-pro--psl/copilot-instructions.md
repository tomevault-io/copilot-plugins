## psl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Prompt Semantic Linter (PSL)** is an LLM-powered linting tool that detects semantic issues in system prompts that cause hallucinations and unpredictable behavior. Based on A.G.I.L.E. principles from the IntentHub framework.

## Architecture

PSL uses a three-stage architecture:

1. **Semantic Parser** ([backend/psl/parser.py](backend/psl/parser.py)) - Uses an LLM to convert natural language prompts into structured Intermediate Representation (IR)
2. **Validator** ([backend/psl/validator.py](backend/psl/validator.py)) - Runs linting rules against the IR to detect issues
3. **Didactic Evaluator** ([backend/psl/evaluation.py](backend/psl/evaluation.py)) - Tests prompts against models to measure didactic effectiveness

### Core Components

- **IR Schema** ([backend/psl/ir.py](backend/psl/ir.py)) - Pydantic models defining the structured representation:
  - `ComputedField` - Fields requiring computation/extraction
  - `Constraint` - Output/behavior constraints
  - `DomainTerm` - Domain-specific terminology
  - `IR` - Top-level intermediate representation

- **LLM Adapter Layer** ([backend/psl/adapters/](backend/psl/adapters/)):
  - `base.py` - Abstract `LLMAdapter` interface with `LLMMessage` and `LLMResponse`
  - `openai_adapter.py` - OpenAI implementation using native `openai` SDK
  - `anthropic_adapter.py` - Anthropic implementation using native `anthropic` SDK
  - `ollama_adapter.py` - Ollama implementation using `httpx` for local models
  - `factory.py` - Factory pattern with `get_adapter()` and `list_available_models()`
  - `config.py` - YAML config loader for model registry
  - `models.yaml` - **Configurable model list** (add/remove models without code changes)
  - **26 models supported** across 4 providers (OpenAI, Anthropic, Google, Ollama)
  - Replaces LiteLLM with direct API calls for latest model support

- **Rules System** ([backend/psl/rules/](backend/psl/rules/)):
  - `base.py` - Abstract `Rule` class and `LintError` model
  - `hallucination.py` - `NoUndefinedComputedFieldsRule` (currently the only rule)
  - Each rule implements `check(ir: IR) -> List[LintError]`

- **Prompt Examples** ([backend/psl/examples/](backend/psl/examples/)):
  - `bad/` - 4 prompts demonstrating semantic issues (K8s, finance, code, sentiment)
  - `good/` - 4 fixed versions with proper definitions and fallback strategies
  - `contexts/` - Test inputs for each example
  - `metadata.json` - Example descriptions, categories, and expected behaviors

- **Didactic Evaluation** ([backend/psl/evaluation.py](backend/psl/evaluation.py)):
  - Tests all prompts × all models to measure didactic effectiveness
  - **Didactic scoring logic**:
    - Bad prompt + hallucination → ✅ Success (demonstrates PSL's value)
    - Good prompt + correct output → ✅ Success (shows solutions work)
    - Bad prompt + no hallucination → ❌ Failure (model too conservative)
    - Good prompt + hallucination → ❌ Failure (model unreliable)
  - Generates evaluation matrix with success rates per model
  - Hallucination detection heuristics

- **FastAPI Backend** ([backend/psl/api.py](backend/psl/api.py)):
  - `POST /lint` - Analyzes prompt for semantic issues, accepts `{prompt: str, model: str}`
    - Returns `{ir: IR, errors: List[LintError], summary: dict}`
  - `POST /execute` - Executes prompt with context to show actual LLM behavior
    - Accepts `{prompt: str, context: str, model: str}`
    - Returns `{output: str, model: str, has_errors: bool}`
  - `GET /models` - Lists all available models grouped by provider
  - `GET /examples` - Fetches all prompt examples with content
  - `POST /evaluate` - Runs didactic evaluation matrix
    - Accepts `{example_ids?: string[], models?: string[]}`
    - Returns `{results: [...], summary: {...}}`
  - CORS enabled for localhost:5173 (Vite dev server)

- **Health Check System** ([backend/psl/health.py](backend/psl/health.py), [backend/psl/cli.py](backend/psl/cli.py)):
  - Uses adapter layer for consistent health checks
  - Verifies API connectivity and model availability
  - Supports OpenAI, Anthropic Claude, and Ollama
  - Used via `make healthcheck` or specific provider checks

### Frontend Architecture

React + TypeScript + Vite app with three-column input layout:
- **Left**: System Prompt editor (Monaco)
- **Center**: Context/Input editor (Monaco, YAML syntax)
- **Right**: Action buttons (Lint Only, Execute Only, Lint + Execute)

Results displayed in two-column layout:
- **Left**: Lint results with issues and suggestions
- **Right**: Actual LLM output showing real behavior (hallucinations or correct output)

The UI correlates lint errors with execution results to demonstrate the impact of semantic issues.

**Component Structure:**
- All UI code is in [frontend/src/App.tsx](frontend/src/App.tsx) - no separate component files
- Uses Monaco Editor for code editing experience
- Uses Tailwind CSS v4 with PostCSS for styling
- API client in [frontend/src/api/client.ts](frontend/src/api/client.ts)

## Common Development Commands

### Quick Start (Root Level)

```bash
# Run both backend and frontend in parallel (recommended)
make dev

# Install all dependencies (backend + frontend)
make install

# Run health checks
make healthcheck

# Run tests
make test

# Clean temporary files
make clean
```

### Backend Only

```bash
cd backend

# Run development server (port 8000)
python main.py

# Run tests
make test                 # Checks environment and runs pytest
pytest tests/ -v          # Run tests directly

# Health checks (verify API keys and model availability)
make healthcheck          # Check all providers
make check-openai         # Check OpenAI specifically
make check-anthropic      # Check Anthropic Claude
make check-ollama         # Check Ollama (local)
make list-models          # List all available models

# Setup environment
make setup                # Create virtual environment
make install              # Install dependencies
make init                 # Full setup (venv + deps + healthcheck)

# Cleanup
make clean                # Remove cache and temporary files
```

### Frontend Only

```bash
cd frontend

# Run development server (port 5173)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Lint TypeScript
npm run lint
```

## Working with LLM Providers

PSL uses a **custom adapter layer** for multi-provider support with direct API calls. API keys are configured in `backend/.env`:

```bash
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-proj-...
GOOGLE_API_KEY=AIza...  # For Gemini models
OLLAMA_API_BASE=http://localhost:11434  # Optional, for local models
```

**Supported Models (26 total):**

**OpenAI** (via `openai` SDK):

- `gpt-4`, `gpt-4-turbo`, `gpt-4-turbo-preview`
- `gpt-3.5-turbo`, `gpt-3.5-turbo-16k`

**Anthropic** (via `anthropic` SDK):

- `claude-sonnet-4-5-20250929` ✨ **Latest model**
- `claude-3-5-sonnet-20241022`, `claude-3-5-haiku-20241022`
- `claude-3-opus-20240229`, `claude-3-sonnet-20240229`, `claude-3-haiku-20240307`

**Google** (via REST API):

- `gemini-1.5-pro`, `gemini-1.5-flash`
- `gemini-pro`, `gemini-pro-vision`

**Ollama** (via `httpx` for local models):

- `ollama/llama2`, `ollama/llama2:13b`, `ollama/llama2:70b`
- `ollama/mistral`, `ollama/mixtral`, `ollama/codellama`
- `ollama/phi`, `ollama/neural-chat`
- `ollama/gemma`, `ollama/gemma:2b`, `ollama/gemma:7b`

**Model Configuration:**

Models are configured in [backend/psl/adapters/models.yaml](backend/psl/adapters/models.yaml). To add new models:

1. Edit `models.yaml` and add the model to the appropriate provider section
2. The factory will automatically load the new model on next import
3. Verify with `make list-models` or `make healthcheck`

**Model Support:**
All models are supported through direct SDK integration - no LiteLLM dependency means latest models work immediately. Verify availability with `make healthcheck` or `make list-models`.

## Testing Philosophy

- Tests use real API calls to LLMs (not mocked)
- Environment variables must be loaded via `tests/conftest.py`
- Fixture prompts in `tests/fixtures/` demonstrate good vs bad examples
- Tests verify that bad prompts trigger errors and good prompts pass

### Running Tests

```bash
cd backend

# Recommended: Use make (validates environment)
make test

# Direct pytest (ensure venv is activated)
pytest tests/ -v

# Run specific test file
pytest tests/test_rules.py -v

# Run with output capture disabled (see print statements)
pytest tests/ -v -s
```

## Key Implementation Details

### Adding New Rules

1. Create new rule class in `backend/psl/rules/`:
```python
from .base import Rule, LintError
from ..ir import IR

class MyNewRule(Rule):
    @property
    def name(self) -> str:
        return "my-rule-name"

    @property
    def description(self) -> str:
        return "What this rule checks for"

    def check(self, ir: IR) -> List[LintError]:
        errors = []
        # Analyze ir and append LintError instances
        return errors
```

2. Register in `backend/psl/validator.py`:
```python
from .rules.my_module import MyNewRule

class Validator:
    def __init__(self):
        self.rules = [
            NoUndefinedComputedFieldsRule(),
            MyNewRule(),  # Add here
        ]
```

### Modifying the IR Schema

Changes to IR models in [backend/psl/ir.py](backend/psl/ir.py) require:
1. Update the Pydantic model
2. Update `SemanticParser.SYSTEM_PROMPT` in [backend/psl/parser.py](backend/psl/parser.py) to extract new fields
3. Update any rules that depend on the modified fields
4. Update frontend TypeScript types in [frontend/src/api/client.ts](frontend/src/api/client.ts) if needed

### Understanding the Semantic Parser

The SemanticParser is the most critical component - it uses an LLM to convert natural language prompts into structured IR. Key implementation details:

- **Uses LLM-to-analyze-LLM pattern**: An LLM (e.g., GPT-4) analyzes another LLM's system prompt
- **Structured extraction via JSON**: The parser prompt explicitly defines the IR schema and requests JSON output
- **Low temperature (0.1)**: Ensures consistent, deterministic parsing
- **Pattern matching**: Looks for keywords like "calculate", "extract", "compute" to identify computed fields
- **No mocking in tests**: Parser tests use real LLM calls to ensure prompt engineering works correctly

When modifying the parser prompt in [backend/psl/parser.py](backend/psl/parser.py):
- Keep the schema definition in sync with [backend/psl/ir.py](backend/psl/ir.py)
- Test with fixture prompts in `tests/fixtures/` to ensure new extractions work
- Validate JSON output is parseable by Pydantic models

### Environment Setup Notes

- Backend uses Python 3.11+ (3.13 recommended with pyenv)
- Virtual environment management via pyenv or venv
- The `make test` command validates environment activation
- Python 3.13 may require Rust toolchain for pydantic-core compilation (see [TROUBLESHOOTING.md](TROUBLESHOOTING.md))

## Demo Use Case

The primary demo is the **Kubernetes metrics example** showing how PSL detects hallucination-prone prompts:

**Bad prompt** (`tests/fixtures/bad_prompts/k8s_metrics.txt`):
- Asks for metrics like `pod_density_ratio`, `mesh_coherence_index`
- No definitions provided
- LLM hallucinates numbers

**Fixed prompt** (`tests/fixtures/good_prompts/k8s_metrics_fixed.txt`):
- Provides explicit formulas
- Uses fallback strategy (null if missing data)
- Prevents hallucinations

## API Testing Without Frontend

You can test the backend API directly using curl:

```bash
# Health check
curl http://localhost:8000/health

# Lint a prompt
curl -X POST http://localhost:8000/lint \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "You are an expert. Calculate metrics: pod_density_ratio, mesh_coherence_index",
    "model": "gpt-4"
  }'

# Execute a prompt with context
curl -X POST http://localhost:8000/execute \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Extract container count from this manifest",
    "context": "apiVersion: v1\nkind: Pod\nmetadata:\n  name: test",
    "model": "gpt-4"
  }'
```

## Technology Stack Details

### Backend

- **Python**: 3.11+ required (3.13 recommended with pyenv)
- **FastAPI**: >=0.115.0 (async web framework)
- **Pydantic**: >=2.10.0 (data validation and IR models)
- **pytest**: >=8.3.0 (testing framework)
- **python-dotenv**: >=1.0.1 (environment variable management)
- **httpx**: >=0.28.0 (async HTTP client for Ollama)

**LLM Provider SDKs:**

- **openai**: >=1.57.0 (OpenAI native SDK)
- **anthropic**: >=0.40.0 (Anthropic native SDK)
- **No LiteLLM** - replaced with custom adapter layer for latest model support

### Frontend

- **React**: 19.1.1 (latest)
- **TypeScript**: ~5.9.3
- **Vite**: ^7.1.7 (build tool)
- **Monaco Editor**: ^4.7.0 (VS Code editor in browser)
- **Tailwind CSS**: ^4.1.14 (v4 with PostCSS, not traditional config)
- **ESLint**: ^9.36.0 with TypeScript support

**Important**: Tailwind v4 uses PostCSS instead of traditional config files. See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) if encountering PostCSS issues.

## Troubleshooting Resources

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for detailed solutions to:
- pytest-asyncio issues
- Environment variable loading
- API key errors
- Tailwind CSS v4 PostCSS configuration
- Model compatibility (LiteLLM vs direct API)
- Virtual environment activation

See [HEALTH_CHECK.md](HEALTH_CHECK.md) for health check system details.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sv-pro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sv-pro)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
