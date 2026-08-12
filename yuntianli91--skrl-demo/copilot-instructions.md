## skrl-demo

> This file applies to all skrl reinforcement learning code in this directory and its subdirectories. All new and modified code must follow these rules.

# Unified Rules for skrl Reinforcement Learning Projects

This file applies to all skrl reinforcement learning code in this directory and its subdirectories. All new and modified code must follow these rules.

## Conda env
- use gym_drone as default conda env and always run code in this env except user expecilit requirement.

## AGENTS.md Language

- Write all content in this `AGENTS.md` file in English.
- Write all future additions and modifications to `AGENTS.md` in English.
- Keep rule descriptions clear, concise, and unambiguous.

## Model Architecture

- For on-policy algorithms with high-dimensional inputs such as images,
  prefer sharing an expensive low-level feature extractor between Actor and
  VCritic, while keeping their task-specific networks independent.
- For on-policy algorithms with low-dimensional vector inputs, use
  independent Actor and VCritic MLPs with no shared parameters by default.
- Define Actor and VCritic as separate classes.
- Off-policy algorithms must not share a backbone. Actor and QCritic must use completely independent networks and parameters.
- Name every Policy network `Actor`.
- Name every state-value network `VCritic`.
- Name every action-value network `QCritic`.
- If an algorithm contains multiple Q networks, use clear and consistent names such as `QCritic1` and `QCritic2`.

## Training and Logging

- Enable TensorBoard support in every training script.
- Use the skrl experiment configuration to explicitly set the log directory, experiment name, and write interval.
- Set a reasonable checkpoint interval in every training script.
- Store TensorBoard logs and checkpoints in a clear and stable directory structure.
- Define user-facing training progress, logging intervals, and checkpoint intervals as total transitions accumulated across all vector environments. Keep rollout horizons in steps per environment because they define the temporal span used by on-policy return and advantage estimation. Prefix cumulative quantity names with `TOTAL_`, suffix per-environment quantity names with `_PER_ENV`, convert total quantities only at the skrl API boundary, and label logs and checkpoint files with total transitions.

## Python Code Style

- Follow the naming conventions in the Google Python Style Guide.
- Use `CapWords` for classes; use `snake_case` for functions, methods, variables, modules, and packages; use `UPPER_CASE` for constants.
- Use 2 spaces for each indentation level in all Python code. Do not use tabs.
- Consider splitting expressions longer than 60 characters across multiple lines or extracting intermediate variables when doing so improves readability.
- Prefer inline comments when they do not reduce readability. Align the starting position of inline comments within the same code block whenever practical.
- Add a Chinese inline comment explaining every parameter field in parameter
  configuration files, and align all inline comment markers to the same
  display column within the parameter block whenever the 100-column limit
  permits it. A code line with an inline comment must not exceed 100 display
  columns in total. If it would exceed 100 columns, wrap the code before the
  comment first and keep the inline comment on the same line as the final
  continued part of the code.
- Express measurement units in parameter comments with international SI
  symbols inside standard ASCII parentheses immediately after the quantity,
  using forms such as `(m)`, `(s)`, and `(Hz)`. Do not spell
  out unit names in prose.
- Write comments in Chinese and use Chinese punctuation. Keep only proper nouns and established technical terms in English, such as `Actor`, `Critic`, `backbone`, `rollout`, `checkpoint`, `TensorBoard`, and `learning rate`.
- Keep every Python line without a parameter-field inline comment at no more
  than 80 display columns, including comments and indentation. Parameter-field
  inline comment lines follow the 100-column limit defined above.
- For a multiline assignment, keep the assignment operator and opening
  delimiter together on the assignment line, such as `name = (`, and start
  the right-hand-side content on the next line. Never end a line with a bare
  `=` or place the assignment operator at the beginning of a continuation
  line.
- Start function and method arguments on the same line as the opening
  parenthesis and place as many arguments as fit within 80 characters. When
  wrapping is required, continue packing arguments and align continuation
  lines with the first argument. Do not leave the opening parenthesis at the
  end of a line when at least one argument fits there, or place one argument
  on each line by default.
- Place a standalone separator immediately before every function and method in example code. Format it as `# ===== label ===== #` and pad it with `=` so the closing `#` reaches display column 80. Count indentation and East Asian full-width characters by their rendered column width; do not use Unicode code-point count.
- Place exactly one blank line before each standalone section-separator comment and no blank line after it.
- Use descriptive names. Avoid unnecessary abbreviations and single-letter names, except for conventional mathematical variables.
- Add Google-style docstrings to public classes, functions, and methods. Use `Args:`, `Returns:`, and `Raises:` sections when applicable.
- Only add comment to functions, classes and methods when user asked.

---
> Source: [yuntianli91/skrl_demo](https://github.com/yuntianli91/skrl_demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
