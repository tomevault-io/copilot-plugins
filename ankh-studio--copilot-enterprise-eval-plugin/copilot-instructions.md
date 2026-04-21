## copilot-enterprise-eval-plugin

> Ensures all evaluation reports meet quality standards and provide actionable insights.


# Evaluation Quality Enforcement

Ensures all evaluation reports meet quality standards and provide actionable insights.

## Rule Logic

When any file matching `evals/**/*.md` is created or modified, this rule validates:

### Required Structure

- Clear evaluation metadata (date, evaluator, artifact type)
- Structured findings with evidence
- Specific scores and rationale
- Actionable recommendations

### Quality Checks

- **Evidence-Based**: All claims supported by specific evidence
- **Constructive**: Recommendations are practical and actionable
- **Consistency**: Scoring aligns with stated criteria
- **Completeness**: All required evaluation sections present

### Validation Process

1. Parse evaluation structure and metadata
2. Check for required sections and formatting
3. Validate evidence-to-claim connections
4. Verify scoring consistency
5. Identify missing or unclear elements

## Enforcement Actions

- **Pass**: Evaluation meets all quality standards
- **Warning**: Minor issues found, suggestions provided
- **Block**: Critical quality issues that must be fixed

## Example Violations

❌ **Missing Evidence**

```
## Findings
The prompt lacks clarity and could be improved.
```

✅ **Proper Format**

```
## Findings
### Clarity Issues
- **Evidence**: "The prompt uses vague terms like 'good' and 'better'"
- **Impact**: Ambiguous evaluation criteria
- **Score**: 2/4 for clarity
- **Recommendation**: Replace with specific, measurable criteria
```

This rule maintains evaluation quality and ensures consistent, actionable assessment reports.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ankh-Studio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
