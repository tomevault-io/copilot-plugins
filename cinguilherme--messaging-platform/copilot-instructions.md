## messaging-platform

> - **Model Name:** Gemini 3.5 Flash

# Gemini Model Card

## Model Details

- **Model Name:** Gemini 3.5 Flash
- **Model Version:** gemini-3-flash-preview
- **Model Description:** cost-efficient model with high performance.
- **Model Owner:** Google

## Intended Use

This model is intended for use in the following scenarios:

- **Primary Use Case:** [Describe the primary use case of the model in this project, e.g., "Summarizing text", "Generating code", "Answering questions"]
- **Target Users:** [Describe the target users of the model, e.g., "Developers", "Content creators"]
- **Secondary Use Cases:** [Describe any secondary use cases]

## Core Roles and Rules

### Coding assistant

- You are a coding assistant.
- You are a helpful assistant that can help with coding tasks.
- Modularization is important.
- Behaviour validation via tests is paramount.
- As much as possible logic should be modeled as pure functions and unit tested.
- Integration tests are important. Eg. run integration tests with INTEGRATION=1 make tests


### General Coding Rules

- Modularization is extremely important.
- Avoid too much nesting, extract functions to avoid too much nesting. If more than just one let binding is needed, extract pieces into dedicated internal functions.
- Use pure functions whenever possible. They should be easy to unit test. 
- Always test in between changes to avoid regressions. "make tests" or clojure -M:test -n "target test name" to run a specific test.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cinguilherme)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cinguilherme)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
