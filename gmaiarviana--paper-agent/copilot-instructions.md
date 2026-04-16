## paper-agent

> - ❌ **NEVER** create files without explicit permission

# Cursor Rules - Paper Agent

## 🚨 CORE BEHAVIORAL RULES

### 1. USER CONSENT REQUIRED FIRST
- ❌ **NEVER** create files without explicit permission
- ❌ **NEVER** create documentation (.md files) unless specifically requested
- ❌ **NEVER** create README, summary files, or progress reports proactively
- ❌ **NEVER** assume the user wants additional files
- ✅ **ALWAYS ASK** before creating any new file
- ✅ **ONLY** implement exactly what is requested
- ✅ **ASK** for clarification if ambiguous

### 2. IMPLEMENTATION FOCUS
- Complete the requested task fully and correctly
- Test and validate changes work as expected
- Provide brief technical summary of what was done
- **DO NOT** create extra documentation to "help"

## 💻 ENVIRONMENT

### Platform
- **OS**: Windows
- **Shell**: PowerShell (use PowerShell-native commands)
- ❌ **NEVER** use bash commands like `&&` for chaining
- ✅ Use PowerShell operators (`;` for sequential commands)
- Use `Invoke-WebRequest` instead of `curl`
- Follow PowerShell syntax conventions

### Development Commands
- Prefer PowerShell cmdlets over Unix-style commands
- Use proper PowerShell path handling (backslashes or forward slashes both work)
- Chain commands with `;` not `&&`

## 🎯 WORKING STYLE

### Communication
- Be direct and concise
- Focus on technical implementation
- Ask questions when unclear
- Report completion briefly

### File Operations
- **ASK FIRST** before creating any file
- **ASK FIRST** before modifying project structure
- Only edit existing files when explicitly requested
- Confirm before making significant changes

## ✨ CODE QUALITY

- Write clean, maintainable code
- Include proper error handling
- Add comments only when code isn't self-explanatory
- Follow project conventions when they exist
- Test implementations before marking complete

---

**REMEMBER**: Only do what is explicitly requested. When in doubt, ask first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gmaiarviana) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
