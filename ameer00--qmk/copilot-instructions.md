## qmk

> This document outlines the complete workflow for updating and deploying QMK keymaps for the Ferris Sweep keyboard.

# QMK Ferris Sweep Keymap Development Workflow

This document outlines the complete workflow for updating and deploying QMK keymaps for the Ferris Sweep keyboard.

## Required Steps for Keymap Updates

When updating the keymap, **ALL** of the following steps must be performed in order:

### 1. Edit the Keymap
- Modify `keymaps/qwerty2/keymap.c` with the required changes
- Follow existing patterns for tap dance, custom keycodes, etc.
- Ensure proper syntax and QMK conventions

### 2. Compile the Firmware
```bash
qmk compile -kb ferris/sweep -km qwerty2
```
- Verify compilation succeeds without errors
- Check firmware size is within limits (should be < 28,672 bytes)
- Note any warnings or issues

### 3. Flash the Firmware
```bash
qmk flash -kb ferris/sweep -km qwerty2
```
- Put keyboard in bootloader mode when prompted
- Verify successful flash and validation
- Test basic functionality

### 4. Generate JSON Keymap
```bash
qmk c2json -kb ferris/sweep -km qwerty2 keymaps/qwerty2/keymap.c > keymaps/qwerty2/keymap.json
```
- This creates/overwrites the JSON representation
- Required for QMK Configurator compatibility
- Ensures C and JSON keymaps stay synchronized

### 5. Generate Visual Layout Documentation
```bash
qmk info -kb ferris/sweep -km qwerty2 -l --ascii > keymap_layout.txt
```
- Creates/updates the text-based visual layout documentation
- Provides ASCII art representation of all layers
- Includes comprehensive key legends and explanations
- Essential for printable keyboard reference

### 6. Commit and Push All Changes
```bash
# Stage all modified and new files
git add keymaps/qwerty2/keymap.c keymaps/qwerty2/keymap.json keymap_layout.txt

# Commit with descriptive message
git commit -m "$(cat <<'EOF'
[Brief description of changes]

[Detailed explanation of what was modified]

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

# Push to GitHub main branch
git push origin main
```

## Current Keymap Features

### Tap Dance Keys (Layer 0)
- **Q** → hold: ! (exclamation)
- **W** → hold: @ (at)
- **E** → hold: # (hash)
- **R** → hold: $ (dollar)
- **T** → hold: % (percent)
- **Y** → hold: ^ (caret)
- **U** → hold: & (ampersand)
- **I** → hold: * (asterisk)
- **O** → hold: ( (left parenthesis)
- **P** → hold: ) (right parenthesis)
- **S** → hold: ~ (tilde)
- **H** → hold: - (minus)
- **L** → hold: ? (question mark)
- **M** → hold: / (slash)
- **,** → hold: : (colon)
- **.** → hold: " (double quote)

### Special Keys
- **BOOT_HOLD**: Hold for 5 seconds to enter bootloader mode

### Layer Structure
- **Layer 0**: Main QWERTY with tap dance
- **Layer 1**: Numbers, navigation, symbols
- **Layer 2**: Function keys, mouse controls

## Important Notes

- **Never skip any step** - all steps are required for proper deployment
- **Always test** the keyboard after flashing before committing
- **Maintain JSON sync** - the JSON file must be regenerated after any C changes
- **Use descriptive commit messages** that explain what functionality was added/changed
- **Follow QMK best practices** for code style and conventions

## Troubleshooting

### Compilation Errors
- Check syntax in keymap.c
- Verify all custom keycodes are properly defined
- Ensure tap dance arrays match enum definitions

### Flash Issues
- Ensure keyboard is in bootloader mode
- Check USB connection
- Try different USB port or cable

### Functionality Issues
- Verify tap dance timing settings
- Check layer definitions
- Test with QMK's built-in debugging tools

## File Structure
```
keymaps/qwerty2/
├── keymap.c          # Main C implementation
├── keymap.json       # JSON representation (auto-generated)
└── rules.mk          # Build configuration

Root directory:
├── CLAUDE.md         # Primary workflow documentation
├── GEMINI.md         # Synchronized copy of workflow documentation
└── keymap_layout.txt # Visual layout documentation (auto-generated)
```

## Complete Workflow Summary

**MANDATORY STEPS** - All must be completed for every keymap update:

1. **Edit** → `keymaps/qwerty2/keymap.c`
2. **Compile** → `qmk compile -kb ferris/sweep -km qwerty2`
3. **Flash** → `qmk flash -kb ferris/sweep -km qwerty2`
4. **Generate JSON** → `qmk c2json ... > keymap.json`
5. **Generate Layout** → `qmk info ... > keymap_layout.txt`
6. **Commit & Push** → `git add`, `git commit`, `git push origin main`

**Files that must be updated/committed:**
- `keymaps/qwerty2/keymap.c` (manual edit)
- `keymaps/qwerty2/keymap.json` (auto-generated)
- `keymap_layout.txt` (auto-generated)

**Never skip the visual layout generation** - it provides essential documentation for understanding the keyboard layout without parsing code.

## Documentation Synchronization

**IMPORTANT**: Whenever `CLAUDE.md` is modified, the following additional steps must be performed:

### 7. Sync Documentation Files
```bash
# Copy CLAUDE.md to GEMINI.md to keep both files synchronized
cp CLAUDE.md GEMINI.md

# Stage both documentation files
git add CLAUDE.md GEMINI.md

# Commit documentation changes
git commit -m "$(cat <<'EOF'
Update workflow documentation

[Description of documentation changes made]

Synchronized CLAUDE.md and GEMINI.md files.

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

# Push to GitHub main branch
git push origin main
```

**Documentation Sync Rules:**
- `CLAUDE.md` is the primary documentation file
- `GEMINI.md` must always be kept identical to `CLAUDE.md`
- Both files must be committed and pushed together
- Never modify `GEMINI.md` directly - always sync from `CLAUDE.md`

This ensures both Claude and Gemini AI assistants have access to the same up-to-date workflow documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ameer00)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ameer00)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
