## void-chaos

> Guidelines for maintaining and updating documentation after task completion


# Documentation Maintenance Guidelines

## Directory Documentation Requirements

- **Required README Structure:**
  ```markdown
  # Directory Name
  
  Brief description of the directory's purpose and role.
  
  ## Directory Structure
  - List of files and subdirectories with descriptions
  
  ## Components/Features
  - Detailed breakdown of major components
  
  ## Implementation Details
  - Key implementation concepts
  - Design patterns used
  
  ## Usage Examples
  - Code examples showing common use cases
  
  ## Updates and Changes
  - Chronological log of major changes
  ```

## Task Completion Documentation Process

1. **Directory Assessment**
   ```typescript
   // ✅ DO: Check for existing README
   if (directoryHasReadme) {
     updateExistingReadme();
   } else {
     createNewReadme();
   }
   ```

2. **Documentation Updates**
   - Add new components or features
   - Update examples with new patterns
   - Document API changes
   - Update dependencies
   - Add migration notes if needed

3. **Change Documentation**
   ```markdown
   ## Updates and Changes
   
   ### [YYYY-MM-DD] - Task #123
   - Added new feature X
   - Modified component Y
   - Updated dependencies
   - Migration notes: ...
   ```

## Documentation Standards

### 1. Component Documentation
- Document public API
- Include type information
- Provide usage examples
- List dependencies
- Note performance considerations

### 2. Code Examples
```typescript
// ✅ DO: Include clear, focused examples
const example = {
  feature: 'New Feature',
  usage: 'Common use case',
  example: codeExample()
};

// ❌ DON'T: Leave examples outdated
const outdatedExample = {
  // Old implementation
};
```

### 3. Directory Structure
```markdown
## Directory Structure
src/
  feature/
    ├── README.md         # Updated documentation
    ├── component.ts      # New/modified component
    ├── component.test.ts # Updated tests
    └── types.ts         # Type definitions
```

## Task Completion Checklist

1. **Documentation Review**
   - [ ] Check for existing README
   - [ ] Review current documentation
   - [ ] Identify sections needing updates

2. **Content Updates**
   - [ ] Add new features/components
   - [ ] Update examples
   - [ ] Document breaking changes
   - [ ] Update dependencies list

3. **Quality Check**
   - [ ] Verify code examples
   - [ ] Check links
   - [ ] Validate formatting
   - [ ] Review for clarity

4. **Version Control**
   - [ ] Include documentation in PR
   - [ ] Reference task/issue numbers
   - [ ] Update change log

## Best Practices

1. **Documentation First**
   - Write documentation as part of task
   - Keep examples current
   - Include migration guides
   - Document breaking changes

2. **Clarity and Completeness**
   - Use clear language
   - Include all dependencies
   - Document edge cases
   - Provide troubleshooting tips

3. **Maintenance**
   - Regular reviews
   - Remove outdated content
   - Update examples
   - Check for accuracy

## Integration with Task System

1. **Task Definition**
   ```json
   {
     "task": "Implement Feature X",
     "documentation": {
       "required": true,
       "files": ["README.md"],
       "sections": ["API", "Examples"]
     }
   }
   ```

2. **Completion Requirements**
   - Documentation updates required for task completion
   - README.md must exist in affected directories
   - Examples must be verified working
   - Changes must be documented

## Automation Support

1. **README Template Generation**
   ```typescript
   // Generate new README from template
   function createReadme(directory: string) {
     const template = getTemplate();
     const content = populateTemplate(template, directory);
     writeReadme(directory, content);
   }
   ```

2. **Documentation Validation**
   ```typescript
   // Verify documentation completeness
   function validateDocs(directory: string) {
     checkRequiredSections();
     validateExamples();
     checkFormatting();
   }
   ```

## When to Update Documentation

- After adding new features
- When modifying existing functionality
- When deprecating features
- When changing dependencies
- When updating examples
- After major refactoring
- When fixing bugs that affect usage

## Documentation Review Process

1. **Pre-Commit**
   - Review documentation changes
   - Verify examples
   - Check formatting
   - Update change log

2. **Post-Merge**
   - Verify links
   - Check cross-references
   - Validate examples
   - Update indexes 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Strange-Hour)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Strange-Hour)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
