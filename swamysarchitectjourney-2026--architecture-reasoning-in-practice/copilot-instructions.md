## 05-primary-directives

> **Last Updated**: January 1, 2026

# Primary Directives

**Version**: 1.1  
**Last Updated**: January 1, 2026

---

## 🎯 Primary Directives

### 1. Educational Content Rules (MANDATORY)

**All content creation must follow these rules without exception.**

See: `01_educational-content-rules.mdc` for complete details.

### 2. Automation-First Approach

**Environment**: Windows 11, PowerShell  
**Note**: All commands and scripts should use PowerShell syntax. File paths use Windows format.

✅ **ALWAYS**: Create reusable PowerShell scripts for repetitive tasks  
✅ **ALWAYS**: Use existing scripts in `tools/psscripts/` first  
✅ **ALWAYS**: Enhance existing scripts rather than duplicating  
❌ **NEVER**: Execute individual commands for tasks repeated 2+ times

**Decision Tree**:

1. Check `tools/psscripts/` for existing script
2. Enhance existing script if close match
3. Create new `.ps1` for any repeated task
4. Document with examples and parameters

**Available Automation** (tools/psscripts/):

- `Get-FileStats.ps1` - File statistics analysis
- `Get-MarkdownSummary.ps1` - Markdown file analysis
- `Get-RepoStats.ps1` - Repository overview
- `Compare-DocFiles.ps1` - Compare multiple files
- `Find-DuplicateContent.ps1` - Find duplicate headings
- `Quick-HealthCheck.ps1` - Fast workspace health check

### 3. Update Verification Protocol (CRITICAL)

**After ANY structural change, IMMEDIATELY update**:

1. ✅ `copilot-instructions.md` - Repository Structure section
2. ✅ `README.md` - Repository Structure section
3. ✅ Relevant documentation files
4. ✅ Run verification: `.\Quick-HealthCheck.ps1` from tools/psscripts/

**Self-Check Question**: "Did I update the instruction file?" - If no, STOP and do it NOW.

### 4. File Naming Validation (CRITICAL)

**Before creating or committing ANY numbered file, validate naming**:

1. ✅ **Check for `00_` prefix**: **NEVER ALLOWED** - Use `01_` instead
2. ✅ **Verify scope**: Rule applies to **ALL files** including:
   - Educational content (`01_Reference/`, `02_Learning/`, `src/01_introduction/`, etc.)
   - Documentation files (`docs/`)
   - Any numbered files anywhere in repository
3. ✅ **Quick validation**: Run `Get-ChildItem -Recurse -Filter "*00_*"` to find violations
4. ✅ **NO EXCEPTIONS**: `00_` is NEVER allowed, even for meta/documentation files

**Self-Check Question**: "Does this file use `00_` prefix?" - If yes, RENAME to `01_` or appropriate number.

---
> Source: [SwamysArchitectJourney-2026/architecture-reasoning-in-practice](https://github.com/SwamysArchitectJourney-2026/architecture-reasoning-in-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
