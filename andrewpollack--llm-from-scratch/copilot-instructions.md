## llm-from-scratch

> - **Use uv**: All package management should be done using uv


# Python Development Guidelines

## Environment and Package Management

- **Use uv**: All package management should be done using uv
  - Install dependencies: `uv sync` 
  - Run Python scripts: `make run`
  - Never use pip directly in this project

- **Virtual Environments**: Always work within the project's virtual environment
  - The project uses `.venv` directory for virtual environments

## Code Style and Quality

- **Use ruff**: All linting and formatting must be done with ruff
  - Format code: `uv run ruff format`
  - Lint code: `uv run ruff check . --fix`

- **Use mypy**: Type checking and static analysis must be done with mypy
  - Run using: `uv run mypy`

- **Type Hints**: Use Python type hints for all function parameters and return values
  ```python
  def process_data(input_data: list[float]) -> float:
      return sum(input_data) / len(input_data)
  ```

- **Docstrings**: Use Google-style docstrings for all functions, classes, and modules
  ```python
  def function_name(param1: type, param2: type) -> return_type:
      """Short description of function.
      
      Longer description if needed.
      
      Args:
          param1: Description of param1
          param2: Description of param2
          
      Returns:
          Description of return value
          
      Raises:
          ExceptionType: When and why this exception is raised
      """
  ```

## Project Structure

- **Module Organization**: Follow a clear hierarchical structure
  - Core functionality in dedicated modules
  - Tests in a separate `tests` directory paralleling the module structure

- **Imports**: Organize imports in the following order:
  1. Standard library imports
  2. Third-party library imports
  3. Local application imports
  
  Separate each group with a blank line

## Testing and Quality Assurance

- **Write Tests**: All significant functionality should have corresponding tests
- **Run Tests**: Tests should be run before committing changes

## Version Control

- **Conventional Commits**: All commit messages must follow the Conventional Commits format
  - Format: `type(scope): description`
  - Examples: `feat: add neural network implementation`, `fix(training): fix gradient calculation`
  - Valid types: `build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `style`, `test`

- **Pull Requests**: Create descriptive pull requests with appropriate context

## ML-Specific Guidelines

- **Reproducibility**: Set random seeds for reproducible experiments
  ```python
  import torch
  import numpy as np
  import random
  
  def set_seeds(seed: int = 42) -> None:
      """Set seeds for reproducibility."""
      torch.manual_seed(seed)
      torch.cuda.manual_seed_all(seed)
      np.random.seed(seed)
      random.seed(seed)
  ```

- **Model Checkpoints**: Save model checkpoints regularly during training

- **Experiment Tracking**: Document hyperparameters and results for all experiments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrewpollack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
