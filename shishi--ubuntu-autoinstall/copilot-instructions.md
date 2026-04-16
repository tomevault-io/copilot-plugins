## ubuntu-autoinstall

> This document contains the rules and best practices for Ubuntu autoinstall configurations that I've learned through our work together.

# Ubuntu Autoinstall Configuration Rules and Guidelines

## Overview
This document contains the rules and best practices for Ubuntu autoinstall configurations that I've learned through our work together.

## System Information
- **Ubuntu Version**: 24.04 LTS
- **Installation Type**: ubuntu-desktop (NOT ubuntu-server)
- **Target System**: Desktop installation with LVM

## Critical Development Rules

### NEVER make changes without evidence:
1. **ALWAYS research and provide evidence before making any configuration changes**
2. **NEVER change settings multiple times without clear justification**
3. **ALWAYS show the source of your information (documentation, official guides, etc.)**
4. **If uncertain, ASK instead of guessing**
5. **Time is valuable - avoid trial and error approaches**

Examples of required evidence:
- Official Ubuntu/Curtin documentation links
- Error message analysis with clear cause-effect relationship
- Tested configurations with proven results
- Community consensus from reliable sources

### Changes must include:
```
## Change Justification
- **What**: Specific change being made
- **Why**: Evidence-based reason for the change
- **Source**: Documentation or error analysis supporting this change
- **Expected Result**: What this change will fix/improve
```

## Shell Script Development Standards

### Idempotency Requirements

**All scripts MUST be idempotent** - they should produce the same result whether run once or multiple times.

#### Key Idempotency Patterns:

1. **Check before create**:
```bash
if [[ ! -f "$file" ]]; then
    create_file "$file"
fi
```

2. **Check before modify**:
```bash
if ! grep -q "pattern" "$file"; then
    echo "content" >> "$file"
fi
```

3. **Use state detection**:
```bash
if systemctl is-enabled service 2>/dev/null; then
    echo "Service already enabled"
else
    systemctl enable service
fi
```

4. **Handle existing resources**:
```bash
# Check for existing recovery keys
if ls /root/.luks-recovery-key-*.txt 2>/dev/null; then
    read -p "Use existing recovery key? (y/N): " response
fi
```

### Shell Script Validation

**YOU MUST validate all shell scripts before committing:**

#### 1. Syntax Validation with bash -n
```bash
# Check syntax without executing
bash -n script.sh

# Must pass with no output (success) before proceeding
```

#### 2. ShellCheck Analysis
```bash
# Install shellcheck if not available
sudo apt-get install -y shellcheck

# Run shellcheck
shellcheck script.sh

# Fix all warnings except those explicitly documented as false positives
```

#### 3. Common Validation Pitfalls

**Invalid brace expansions**:
```bash
# ❌ WRONG - Syntax error
for i in {1..5} {7..9}; do

# ✅ CORRECT - Valid alternatives
for i in {1..5} 7 8 9; do
for i in 1 2 3 4 5 7 8 9; do
```

**Unquoted variables**:
```bash
# ❌ WRONG - Breaks with spaces
if [ $var = "value" ]; then

# ✅ CORRECT - Always quote
if [ "$var" = "value" ]; then
```

**Echo vs Printf**:
```bash
# ❌ RISKY - Can fail with special characters
echo -n "$password" | command

# ✅ SAFE - Always works correctly
printf '%s' "$password" | command
```

### Shell-Breaking Character Protection

**Recovery keys and passwords MUST be safe for shell use:**

1. **Use URL-safe Base64 for generated keys**:
```bash
# Generate safe recovery key
openssl rand -base64 48 | tr -d '\n' | tr '+/' '-_'
```

2. **Validate input**:
```bash
# Check for safe characters only
if ! [[ "$RECOVERY_KEY" =~ ^[A-Za-z0-9_=-]+$ ]]; then
    print_error "Invalid characters in recovery key"
    return 1
fi
```

3. **Use printf for safety**:
```bash
# Always use printf instead of echo -n
printf '%s' "$variable" | cryptsetup command
```

### Required Script Headers

Every script must include:
```bash
#!/usr/bin/env bash
set -euo pipefail

# Script description
# Usage: script.sh [options]
# Dependencies: list required tools
```

## Validation Requirements

### Always validate before installation:

#### Common Validation Errors

## Time-Saving Practices
1. **ALWAYS** run validation before attempting installation
2. **NEVER** trust that valid YAML means valid autoinstall config
3. **ALWAYS** check for known problematic packages (systemd-cryptenroll)
4. **USE** the pre-install-check.sh script to catch issues early

## Testing Best Practices

### Before Committing Any Script:
1. Run `bash -n script.sh` - MUST pass
2. Run `shellcheck script.sh` - MUST pass or have documented exceptions
3. Test idempotency - run twice, verify same result
4. Test edge cases - empty variables, missing files, etc.
5. Verify error handling - script should fail gracefully

### Idempotency Testing Checklist:
- [ ] First run: Creates/configures as expected
- [ ] Second run: Detects existing state, skips or asks user
- [ ] No duplicate files created
- [ ] No duplicate entries in configuration
- [ ] Proper cleanup of temporary resources
- [ ] Clear status messages about what was done/skipped

## References
- [Ubuntu Autoinstall Reference](https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html)
- [Subiquity GitHub](https://github.com/canonical/subiquity)
- [ShellCheck Wiki](https://www.shellcheck.net/wiki/)
- Bug #1969375 - systemd-cryptenroll TPM2 support issue

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shishi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
