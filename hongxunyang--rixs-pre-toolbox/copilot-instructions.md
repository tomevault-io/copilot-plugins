## rixs-pre-toolbox

> Guidelines for continuously improving Cursor rules based on emerging code patterns and best practices.

## Rule Improvement Triggers

- New code patterns not covered by existing rules
- Repeated similar implementations across files
- Common error patterns that could be prevented
- New libraries or tools being used consistently
- Emerging best practices in the codebase

# Analysis Process:
- Compare new code with existing rules
- Identify patterns that should be standardized
- Look for references to external documentation
- Check for consistent error handling patterns
- Monitor test patterns and coverage

# Rule Updates:

- **Add New Rules When:**
  - A new technology/pattern is used in 3+ files
  - Common bugs could be prevented by a rule
  - Code reviews repeatedly mention the same feedback
  - New security or performance patterns emerge

- **Modify Existing Rules When:**
  - Better examples exist in the codebase
  - Additional edge cases are discovered
  - Related rules have been updated
  - Implementation details have changed


- **Rule Quality Checks:**
- Rules should be actionable and specific
- Examples should come from actual code
- References should be up to date
- Patterns should be consistently enforced

## Continuous Improvement:

- Monitor code review comments
- Track common development questions
- Update rules after major refactors
- Add links to relevant documentation
- Cross-reference related rules

## Rule Deprecation

- Mark outdated patterns as deprecated
- Remove rules that no longer apply
- Update references to deprecated rules
- Document migration paths for old patterns

## Documentation Updates:

- Keep examples synchronized with code
- Update references to external docs
- Maintain links between related rules
- Document breaking changes

Follow [cursor-rules.mdc](mdc:.cursor/rules/curs or-rules.mdc) for proper rule formatting and structure.

## Project Logic Maintenance

### Update [project-logic.mdc](mdc:.cursor/rules/project-logic.mdc) When:

- **Core Architecture Changes:**
  - New classes or major refactoring in [report.py](mdc:report_generator/report.py)
  - Configuration system modifications in [config.py](mdc:report_generator/config.py) or [config.yml](mdc:report_generator/config.yml)
  - Template system changes or new templates added

- **Feature Additions:**
  - New report generation capabilities
  - Additional content processing features  
  - New navigation or metadata systems
  - Enhanced security or authentication features

- **Workflow Changes:**
  - Modified initialization process
  - Updated content addition patterns
  - Changed generation or output procedures
  - New data management approaches

- **Integration Updates:**
  - New dependencies or libraries
  - Modified external integrations (MathJax, markdown, etc.)
  - Changed file format support
  - Updated API or interface patterns

### Maintenance Checklist:
- [ ] Update architecture flow when core processes change
- [ ] Document new features with usage examples
- [ ] Revise file organization patterns
- [ ] Update development patterns and error handling
- [ ] Cross-reference with [project-structure.mdc](mdc:.cursor/rules/project-structure.mdc) changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HongXunyang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HongXunyang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
