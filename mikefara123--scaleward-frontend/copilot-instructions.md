## scaleward-frontend

> Rules for documenting tasks before implementation starts


# Task Documentation

## When to Document

Document tasks BEFORE starting implementation when they meet these criteria:

### **ALWAYS Document**

- **New features** that add significant functionality
- **Major refactoring** that affects multiple components/files
- **Architecture changes** that impact project structure
- **New pages or routes** being added to the application
- **API integrations** or external service implementations
- **Authentication/authorization** changes
- **Database schema changes** or major data flow modifications
- **Performance optimization** initiatives
- **Security implementations** or updates

### **NEVER Document**

- **Bug fixes** - regardless of complexity
- **Small UI tweaks** (color changes, spacing adjustments, text updates)
- **Typo corrections** in code or documentation
- **Dependency updates** unless they require code changes
- **Configuration changes** that don't affect functionality
- **Style/formatting** improvements
- **Simple component prop additions** without new functionality

## Documentation Requirements

For tasks that qualify for documentation, create a document in the `_documents/planned/` directory:

### **File Naming Convention**

```
_documents/planned/[task-name].md
```

- Use lowercase with dashes (e.g., `user-authentication-system.md`)
- Be descriptive and specific
- Avoid generic names like `feature.md` or `update.md`
- **All NEW task documents MUST be created in `_documents/planned/` folder**

### **Required Document Sections**

```markdown
# [Task Title]

## Overview

Brief description of what needs to be implemented and why.

## Requirements

- Functional requirements (what the feature should do)
- Technical requirements (performance, compatibility, etc.)
- Business requirements (user stories, acceptance criteria)

## Technical Approach

- Architecture decisions
- Key components/files to be created or modified
- Dependencies or integrations needed
- Database changes (if applicable)

## Implementation Plan

1. Step-by-step breakdown of implementation
2. Order of development (what depends on what)
3. Testing strategy
4. Deployment considerations

## Acceptance Criteria

- Clear, testable criteria for when the task is complete
- User experience expectations
- Performance benchmarks (if applicable)

## Risks & Considerations

- Potential challenges or blockers
- Alternative approaches considered
- Impact on existing functionality
```

## Documentation Workflow

1. **Before starting any qualifying task:**

   - Verify whether the task or feature is already implemented in the codebase or currently in progress, to avoid duplicate work
   - Create the documentation file
   - Fill out all required sections
   - Review with team if needed (for complex features)

2. **During implementation:**

   - Update documentation if approach changes significantly
   - Note any discoveries or changes from original plan

3. **After completion:**
   - **REQUIRED**: Add comprehensive "Implementation Results" section with:
     - ✅ Status and completion date
     - Actual file structure created (with checkmarks)
     - Dependencies added or modified
     - Key implementation decisions made
     - Deviations from original plan (if any)
     - Core features implemented checklist
     - Performance optimizations applied
     - Browser/device testing results
   - Move document to `_documents/completed/` folder
   - Update document title to indicate completion status

## File Organization

Store all task documents in:

```
_documents/
├── completed/          # Completed task documents
├── in-progress/        # Currently being implemented
└── planned/            # Documented but not started
```

Move documents between folders as status changes.

## Integration with Development

- Reference the documentation file in your initial commit message
- Include the document path in pull request descriptions
- Use the document as a checklist during code review
- Update documentation if requirements change during development

## Completion Documentation Requirements

When a task is completed, the documentation **MUST** be updated with an "Implementation Results" section that includes:

### **Required Completion Sections**

```markdown
---

## Implementation Results

### ✅ **COMPLETED - Implementation Summary**

**Implementation Date**: [Date]  
**Status**: ✅ Complete and Functional  
**Access URL**: [URL/Route if applicable]

### **Actual File Structure Created**
```

📦 [Feature Name] - IMPLEMENTED
├── [actual files created with ✅ checkmarks]
├── [list all new files]
├── [list all modified files]
└── [note any files planned but not created]

````

### **Dependencies Added**

```json
{
  "dependency-name": "version"
}
````

### **Key Implementation Decisions**

1. **Decision 1**: Explanation and reasoning
2. **Decision 2**: What was chosen and why
3. **Deviations**: Any changes from original plan

### **Core Features Implemented**

✅ **Feature 1**: Description and status  
✅ **Feature 2**: Description and status  
✅ **Feature 3**: Description and status

### **Performance Optimizations**

- List performance improvements made
- Optimization techniques used
- Benchmarks achieved

### **Browser Testing Results**

✅ **Chrome/Edge**: Status  
✅ **Firefox**: Status  
✅ **Safari**: Status  
✅ **Mobile**: Status

```

## Benefits

This documentation approach ensures:

- ✅ Clear understanding before starting work
- ✅ Better estimation and planning
- ✅ Smoother code reviews
- ✅ Knowledge transfer and team alignment
- ✅ Historical record of architectural decisions
- ✅ Reduced scope creep and requirements drift
- ✅ **Complete implementation record for future reference**
- ✅ **Actual vs. planned comparison for better future estimates**
- ✅ **Comprehensive testing and deployment documentation**
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mikefara123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mikefara123)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
