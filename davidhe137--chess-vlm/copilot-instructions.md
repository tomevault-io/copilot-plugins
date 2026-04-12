## chess-vlm

> This is a research project evaluating Vision-Language Models (VLMs) vs Language Models (LLMs) on chess puzzle solving to test spatial reasoning capabilities. The core hypothesis is that VLMs should outperform LLMs on chess puzzles due to better spatial inductive biases.

# Chess VLM Research Project - Cursor AI Rules

## Project Context
This is a research project evaluating Vision-Language Models (VLMs) vs Language Models (LLMs) on chess puzzle solving to test spatial reasoning capabilities. The core hypothesis is that VLMs should outperform LLMs on chess puzzles due to better spatial inductive biases.

## Research Goals
- Compare VLM vs LLM performance on chess puzzles from Lichess dataset
- Evaluate spatial reasoning through chess tactics (forks, pins, skewers, etc.)
- Test across difficulty levels (1200-2400 rating) with stratified sampling
- Analyze first-move accuracy and full-sequence solving

## Chess Domain Knowledge
- FEN notation: Standard chess position representation (e.g., "rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1")
- Move formats: Algebraic (Nf3, Qxe4+), UCI (e2e4, g1f3), natural language
- Puzzle structure: Position + solution sequence + difficulty rating + themes
- Key metrics: First move accuracy, full sequence accuracy, stratified by rating/theme

## Coding Standards
- Use type hints for all function parameters and return values
- Write concise docstrings for all public functions and classes
- Prefer dataclasses or Pydantic models for structured data
- Include error handling for API calls and model responses
- Log important steps and results for reproducibility

## Preferred Libraries & Patterns
- `datasets` for Hugging Face dataset loading
- `python-chess` for move validation and board manipulation
- `chessboard-image` for generating board visualizations
- `openrouter` for unified access to multiple closed-source models (GPT, Claude, Gemini)
- `vllm` for open-source model inference
- `pandas` for results analysis and stratification
- `matplotlib`/`seaborn` for visualization

## Model Evaluation Patterns
- Parse multiple move formats robustly (algebraic, UCI, natural language)
- Handle model response variations gracefully
- Cache expensive operations (dataset loading, image generation)

## File Organization Preferences
- Keep data loading separate from model evaluation
- Modularize prompt templates and response parsing
- Separate visualization code from core evaluation
- Use configuration files for model parameters and API keys
- Structure experiments as reproducible scripts with clear outputs

## Research Best Practices
- Always include random seeds for reproducibility
- Save intermediate results for analysis
- Document experimental choices and hyperparameters
- Include baseline comparisons and ablation studies

## Code Quality
- Prefer explicit over implicit behavior
- Use meaningful variable names (puzzle_rating vs rating)
- Break complex functions into smaller, testable units
- Include input validation for critical functions
- Use enums for categorical variables (PuzzleTheme, ModelType)
- Add progress bars for long-running evaluations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DavidHe137)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DavidHe137)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
