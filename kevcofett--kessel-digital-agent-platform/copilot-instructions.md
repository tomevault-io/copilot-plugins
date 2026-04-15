## kessel-digital-agent-platform

> CMAP Validation Suite - use when validating Power Platform solutions, diagnosing import failures, fixing solution XML, or running the validation pipeline


# CMAP Validation Suite

## When to Use
When validating Dataverse/Power Platform solutions, diagnosing import failures, fixing solution XML issues, or running the validation pipeline.

## ZERO WARNINGS POLICY (MANDATORY)
Validators produce ONLY pass or fail — NO WARNINGS ALLOWED.
- PASS: Meets Microsoft requirements OR is system-expected pattern
- FAIL: Does not meet requirements, must fix before import

### System-Expected Patterns (automatically PASS)
- System relationships: owner_, team_, user_, business_unit_, organization_
- Microsoft auto-generates these without publisher prefixes

## Mandatory Validation Workflow

EVERY fix MUST follow this order — NO EXCEPTIONS:

1. **MICROSOFT DOCUMENTATION** (Primary Source)
   - docs/POWER_PLATFORM_SPECIFICATIONS.md
   - learn.microsoft.com/power-apps/developer
2. **ERROR PATTERN HISTORY** (Secondary Source)
   - docs/ERROR_PATTERN_HISTORY.md
3. **APPLY FIX + DOCUMENT**
   - Update ERROR_PATTERN_HISTORY.md and validators

## Fix Verification Protocol (MANDATORY)

1. **SCAN ALL INSTANCES**: Scan ENTIRE solution for ALL instances of the pattern
2. **VERIFY VALIDATOR CATCHES PATTERN**: Run on UNFIXED solution — must detect error
3. **TEST BEFORE/AFTER**: OLD solution shows errors, NEW solution shows 0 errors

## Primary Commands

```bash
# Primary validator
python3 scripts/validators/validate_all_msft_requirements.py <solution.zip>

# Full pipeline
make validate SOL=<solution.zip>

# Auto-fix + validate
make fix SOL=<solution.zip>

# Nuclear option (all V10 fixes)
python scripts/nuclear_transform.py <input.zip> <output.zip>

# Error diagnosis
python scripts/validators/error_pattern_matcher.py "error message"
```

## Error Pattern Quick Reference

| Error | Fix Script |
|-------|-----------|
| OptionSet prefix wrong | `fix_optionset_prefix.py` |
| Solution naming mismatch | `fix_solution_naming.py` |
| System attr as custom | `fix_system_attrs.py` |
| Duplicate OptionSets | `fix_optionset_duplicates.py` |
| V10 compatibility | `nuclear_transform.py` |
| Missing displayname | Check displaynames element in entity XML |
| Missing connection ref | Add to connectionreferences in customizations.xml |

## Acceptable Publisher Prefixes
eap_, mpa_, ca_, cmap_, msdyn_, mspp_

## Current Solution
release/v7.0/solutions/CMAPPlatform_v7.12.5.zip
Status: VALID (clean sandbox import 2026-02-16)

## Documentation Files
- docs/POWER_PLATFORM_SPECIFICATIONS.md — Microsoft requirements
- docs/ERROR_PATTERN_HISTORY.md — Past errors and fixes
- scripts/validators/VALIDATOR_REGISTRY.md — Validator hierarchy

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevcofett)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kevcofett)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
