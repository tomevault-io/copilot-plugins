## prompt-builder

> These rules ensure that any code generation involving external libraries triggers immediate dependency installation to prevent import errors and streamline testing.

 # Dependency Management Rules

## Overview
These rules ensure that any code generation involving external libraries triggers immediate dependency installation to prevent import errors and streamline testing.

## Node.js Dependencies

### Package Management
1. When generating code with npm dependencies:
   ```bash
   # Check if package.json exists
   if [ ! -f package.json ]; then
     npm init -y
   fi
   
   # Install dependencies immediately
   npm install <package-name> --save
   
   # For dev dependencies
   npm install <package-name> --save-dev
   ```

2. Version Specification:
   - Always specify version constraints
   - Use exact versions for critical dependencies
   - Use caret (^) for flexible minor version updates
   - Example: `"express": "^4.18.2"`

3. Package Lock:
   - Always commit package-lock.json
   - Never modify package-lock.json manually
   - Run `npm ci` for clean installs

## Python Dependencies

### Package Management
1. When generating code with pip dependencies:
   ```bash
   # Check if requirements.txt exists
   if [ ! -f requirements.txt ]; then
     touch requirements.txt
   fi
   
   # Install dependencies immediately
   pip install <package-name>
   
   # Update requirements.txt
   pip freeze > requirements.txt
   ```

2. Virtual Environment:
   ```bash
   # Create venv if not exists
   if [ ! -d venv ]; then
     python -m venv venv
   fi
   
   # Activate venv
   source venv/bin/activate  # Unix
   # or
   .\venv\Scripts\activate  # Windows
   ```

3. Version Specification:
   - Use exact versions for reproducibility
   - Example: `requests==2.31.0`

## Implementation Guidelines

### For Code Generation
1. Pre-generation Checks:
   - Verify package manager availability
   - Check for existing dependency files
   - Validate environment setup

2. During Generation:
   - Add imports at the top of files
   - Document required dependencies
   - Include version requirements

3. Post-generation Actions:
   - Auto-install dependencies
   - Update lock files
   - Verify installations

### For Testing
1. Before Tests:
   - Verify all dependencies are installed
   - Check for version conflicts
   - Ensure virtual environment activation

2. Test Environment:
   - Use isolated test environments
   - Include test-specific dependencies
   - Maintain separate test requirements

### Error Prevention
1. Dependency Conflicts:
   - Check for compatibility
   - Resolve version conflicts
   - Document breaking changes

2. Missing Dependencies:
   - Auto-detect missing imports
   - Suggest installations
   - Provide fallback options

## Automation Scripts

### Node.js
```javascript
// dependency-check.js
const fs = require('fs');
const { execSync } = require('child_process');

function installNodeDependencies(dependencies) {
  const pkgJson = JSON.parse(fs.readFileSync('package.json', 'utf8'));
  const newDeps = dependencies.filter(dep => !pkgJson.dependencies[dep]);
  
  if (newDeps.length > 0) {
    execSync(`npm install ${newDeps.join(' ')} --save`);
    console.log('Installed new dependencies:', newDeps);
  }
}
```

### Python
```python
# dependency_check.py
import subprocess
import pkg_resources
import sys

def install_python_dependencies(dependencies):
    installed = {pkg.key for pkg in pkg_resources.working_set}
    missing = [pkg for pkg in dependencies if pkg.split('==')[0] not in installed]
    
    if missing:
        subprocess.check_call([sys.executable, '-m', 'pip', 'install'] + missing)
        subprocess.check_call([sys.executable, '-m', 'pip', 'freeze', '>', 'requirements.txt'])
        print(f"Installed new dependencies: {missing}")
```

## Best Practices

1. Dependency Documentation:
   - Maintain clear documentation
   - Include purpose of each dependency
   - Document version constraints

2. Security:
   - Regular security audits
   - Update vulnerable packages
   - Use trusted sources only

3. Performance:
   - Minimize dependency count
   - Use lightweight alternatives
   - Remove unused dependencies

4. Maintenance:
   - Regular updates
   - Dependency cleanup
   - Version compatibility checks

## Integration with CI/CD

1. Automated Checks:
   ```yaml
   dependency_check:
     runs-on: ubuntu-latest
     steps:
       - uses: actions/checkout@v3
       - name: Check Dependencies
         run: |
           ./scripts/check_dependencies.sh
   ```

2. Version Control:
   - Commit dependency files
   - Review dependency changes
   - Track version updates

## Error Messages and Solutions

Common errors and their solutions:
```text
Error: Cannot find module 'xyz'
Solution: Auto-run `npm install xyz`

Error: ModuleNotFoundError: No module named 'xyz'
Solution: Auto-run `pip install xyz`
```

## Rule Enforcement

This rule should be enforced:
1. During code generation
2. Before running tests
3. In CI/CD pipelines
4. During code review

## Prefix Convention
Prefix chat messages with (📦 rule:dependency-management) when this rule is referenced 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kingler) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
