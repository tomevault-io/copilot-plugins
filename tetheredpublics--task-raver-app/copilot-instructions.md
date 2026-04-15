## task-raver-app

> 1. **Framework Rules** (e.g., `sveltekit.mdc`)

# Cursor Rules Organization

## Rule Types

1. **Framework Rules** (e.g., `sveltekit.mdc`)
   - Framework-specific conventions and best practices
   - File structure and naming conventions
   - Framework-specific patterns and anti-patterns
   - Framework version-specific features

2. **Project Structure Rules** (e.g., `project-structure.mdc`)
   - Overall project organization
   - Directory structure
   - Key configuration files
   - Dependencies and their purposes
   - Build and deployment setup

3. **Development Workflow Rules** (e.g., `development-workflow.mdc`)
   - Development environment setup
   - Common development tasks
   - Git workflow
   - Deployment process
   - Troubleshooting guides

4. **Coding Standards Rules** (e.g., `coding-standards.mdc`)
   - Code style guidelines
   - TypeScript usage
   - Component patterns
   - Styling conventions
   - General best practices

5. **Testing Rules**
   - Unit Testing (e.g., `unit-testing.mdc`)
     - Component testing
     - Store testing
     - Test organization
     - Testing patterns
   - End-to-End Testing (e.g., `e2e-testing.mdc`)
     - User flow testing
     - API testing
     - Test environment
     - Debugging strategies

## Rule Inclusion

Rules should be included in the `.cursor/rules` directory with the following structure:

```
.cursor/rules/
├── framework/
│   └── sveltekit.mdc
├── structure/
│   └── project-structure.mdc
├── workflow/
│   └── development-workflow.mdc
├── standards/
│   └── coding-standards.mdc
└── testing/
    ├── unit-testing.mdc
    └── e2e-testing.mdc
```

## Rule File Format

Each rule file should:
1. Have a `.mdc` extension
2. Include a clear title
3. Use proper Markdown formatting
4. Include code examples where relevant
5. Reference other rules using `[Rule Name](mdc:path/to/rule.mdc)`
6. Be focused on a specific aspect of development

## Rule Dependencies

Rules can reference each other using the `mdc:` protocol:
```markdown
For more information, see [Coding Standards](mdc:standards/coding-standards.mdc)
```

## Rule Updates

- Rules should be updated when:
  - Framework versions change
  - Project structure changes
  - New patterns are established
  - Best practices evolve
  - New tools are added

## Rule Maintenance

- Keep rules up to date
- Remove outdated information
- Add new patterns and practices
- Update examples as needed
- Ensure cross-references are valid

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tetheredpublics)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tetheredpublics)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
