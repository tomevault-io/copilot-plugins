## 25su-ccny-eas-ml-group

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT REMINDERS

**ALWAYS follow these critical instructions:**

- **GIT COMMIT after every nontrivial edit** - Never forget to commit and push changes
- **Use `mamba` and the environment `25su-ml-group`** for all code execution and package management
- **Inspect every plot you generate** - Check for bugs and interpret scientifically
- **Verify your work thoroughly** - Don't just check if files exist, examine their contents

## Repository Overview

This is a summer 2025 CCNY EAS machine learning working group repository focused on learning PyTorch and applying ML models to earth science problems. The emphasis is on understanding both ML concepts and practical implementation.

## Environment Setup

The repository uses conda environments with PyTorch.  The only nonstandard thing is that pytorch needs numpy<2.

- The project's conda environment name is 25su-ml-group. Always use that environment whenever running any code or installing or updating packages.

## Repository Structure

- `docs/`: Meeting notes, ML concept guides, and example environment files
- `projects/`: Individual member projects with specific ML implementations
  - `michelle/`: Hadley cell edge detection using feedforward neural networks
  - `nielsen-handwriting-nn/`: MNIST digit recognition implementation
  - `spencer/`: NYC precipitation ML model

## Key Concepts

The repository emphasizes understanding:
- Supervised vs unsupervised learning
- Train/validate/test splits and cross-validation
- Data normalization and standardization
- Neural network components (neurons, layers, weights, biases)
- Gradient descent and backpropagation
- Overfitting prevention strategies

## Coding Style Guidelines

For Spencer's project and similar pedagogical implementations:

**Structure:**
- Single script for initial implementations (avoid premature modularization)
- Encapsulate functionality into clear functions
- Use classes only when necessary
- Keep it simple - avoid scope creep and "while we're at it" additions
- Implement precisely what's specified, nothing more, nothing less

**Style:**
- Follow standard Python naming conventions (snake_case)
- Use Black formatting
- Minimal but clear docstrings
- Type hints where helpful
- One function = one clear purpose
- Wrap main execution in `if __name__ == "__main__":`

**Imports:**
- Import only what you need
- No unnecessary dependencies

**Variable Naming:**
- Always use descriptive variable names.

**Logging:**
- Use proper logging rather than just print statements, including saving logs to disk.

## Common Workflow

1. Load and explore data in Jupyter notebooks
2. Implement preprocessing scripts
3. Create custom Dataset classes
4. Define model architecture
5. Train with proper validation
6. Evaluate on test set

## Version Control Best Practices

- Git commit after every nontrivial edit. Commit messages should follow best practices:
  - Short summary line ~<80 chars
  - Break with more descriptive summary if needed
  - Additional info in a footer if further needed
  - Upon each git commit, push to remote

## Code Verification Guidelines

- Be careful and thorough in checking your work.  For example, if you are tasked with writing output to a file on disk, it is insufficient to merely check that the file now exists.  You need to then carefully inspect the file contents and verify that they match what was intended.

## Visualization Guidelines

- Whenever you generate a new plot, *always* inspect that plot, evaluating it for (1) potential signs of bugs or other problems, and then (2) to interpret it scientifically.

## Pedagogical Code Writing Guidelines

- Assume that every line of code you write will be used by students new to ML, and some of them pretty new to programming entirely.
  - Strive for clarity and simplicity
  - Include a bit more commenting than you'd typically include
  - Balance between being straightforward and elegant
  - Remember that code can be both clear and elegant
  - When clarity and elegance are in tension, make reasonable judgments or seek input

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spencerahill) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
