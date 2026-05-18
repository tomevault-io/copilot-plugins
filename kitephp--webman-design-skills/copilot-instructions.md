## webman-design-skills

> > Guidelines for developing and maintaining Webman Agent Skills

# Webman Skills Development Guide

> Guidelines for developing and maintaining Webman Agent Skills

---

## Skill Scope

**webman-best-practices**: Webman framework with DDD architecture, dependency injection, PER Coding Style, and clean architecture principles.

**Coverage**:
- Architecture & Dependencies (controller/service/domain/infrastructure layers)
- Naming Conventions (directories, classes, interfaces, methods)
- Code Style (PER Coding Style, type declarations, readonly properties)
- Domain Patterns (entities, value objects, aggregates, domain events)

---

## Development Workflow

### Adding New Rules

1. **Identify the problem**: What mistake do developers commonly make?
2. **Create reference file**: Add to `skills/webman-best-practices/references/`
3. **Update SKILL.md**: Add entry with problem description and reference link
4. **Follow template**: Use the reference file template below

### Reference File Template

Each reference file must include:

```markdown
# Rule Name

## Impact
High | Medium | Low

## Problem
Clear description of what goes wrong when this rule is violated.

## Why This Matters
Explanation of the consequences (maintainability, testability, performance, etc.)

## ❌ Incorrect Example

\`\`\`php
<?php

declare(strict_types=1);

// Code showing the problem
\`\`\`

## ✅ Correct Example

\`\`\`php
<?php

declare(strict_types=1);

// Code following best practice
\`\`\`

## Detection
How to identify this issue in code reviews or automated checks.

## Related Rules
- Link to related reference files
```

---

## Code Standards

All code examples must follow:

### PER Coding Style
- `declare(strict_types=1);` at the top of every file
- `final class` by default (unless designed for inheritance)
- `private readonly` for immutable properties
- Constructor property promotion for PHP 8.0+
- Type declarations for all parameters and return types
- Named arguments for better readability

### Naming Conventions
- Directories: lowercase with underscores (`value_object/`, `domain_event/`)
- Namespaces: follow directory structure (`app\domain\order\entity`)
- Classes: PascalCase (`Order`, `OrderStatus`)
- Interfaces: `Interface` suffix (`OrderRepositoryInterface`)
- Methods: camelCase (`calculateTotal()`, `markAsPaid()`)
- Constants: SCREAMING_SNAKE_CASE (`MAX_ITEMS`, `DEFAULT_STATUS`)

### Architecture Rules
- Controller → Service → Domain + Contract
- Infrastructure → Contract + Domain
- Domain NEVER depends on framework/infrastructure
- Use dependency injection, not static calls or globals

---

## Progressive Disclosure

Follow the three-tier loading model:

### Tier 1: Metadata (~100 tokens)
- `name` and `description` fields in SKILL.md frontmatter
- Loaded at startup for all skills
- Must be concise and keyword-rich

### Tier 2: Instructions (<5000 tokens recommended)
- SKILL.md body content
- Loaded when skill is activated
- Keep under 500 lines
- Use bullet points and clear structure

### Tier 3: Resources (as needed)
- Files in `references/` directory
- Loaded only when referenced
- Keep each file focused on one rule
- Use relative paths from skill root

---

## Writing Effective Rules

### DO ✅

1. **Be specific**: "Controller directly depends on Model" not "Bad architecture"
2. **Show code**: Include both incorrect and correct examples
3. **Explain why**: Don't just say "don't do X", explain the consequences
4. **Use real scenarios**: Base examples on actual Webman patterns
5. **Keep it focused**: One rule per reference file
6. **Use keywords**: Include terms that help AI match relevant tasks

### DON'T ❌

1. **Don't be vague**: Avoid "follow best practices" without specifics
2. **Don't skip examples**: Every rule needs code examples
3. **Don't nest deeply**: Keep references one level deep from SKILL.md
4. **Don't duplicate**: If two rules overlap, merge them
5. **Don't add edge cases**: Focus on common mistakes, not rare scenarios
6. **Don't include time-sensitive info**: No "as of 2024" or version-specific notes

---

## File Organization

