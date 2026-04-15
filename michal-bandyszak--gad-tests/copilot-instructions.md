## gad-tests

> - Configure project-specific rules in eslint.config.js to enforce consistent coding standards

## CODING_PRACTICES

### Guidelines for STATIC_ANALYSIS

#### ESLINT

- Configure project-specific rules in eslint.config.js to enforce consistent coding standards
- Use shareable configs like eslint-config-airbnb or eslint-config-standard as a foundation
- Implement custom rules for {{project_specific_patterns}} to maintain codebase consistency
- Configure integration with Prettier to avoid rule conflicts for code formatting
- Use the --fix flag in CI/CD pipelines to automatically correct fixable issues
- Implement staged linting with husky and lint-staged to prevent committing non-compliant code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Michal-Bandyszak)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Michal-Bandyszak)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
