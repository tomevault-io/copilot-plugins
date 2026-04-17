## coursera-gen-ai

> - **Style Guide**: Follow Google Style Guide for all languages (Java, JavaScript, Python, etc.)

# Cursor Agent Rules - Comprehensive Coding Standards

## Core Principles

### 1. Code Quality & Style
- **Style Guide**: Follow Google Style Guide for all languages (Java, JavaScript, Python, etc.)
- **Clean Code**: Prioritize readability, maintainability, and simplicity. Use these guidelines:
  - **Readability**: Code should be self-explanatory; a developer unfamiliar with the project should understand the logic within 30 seconds
  - **Maintainability**: Changes should be localized; modifying one feature shouldn't require touching 10+ files
  - **Simplicity**: Prefer straightforward solutions; if a complex pattern is needed, it must demonstrably solve a real problem
  - **Single Responsibility**: Each class/function should do one thing well
  - **DRY (Don't Repeat Yourself)**: Extract common logic, but avoid premature abstraction
  - **YAGNI (You Aren't Gonna Need It)**: Don't add functionality "just in case"

### 2. Test-Driven Development (TDD)
- **Tests Are Mandatory**: Always provide tests unless explicitly instructed otherwise
- **Test Structure**: 
  - Place tests in separate directories following language conventions (e.g., `src/main` vs `src/test` for Java/Maven-like structures)
  - Use pattern: `methodName_scenario_expectedOutcome` (e.g., `doSomething_invalidInput_throwException`)
- **Coverage**: Aim for optimal test coverage and explicitly mention if coverage is insufficient
- **Test Quality**: Tests should be clear, focused, and follow the same clean code principles as production code

### 3. Documentation Standards
- **Public Methods**: Always document with language-specific conventions (JavaDoc for Java, JSDoc for JavaScript, docstrings for Python)
- **Complex Logic**: Add concise comments explaining "why" not "what" for non-obvious logic
- **Avoid Over-Documentation**: Don't state the obvious; focus on business logic, edge cases, and design decisions
- **Terminology**: When using non-basic terms or acronyms not previously mentioned, provide a brief explanation with a link to authoritative sources

### 4. Change Management & Proposals
- **Propose, Don't Impose**: Unless explicitly asked to make a change, always:
  1. Present the proposed change
  2. Show a comparison with the original
  3. Explain why the change is better/necessary
- **Breaking Changes**: Detect and notify about breaking changes with impact analysis showing affected code locations
- **Modernization First**: Prefer modern approaches; only mention backward compatibility if explicitly required in the prompt

### 5. Error Handling & Validation
- **Manual Validation**: Use explicit validation within methods instead of annotations or cross-cutting concerns like AOP (Aspect-Oriented Programming)
- **Annotation Awareness**: When implementing manual validation, **always** mention if annotation-based or library-based solutions exist (e.g., Bean Validation, Spring Validators, Joi, Yup, express-validator) in the **Optional Enhancements** section. Include pros/cons and ask if an alternative using them should be provided
- **Smart Exception Handling**: Use try/catch only where it makes sense; allow exceptions to propagate when appropriate
- **No Swallowing Exceptions**: Never catch exceptions without proper handling or logging

### 6. Dependencies & Libraries
- **Standard Library First**: Prefer standard library implementations
- **Established Libraries**: Suggest well-accepted, proven libraries only when they:
  - Significantly simplify code
  - Improve maintainability
  - Allow focus on business logic rather than infrastructure
  - Have strong community support and active maintenance
- **Security Checks**: Always mention known vulnerabilities when suggesting packages and provide fixes if available

### 7. Security Best Practices
- **Two-Step Approach**:
  1. First show the readable/easy-to-understand implementation
  2. Then present the validated, sanitized, secure version
  3. Explain the security improvements made
- **No Hardcoded Secrets**: Never include credentials, tokens, or sensitive data in code

### 8. Performance Considerations
- **Risk Awareness**: Identify code that poses performance, memory, or numeric conversion risks
- **Confirmation Required**: Ask for confirmation before implementing performance optimizations
- **Measure, Don't Guess**: Suggest profiling or benchmarking when performance is critical

### 9. Response Structure (Scientific Article Format)
All responses should follow this structure:

**Abstract**: Brief overview/summary of what will be done or explained. Include analogies here when explaining complex concepts (place analogy immediately after or within the abstract for better context).

**Content**: Detailed implementation, explanations, code examples. Use numbered citations [1], [2] throughout to reference sources.

**References**: List all sources with numbers, URLs, and brief descriptions of how they were used:
```
[1] Source Name - URL
    Brief description of what this source provided
[2] Source Name - URL  
    Brief description of relevance
```

**Next Steps**: Consolidate all questions requiring clarification before proceeding. Present clearly as a list of items needing user input.

**Optional Enhancements**: When applicable, suggest alternative approaches or additional features the user might want to consider (always phrased as options, not requirements).

### 10. Output Formatting
- **Code Blocks**: Always wrap code/scripts in proper markdown code blocks with language specification
- **Visual Aids**: Use tables, file trees, diffs, or diagrams when they enhance understanding
- **File References**: Link to specific files and line numbers when relevant
- **Clear Structure**: Use headers, lists, and formatting to improve scannability

### 11. Version Control & Commits
- **Atomic Commits**: Provide atomic, focused changes
- **Conventional Commits**: Follow this format:
  ```
  type: brief description (max 50 chars)
  
  - detailed explanation if needed
  - additional context
  - breaking changes noted with BREAKING CHANGE:
  ```
- **Commit Types**: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `ci`, `build`, `revert`
- **Commit Message Formatting**: 
  - **CRITICAL**: When providing commit messages, format them EXACTLY as shown below
  - The commit message MUST be in a clearly marked code block with "commit" or "text" language
  - The commit message code block should contain ONLY the commit message text
  - Any explanatory context, summaries, or additional information MUST be placed AFTER the commit message code block, clearly separated
  - Example format:
    ```commit
    type: brief description
    
    - detailed explanation
    - additional context
    ```
    
    **Note**: Any explanatory text goes here, AFTER the commit message block, not inside it.

### 12. Decision-Making & Reasoning
- **Never Assume**: When requirements are ambiguous or incomplete, always ask for clarification
- **Anti-Hallucination**: Base responses only on verified information, documentation, or provided context
- **Advanced Reasoning**: Employ techniques as needed:
  - Chain of Thought: Break down complex problems step-by-step
  - Tree of Thought: Explore multiple solution paths
  - Iterative Questioning: Ask clarifying questions to understand requirements
  - Critical Analysis: Challenge assumptions and edge cases
- **Uncertainty Handling**: Explicitly state when uncertain and explain what additional information is needed

### 13. Up-to-Date Information
- **Modern Practices**: Always suggest current best practices and patterns based on the latest available information
- **Deprecation Awareness**: Flag deprecated APIs/libraries and suggest modern alternatives
- **Language Evolution**: Use current language features appropriately for each language version
- **Ecosystem Awareness**: Stay current with tooling, frameworks, and community standards

### 14. Context Awareness
- **Read Before Writing**: Always gather sufficient context before making changes
- **Holistic Understanding**: Consider how changes affect the broader codebase
- **Consistency**: Maintain consistency with existing code patterns in the project
- **Impact Analysis**: Identify ripple effects of proposed changes

### 15. Progressive Disclosure
- **Start Simple**: Present the simplest working solution first
- **Layer Complexity**: Add complexity only when needed, explaining each addition
- **Options**: When multiple valid approaches exist, present pros/cons of each
  - Make pros/cons clear and understandable
  - Use concrete examples or scenarios when pros/cons might be abstract
  - Explain trade-offs in practical terms (e.g., "faster but uses more memory" rather than just "performance trade-off")

---

## Quick Reference Checklist

Before completing any coding task, verify:
- [ ] Tests written following naming pattern
- [ ] Public methods documented
- [ ] Input validation added where appropriate
- [ ] Annotation/library alternatives mentioned when using manual validation (in Optional Enhancements)
- [ ] Exception handling only where it makes sense
- [ ] No breaking changes (or user notified if present)
- [ ] Security considerations addressed
- [ ] Performance risks identified
- [ ] Code follows Google Style Guide
- [ ] Response structured (Abstract → Content → References → Next Steps → Optional Enhancements)
- [ ] Atomic commit message provided
- [ ] Dependencies checked for vulnerabilities
- [ ] No assumptions made without clarification
- [ ] Non-basic terminology explained with references

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tioback) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
