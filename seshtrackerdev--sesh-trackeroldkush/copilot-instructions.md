## issue-documentation

> Description: Rule to ensure proper issue documentation and reference to existing fixes

---
Description: Rule to ensure proper issue documentation and reference to existing fixes
---
# 🔍 Cursor Rule: Issue Resolution Documentation

This rule establishes a systematic approach for handling and documenting technical issues encountered during development. Cursor AI **must follow this workflow** when addressing major bugs, errors, or technical challenges.

---

## 📋 Issue Resolution Process

### Step 1: Check Existing Fixes

**BEFORE** attempting to solve any major technical issue, Cursor must:

1. Check the `.docs/fixes/` directory to determine if a similar issue has been previously documented and resolved
2. Search for relevant keywords from the error message or problem description
3. Review any matching documentation to understand previous approaches

```javascript
// Example approach pseudocode
if (majorIssueDetected) {
  firstCheckDirectory('.docs/fixes/');
  searchForSimilarIssues(errorKeywords);
  if (existingFixFound) {
    suggestExistingApproach();
  } else {
    proceedToDocumentNewFix();
  }
}
```

### Step 2: Document Resolution Process

When solving a previously undocumented issue, Cursor must **create documentation** following this structure:

| Section | Content Requirements |
|---------|---------------------|
| **Issue Description** | • Clear statement of the error/problem<br>• Error messages (if applicable)<br>• Context in which the issue occurred |
| **Root Cause Analysis** | • Identification of the underlying problem<br>• Explanation of why the issue occurred<br>• Any architectural or dependency concerns |
| **Investigation Steps** | • Chronological documentation of debugging process<br>• Files examined and their relevant content<br>• Tests/validations performed |
| **Solution** | • Specific code changes made<br>• Before/after comparisons<br>• Explanation of why the solution works |
| **Guidelines** | • Recommendations to prevent similar issues<br>• Best practices derived from the fix |
| **Verification** | • Evidence that the solution resolved the issue<br>• Any additional validation steps |

### Step 3: Save Documentation

The fix documentation must be:

1. Saved in `.docs/fixes/` directory
2. Named descriptively (e.g., `resolve-circular-imports-in-dashboard-widgets.md`)
3. Formatted in Markdown for readability
4. Referenced in subsequent responses when similar issues arise

---

## ⚠️ Issue Severity Classification

Cursor should apply this documentation process for issues that meet any of the following criteria:

| Severity | Characteristics |
|----------|-----------------|
| **Critical** | • Application crashes or fails to start<br>• Security vulnerabilities<br>• Data loss or corruption |
| **Major** | • Features completely non-functional<br>• Blocking development progress<br>• Complex runtime errors<br>• Build or deployment failures |
| **Notable** | • Non-trivial bugs requiring significant debugging<br>• Issues that took multiple attempts to resolve<br>• Problems that reveal architectural weaknesses |

Simple bugs, typos, or straightforward fixes do not require detailed documentation.

---

## 📚 Example Documentation Structure

```markdown
# Fix: [Concise Issue Title]

## Issue Description
[Error message and context]

## Root Cause Analysis
[Explanation of the underlying problem]

## Investigation Steps
1. [First action taken]
2. [Observations]
3. [Next steps]
...

## Solution
[Detailed explanation with code examples]

## General Guidelines for Avoiding Similar Issues
1. [First recommendation]
2. [Second recommendation]
...

## Verification
[How the fix was tested/validated]
```

---

## 🔄 Documentation Maintenance

- When documentation in `.docs/fixes/` exceeds 20 files, suggest categorizing by issue type
- Reference relevant fixes by linking directly to the markdown file when providing solutions
- Maintain a consistent documentation style across all fix files

---

_Last updated: April 8, 2024_

---
> Source: [seshtrackerdev/sesh-trackeroldKUSH](https://github.com/seshtrackerdev/sesh-trackeroldKUSH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
