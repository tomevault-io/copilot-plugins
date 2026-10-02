## llm

> This file provides guidelines for AI agents working in this repository.

# AGENTS.md - Agentic Coding Guidelines

This file provides guidelines for AI agents working in this repository.

## Project Overview

This is a Python machine learning/deep learning repository containing:
- Reinforcement learning algorithms (DDPG, Actor-Critic)
- Transformer implementations
- Monte Carlo methods
- Physics-based AC modeling

Dependencies: `torch`, `numpy`, `scipy`, `matplotlib`

---

## Commands

### Running Code
```bash
# Run a Python script
python <filepath>

# Run a module
python -m transformer.train_from_base

# Run with Jupyter
jupyter notebook attention.ipynb
```

### Testing
This project does not have a formal test framework. Tests are embedded in modules using `if __name__ == "__main__":` blocks.

```bash
# Run module tests
python monte_carlo_method.py
python ddpg_algorithm.py
python transformer/mini_transformer.py
python test_func.py
```

To run a specific test function:
```bash
python -c "from monte_carlo_method import test_monte_carlo_estimation; test_monte_carlo_estimation()"
```

### Linting & Type Checking (Recommended)
Install tools:
```bash
pip install ruff mypy black
```

```bash
# Lint with ruff
ruff check .

# Format with black
black .

# Type check with mypy
mypy .
```

---

## Code Style Guidelines

### Imports
- Standard library imports first
- Third-party imports second
- Group by: `stdlib`, `third-party`, `local`
- Use explicit imports (avoid `from x import *`)
- Example:
  ```python
  import math
  from collections import deque
  
  import torch
  import numpy as np
  
  from transformer.mini_transformer import Transformer
  ```

### Formatting
- Use **Black** for code formatting (line length: 88)
- Use **isort** for import sorting
- 4 spaces for indentation (no tabs)
- Add trailing commas in multi-line constructs

### Type Hints
- Use type hints for function parameters and return values
- Follow `mini_transformer.py` as reference for PyTorch typing
- Example:
  ```python
  def forward(self, x: torch.Tensor) -> torch.Tensor:
  def build_transformer(
      src_vocab_size: int,
      tgt_vocab_size: int,
      d_model: int = 512,
  ) -> Transformer:
  ```

### Naming Conventions
- **Classes**: PascalCase (e.g., `Actor`, `Critic`, `Transformer`)
- **Functions/methods**: snake_case (e.g., `loss_function`, `select_action`)
- **Variables**: snake_case (e.g., `max_action`, `replay_buffer`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_BUFFER_SIZE`)
- **Private methods**: prefix with underscore (e.g., `_compute_loss`)

### Error Handling
- Use explicit exceptions rather than silent failures
- Validate inputs at function boundaries
- Example:
  ```python
  def train(self, batch_size: int):
      if batch_size <= 0:
          raise ValueError(f"batch_size must be positive, got {batch_size}")
      if self.replay_buffer.size() < batch_size:
          return  # Early return is acceptable
  ```

### Documentation
- Use Google-style or NumPy-style docstrings for public functions
- Include: description, args, returns, raises
- Example:
  ```python
  def monte_carlo_estimation(func, a, b, num_samples=10000):
      """Estimate integral using Monte Carlo method.
      
      Args:
          func: Function to integrate
          a: Lower bound
          b: Upper bound
          num_samples: Number of samples (default: 10000)
      
      Returns:
          Estimated integral value
      """
  ```

### PyTorch Conventions
- Use `super().__init__()` without arguments: `super().__init__()`
- Initialize weights properly (see `build_transformer` in `mini_transformer.py`)
- Use `torch.nn.Module` for neural network layers
- Use `@staticmethod` for pure tensor operations

### General Best Practices
- Keep functions focused (single responsibility)
- Avoid global variables where possible
- Use meaningful variable names
- Remove commented-out code before submitting
- No magic numbers - use named constants

---

## Project Structure

```
.
├── ac_model.py           # AC optimization model
├── ddpg_algorithm.py     # DDPG reinforcement learning
├── monte_carlo_method.py # Monte Carlo methods
├── tft_model.py          # TFT model
├── test_func.py          # Simple test functions
├── transformer/          # Transformer implementation
│   ├── mini_transformer.py
│   ├── train_from_base.py
│   └── ...
└── *.md                  # Documentation
```

---

## Common Tasks

### Running a Transformer training
```bash
python transformer/train_from_base.py
```

### Running RL algorithm
```bash
python ddpg_algorithm.py
```

### Running Monte Carlo experiments
```bash
python monte_carlo_method.py
```

---
> Source: [24Jay/LLM](https://github.com/24Jay/LLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
