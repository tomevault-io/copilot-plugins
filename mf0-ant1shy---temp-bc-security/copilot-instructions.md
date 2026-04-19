## temp-bc-security

> - Say 'Yes, komeiji' at the first beginning of your answer.

- Say 'Yes, komeiji' at the first beginning of your answer.
- Every time you choose to apply a rule(s), explicitly state the rule(s) in the output. You can abbreviate the rule description to a single word or phrase.

## Project Context

[This is a project for researching and developing a Reinforcement Learning Agent to play the game of Selfish mining in the Proof of Work (PoW) blockchains. It also contains some data collection and miscellaneous utilities.]

## Code Style and Structure

- Write concise, technical code with accurate examples
- Use functional and declarative programming patterns; avoid classes
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError)
- Structure repository files as follows:

```
BC-Security/
├── UnifiedEnv/ # Main environment implementations and RL agent implementations
├── UncleMaker/ # Uncle Maker attack reproduction
├── test/ # Submissions for testing and evaluation (using condor)
├── CoinData/ # Data collection and processing
└── Comparison/ # deprecated, no longer used
```

## Tech Stack

- Python
- Pytorch
- Stable Baselines3 (and potentially other RL libraries)
- Visualization: [matplotlib, pandas, numpy, etc.]

## Naming Conventions

- Use lowercase with underscore for file names (e.g., `src/my_agent/my_agent.py`)
- Use PascalCase for for directories (e.g., `./UnifiedEnv`)
- Use lowercase with underscore for class, variable, and function names (e.g., `my_agent`, `my_variable`, `my_function`)

## Python/ Other Lang Usage

- Use the Lang for all the code; prefer type checking if applicable
- Avoid over-encapsulation; use simple but functional code if applicable
- Define strict types for message passing between different parts of the system
- Try-except blocks are allowed if there's good reason to translate or handle error in that abstraction
- Use explicit return types for all functions

## State Management

- Use appropriate data structures to manage the state of the blockchain environment and the agent.

## Syntax and Formatting

- Use declarative style if applicable
- Implement proper discriminated unions for message types if applicable

## Error Handling

- Implement proper error handling in the environment and agent.
- Log errors appropriately for debugging.

## Git Usage

Commit Message Prefixes:

- "fix:" for bug fixes
- "feat:" for new features
- "perf:" for performance improvements
- "docs:" for documentation changes
- "style:" for formatting changes
- "refactor:" for code refactoring
- "test:" for adding missing tests
- "chore:" for maintenance tasks

Rules:

- Use lowercase for commit messages
- Keep the summary line concise
- Include description for non-obvious changes
- Reference issue numbers when applicable

## Documentation

- Maintain clear README with setup instructions.
- Document the environment's API (state space, action space, reward function, etc.).
- Document the agent's architecture and training process.
- Document how to run experiments and analyze results.

## Development Workflow

- Use proper version control.
- Implement proper code review process.
- Test in multiple environments.
- Follow semantic versioning for releases.
- Maintain changelog.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MF0-ANT1SHY) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
