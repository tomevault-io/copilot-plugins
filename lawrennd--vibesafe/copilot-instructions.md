## vibesafe-template-editing

> When working on the VibeSafe repository itself, you must understand the distinction between **template files** and **deployed files**.

# VibeSafe Template Editing Rule

## ⚠️ CRITICAL: Always Edit Templates, Not Deployed Files

When working on the VibeSafe repository itself, you must understand the distinction between **template files** and **deployed files**.

### 🔧 Template Files (ALWAYS EDIT THESE)

These are the source files that get copied to user projects during installation:

```
templates/
├── .cursor/rules/              # ← Edit cursor rules HERE
├── scripts/                    # ← Edit user-facing scripts HERE
│   ├── whats_next.py          # ← Edit whats-next script HERE
│   └── validate_vibesafe_structure.py  # ← Edit validator HERE
├── backlog/
│   ├── README.md              # ← Edit backlog docs HERE
│   ├── task_template.md       # ← Edit task template HERE
│   └── update_index.py        # ← Edit update script HERE
├── cip/
│   ├── README.md              # ← Edit CIP docs HERE
│   └── cip_template.md        # ← Edit CIP template HERE
├── tenets/
│   ├── README.md              # ← Edit tenet docs HERE
│   ├── tenet_template.md      # ← Edit tenet template HERE
│   └── combine_tenets.py      # ← Edit tenet script HERE
└── requirements/              # ← Edit requirements framework HERE
```

### 📦 Deployed Files (NEVER EDIT THESE DIRECTLY)

These are copies that exist in the VibeSafe repo root for **dogfooding** (testing VibeSafe on itself):

```
.cursor/rules/                 # ❌ DON'T edit - deployed from templates
backlog/README.md              # ❌ DON'T edit - deployed from templates
backlog/task_template.md       # ❌ DON'T edit - deployed from templates
cip/README.md                  # ❌ DON'T edit - deployed from templates
cip/cip_template.md            # ❌ DON'T edit - deployed from templates
tenets/README.md               # ❌ DON'T edit - deployed from templates
tenets/tenet_template.md       # ❌ DON'T edit - deployed from templates
```

### ✅ Correct Workflow

**1. Edit the template:**
```bash
# ✅ CORRECT
vim templates/.cursor/rules/vibesafe_general.mdc
```

**2. Run dogfood install to test:**
```bash
# ✅ Deploy templates to local VibeSafe repo for testing
bash <(curl -s https://raw.githubusercontent.com/lawrennd/vibesafe/main/scripts/install-minimal.sh)

# OR use local install script
bash scripts/install-minimal.sh
```

**3. Verify the changes:**
```bash
# Check that deployed file matches template
diff templates/.cursor/rules/vibesafe_general.mdc .cursor/rules/vibesafe_general.mdc
```

### ❌ Wrong Workflow

```bash
# ❌ WRONG - Editing deployed file
vim .cursor/rules/vibesafe_general.mdc

# This change will be LOST on next install!
# Other users won't get this change!
```

### 🎯 Quick Check

Before editing a VibeSafe system file, ask yourself:

1. **Am I in the templates/ directory?**
   - Yes → ✅ Proceed with edit
   - No → ❌ Stop! Find the template version

2. **Is this file in the VibeSafe gitignore?**
   - Yes → ❌ It's deployed, find the template
   - No → Check if it's a template

3. **When in doubt:**
   - Search for the file in `templates/` first
   - If it exists there, edit the template
   - Then run dogfood install to deploy

### 📁 Exception: Files That Don't Have Templates

These files are unique to the VibeSafe repo and should be edited directly:

- `README.md` (root) - VibeSafe project README
- `scripts/install-minimal.sh` - Installation script
- `install-whats-next.sh` - Installation script
- `scripts/add_*.py` - VibeSafe development tools (not deployed)
- CIP files (`cip000*.md`) - VibeSafe's own CIPs
- Backlog tasks (`backlog/*/YYYY-MM-DD_*.md`) - VibeSafe's own tasks
- Tenet files (actual tenets, not template) - VibeSafe's own tenets

### ⚠️ Special Case: User-Facing Scripts

User-facing scripts (`whats_next.py`, `validate_vibesafe_structure.py`) should be:
1. **Edited in `scripts/`** (for dogfooding)
2. **Copied to `templates/scripts/`** (for user deployment)
3. **Both kept in sync**

After editing these scripts, always copy to templates:
```bash
cp scripts/whats_next.py templates/scripts/
cp scripts/validate_vibesafe_structure.py templates/scripts/
```

### 🔄 Dogfooding Process

VibeSafe uses itself for project management, which means:

1. Templates are the **source of truth**
2. Deployed files in the repo are for **testing VibeSafe on itself**
3. Changes to templates must be **dogfooded** (installed locally) to test
4. This ensures what we ship to users is what we use ourselves

### 📝 Summary

- ✅ **DO**: Edit files in `templates/`
- ✅ **DO**: Run dogfood install after template changes
- ✅ **DO**: Test that deployed files match expectations
- ❌ **DON'T**: Edit deployed cursor rules directly
- ❌ **DON'T**: Edit deployed system READMEs or templates directly
- ❌ **DON'T**: Forget to dogfood your changes

---
> Source: [lawrennd/vibesafe](https://github.com/lawrennd/vibesafe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
