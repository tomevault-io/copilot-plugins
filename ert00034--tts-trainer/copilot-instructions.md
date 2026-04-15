## tts-trainer

> This project must work across Windows, macOS, and Linux environments. Always use cross-platform approaches.

# Cross-Platform Compatibility Guide

This project must work across Windows, macOS, and Linux environments. Always use cross-platform approaches.

## User Environment Detection

This project is currently being developed on **Windows 10.0.26100** with **PowerShell** as the shell. Always consider Windows-specific requirements.

## Terminal Commands

### ❌ **NEVER use Unix-only commands**
```bash
# WRONG - Unix/Linux only
mkdir -p directory/subdirectory
cp -r source dest
ls -la
rm -rf directory
```

### ✅ **Use PowerShell commands for Windows**
```powershell
# CORRECT - Windows PowerShell
New-Item -ItemType Directory -Force -Path "directory\subdirectory"
Copy-Item -Recurse source dest
Get-ChildItem -Force
Remove-Item -Recurse -Force directory
```

### ✅ **Or use Python for cross-platform operations**
```python
# BEST - Cross-platform Python
from pathlib import Path
Path("directory/subdirectory").mkdir(parents=True, exist_ok=True)
```

## File Paths

### Windows Path Conventions
- Use backslashes `\` in PowerShell commands
- Use forward slashes `/` in Python code (Path handles conversion)
- Always quote paths with spaces: `"Program Files\MyApp"`

### Python Path Handling
```python
# CORRECT - Always use pathlib.Path
from pathlib import Path
config_path = Path("config") / "models" / "xtts_v2.yaml"
```

## Directory Creation Patterns

### For multiple directories at once:
```powershell
# Windows PowerShell - create multiple directories
New-Item -ItemType Directory -Force -Path "dir1", "dir2", "dir3"
```

### For nested directory structures:
```powershell
# Windows PowerShell - create nested directories
New-Item -ItemType Directory -Force -Path "parent\child\grandchild"
```

## Common Cross-Platform Issues

### 1. **Case Sensitivity**
- Windows: Case-insensitive filesystem
- Linux/macOS: Case-sensitive filesystem
- Always use consistent casing in file references

### 2. **Line Endings**
- Windows: `\r\n` (CRLF)
- Unix: `\n` (LF)
- Configure Git: `git config core.autocrlf true` (Windows)

### 3. **Environment Variables**
```powershell
# Windows
$env:VARIABLE_NAME
```
```bash
# Unix
$VARIABLE_NAME
```

### 4. **Python Virtual Environments**
```powershell
# Windows activation
venv\Scripts\Activate.ps1
```
```bash
# Unix activation
source venv/bin/activate
```

## Development Environment Setup

### Always Check User OS
Before running terminal commands, consider the user's environment:
- **Windows**: Use PowerShell commands or Python
- **macOS/Linux**: Use Unix commands or Python
- **Python**: Always prefer for cross-platform compatibility

### Required Tools Installation
Ensure these are available across platforms:
- **ffmpeg**: Audio/video processing
- **Python 3.9+**: Core language
- **Git**: Version control
- **CUDA** (optional): GPU acceleration

## Best Practices

1. **Always use Python's pathlib** for file operations
2. **Quote all paths** in terminal commands
3. **Test commands** before suggesting them to users
4. **Prefer Python solutions** over shell commands when possible
5. **Document OS-specific requirements** clearly
6. **Use forward slashes** in configuration files and Python code

## Emergency Fixes

If a Unix command was used on Windows:
1. Identify the equivalent PowerShell command
2. Re-run with proper Windows syntax
3. Update documentation to prevent future issues
4. Consider creating a Python alternative

## Platform Detection in Code

```python
import platform
import sys

def get_platform_info():
    return {
        'system': platform.system(),  # Windows, Darwin, Linux
        'release': platform.release(),
        'version': platform.version(),
        'python_version': sys.version
    }

# Use this to adapt behavior per platform
if platform.system() == 'Windows':
    # Windows-specific code
    pass
elif platform.system() == 'Darwin':
    # macOS-specific code
    pass
else:
    # Linux/Unix-specific code
    pass
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ert00034)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ert00034)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
