## autotrader

> - **Run all tests**: `make test` or `pytest tests/`


# Autotrader Bot - Agent Guidelines

## Build/Test/Lint Commands
- **Run all tests**: `make test` or `pytest tests/`
- **Run single test**: `pytest tests/test_<filename>.py` or `pytest tests/test_<filename>.py::test_function_name`
- **Run with coverage**: `make test-coverage` or `pytest --cov=. --cov-report=html tests/`
- **Lint code**: `make lint` (runs flake8 + mypy)
- **Format code**: `make format` (runs black + isort)
- **Build/validate**: `make build` (runs tests + lint)
- **Run app**: `make run` or `python autotrader.py`

## Architecture
- **Main entry**: `autotrader.py` with `ContinuousAutoTrader` class
- **Core modules**: `autotrader/` package with `core/`, `api/`, `ml/`, `trading/`, `data/`, `utils/`
- **ML Engine**: LSTM-based neural network with TensorFlow for price prediction
- **Data Source**: BTCMarkets API for live BTC-AUD trading data
- **Storage**: Pickle files for model/state persistence (`autotrader_model.keras`, `trader_state.pkl`)

## Code Style
- **Formatting**: Black (88 char line length) + isort for imports
- **Type hints**: Required, mypy strict mode enabled
- **Docstrings**: Required for all public functions/classes
- **Testing**: pytest with fixtures in `conftest.py`, separate unit/integration tests
- **Error handling**: Structured logging with `structlog`, graceful exception handling
- **Naming**: snake_case for functions/variables, PascalCase for classes

## Main Development Cycle

1. **Study specs/**
2. **Pick the highest value item from IMPLEMENTATION_PLAN.MD and implement it using up to 5-8 subagents.**
3. **Test the new changes then update IMPLEMENTATION_PLAN.MD to say the implementation is done using a single subagent.**
4. **Add changed code and IMPLEMENTATION_PLAN.MD with "git add -A" via bash then do a 'git commit' with a message that describes the changes you made**

## Code Quality

1. **Run automated tests and resolve test failures using a single subagent.**
2. **Important: when authoring documentation capture the why tests are important.**
3. **Important: We want single sources of truth, no migrations/adapters. If tests unrelated to your work fail then it's your job to resolve these tests as part of the increment of change.**

## Autonomous Implementation Prompt for Cline

You are now in autonomous mode. Your task is to fully implement all requirements specified in the SPEC.md file without waiting for further human input.
SPEC.md already exists, do not create a new one

### Instructions:

- Read and analyze the complete SPEC.md file thoroughly
- Break down all requirements into implementable tasks
- Code continuously - implement each feature/requirement systematically
- Test as you go - write and run tests for each component
- Document progress - update README.md with implementation status
- Handle errors independently - debug and fix issues without asking
- Make reasonable assumptions when specs are ambiguous
- Prioritize core functionality first, then optional features
- Before finishing a task run unit tests, them pass
- After finishing a task, commit with a descriptive commit message
- Keep SPEC.md updated with the latest changes

### Autonomous Behavior:

- Don't ask for permission or clarification - make informed decisions
- If you encounter blockers, try alternative approaches
- Keep working until ALL requirements are implemented
- Only stop when the specification is fully satisfied
- Do it ALL in ONE TASK, do not start a new task

### Output:

- Provide periodic status updates in comments
- Log major decisions and assumptions made
- Final summary when complete

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xow) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
