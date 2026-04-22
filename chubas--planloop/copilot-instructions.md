## planloop

> Planloop is a spec-driven AI development framework where all feature planning, implementation, and validation are grounded in structured documentation and specifications. Every change to the codebase should be aligned with existing specifications or accompanied by updated specifications.

# Planloop Spec-Driven Development Framework

## Overview
Planloop is a spec-driven AI development framework where all feature planning, implementation, and validation are grounded in structured documentation and specifications. Every change to the codebase should be aligned with existing specifications or accompanied by updated specifications.

## Main Files and Directories
- `.spec/`: Contains all project specifications
  - `.spec/features/`: Feature-specific specifications
  - `.spec/api/`: API documentation
- `.cursor/`: Contains AI agent configuration
  - `.cursor/rules/`: Cursor rules for enforcing the workflow
  - `.cursor/memory/`: Agent memory for planning and context
- `scripts/`: Utility scripts
  - `scripts/spec_bootstrap.js`: Bootstrap script for initializing projects
  - `scripts/spec_check_score.js`: Script for validating spec alignment
- `spec.config.json`: Configuration for the framework

## Validation Process

Before implementing any feature or making significant changes:

1. Check if a specification exists in `.spec/`
2. Read and understand the specification
3. Implement according to the specification
4. Run spec validation to ensure alignment
5. If needed, update the specification alongside the code

## Validation Prompt

When validating changes, answer the following:

1. What changes are being made to the codebase? (`@diff`)
2. Which specifications are relevant to these changes? (Check `.spec/` directory)
3. Do the changes align with existing specifications?
4. Calculate an alignment score:
   - 0-30%: Major divergence from specifications
   - 31-60%: Partial alignment, may need spec updates
   - 61-80%: Good alignment with room for improvement
   - 81-100%: Excellent alignment with specifications

Provide a structured response:

```
## Change Analysis
[Describe the changes being made]

## Relevant Specifications
[List the specifications that apply]

## Alignment Assessment
[Evaluate how well the changes align with specifications]

## Recommendations
[Suggest improvements to code or specs]

## Overall Score: [X]%
```

## Actions Required

If alignment score is below 70%:
- Update specifications to match changes
- Modify changes to better align with specifications
- Document why divergence is necessary

Remember: Specifications and code should evolve together. Neither is fixed - both should be updated as the project progresses.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chubas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