```
skills/
└── webman-best-practices/
    ├── SKILL.md                           # Main skill file
    └── references/                        # Reference documentation
        ├── architecture/
        │   ├── controller-skip-service.md
        │   ├── domain-framework-dependency.md
        │   └── service-circular-dependency.md
        ├── naming/
        │   ├── directory-lowercase.md
        │   ├── interface-naming.md
        │   └── service-naming-pattern.md
        ├── code-style/
        │   ├── strict-types-declaration.md
        │   ├── prefer-final-classes.md
        │   └── readonly-properties.md
        └── domain/
            ├── entity-identity.md
            ├── value-object-immutability.md
            └── domain-events.md
```

---

## Validation Checklist

Before committing new rules:

### Content Quality
- [ ] Rule addresses a real, common problem
- [ ] Problem description is clear and specific
- [ ] Both incorrect and correct examples are provided
- [ ] Code examples follow PER Coding Style
- [ ] Explanation includes "why" not just "what"
- [ ] Keywords are included for AI matching

### Format Compliance
- [ ] Frontmatter is valid YAML
- [ ] `name` field matches directory name
- [ ] `description` is 1-1024 characters
- [ ] `description` includes what and when to use
- [ ] SKILL.md is under 500 lines
- [ ] References use relative paths
- [ ] No deeply nested reference chains

### Code Examples
- [ ] All examples have `declare(strict_types=1);`
- [ ] Classes are `final` unless inheritance is needed
- [ ] Properties use `private readonly` where appropriate
- [ ] Type declarations are complete
- [ ] Namespaces follow lowercase convention
- [ ] Examples are self-contained and runnable

---

## Common Pitfalls

### ❌ Too Generic
```markdown
### Architecture
- Follow clean architecture principles
```

### ✅ Specific and Actionable
```markdown
### Architecture
- Controller directly depends on Model, skipping Service layer → See [controller-skip-service](references/architecture/controller-skip-service.md)
```

---

### ❌ Missing Context
```markdown
Don't use static methods.
```

### ✅ With Explanation
```markdown
## Problem
Static methods make code hard to test and violate dependency injection principles.

## Why This Matters
- Cannot mock static methods in tests
- Creates hidden dependencies
- Breaks dependency inversion principle
```

---

### ❌ No Code Examples
```markdown
Use dependency injection instead of static calls.
```

### ✅ With Examples
```markdown
## ❌ Incorrect
\`\`\`php
public function handle(): void
{
    $user = User::find($this->userId); // Static call
}
\`\`\`

## ✅ Correct
\`\`\`php
public function __construct(
    private readonly UserRepositoryInterface $userRepository
) {}

public function handle(): void
{
    $user = $this->userRepository->findById($this->userId);
}
\`\`\`
```

---

## Maintenance

### Updating Existing Rules

1. Read the reference file completely
2. Make changes while preserving structure
3. Update examples if needed
4. Verify code examples still follow PER Coding Style
5. Update version in metadata if significant changes

### Deprecating Rules

If a rule becomes obsolete:
1. Don't delete immediately
2. Add deprecation notice at the top
3. Explain why it's deprecated
4. Point to replacement rule if applicable
5. Remove after 2 versions

---

## Version Management

Use semantic versioning in metadata:

- **Major** (1.0.0 → 2.0.0): Breaking changes, rule removals
- **Minor** (1.0.0 → 1.1.0): New rules added
- **Patch** (1.0.0 → 1.0.1): Fixes to existing rules

---

## Testing Your Skills

### Manual Testing

1. Install skill locally: `npx skills add ./webman-design-skills`
2. Create a test Webman project
3. Ask AI to create components following the rules
4. Verify AI applies the rules correctly
5. Check for false positives/negatives

### Validation

```bash
# Validate SKILL.md format (if using skills-ref)
skills-ref validate ./skills/webman-best-practices
```

---

## Contributing

### Pull Request Checklist

- [ ] New reference file follows template
- [ ] SKILL.md updated with new entry
- [ ] Code examples tested and working
- [ ] No typos or formatting issues
- [ ] Version number incremented
- [ ] CHANGELOG.md updated

### Review Criteria

Reviewers should check:
1. Is the rule addressing a real problem?
2. Are examples clear and correct?
3. Does it follow PER Coding Style?
4. Is the explanation helpful?
5. Are there any conflicts with existing rules?

---

## Resources

- [Agent Skills Specification](https://agentskills.io/specification)
- [Webman Design Guide](https://github.com/your-github/webman-design-guide)
- [PER Coding Style](https://www.php-fig.org/per/coding-style/)
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)

---

**Last Updated**: 2026-02-03
**Version**: 1.0.0

---
> Source: [kitephp/webman-design-skills](https://github.com/kitephp/webman-design-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
