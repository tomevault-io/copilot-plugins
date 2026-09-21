## always-rules-1-setup

> - **DO NOT** create new virtual environments


# Use Existing Setup - NO NEW VENVS

## 🚫 FORBIDDEN ACTIONS:
- **DO NOT** create new virtual environments
- **DO NOT** use `python3 -m venv` 
- **DO NOT** use `pip install --user`
- **DO NOT** create `test_venv` or similar
- **DO NOT** override existing dependencies

## ✅ REQUIRED ACTIONS:
- **ALWAYS** use the existing `venv/` directory
- **ALWAYS** use `source venv/bin/activate` 
- **ALWAYS** use `make install` if dependencies are missing
- **ALWAYS** use `make test-unit` to run tests
- **ALWAYS** work with the current setup

### **Dependency Management Workflow:**
1.  **Modify `pyproject.toml`**: Add new runtime dependencies under `[project].dependencies` or development dependencies under `[project.optional-dependencies].dev`.
2.  **Activate Virtual Environment**: Run `source .venv/bin/activate`.
3.  **Install Dependencies**: Run `make install` (this will automatically use `uv` if available).

## 🔧 EXISTING TOOLS:
- **Makefile**: Contains all setup commands
- **venv/**: Existing virtual environment
- **make install**: Installs dependencies properly
- **make test-unit**: Runs unit tests
- **make clean**: Cleans up properly

## 📋 WORKFLOW:
1. **Check existing setup**: `ls venv/`
2. **Activate existing venv**: `source venv/bin/activate`
3. **Install if needed**: `make install`
4. **Test**: `make test-unit`
5. **Fix issues in existing code, don't rebuild infrastructure**

## 🎯 GOAL:
**Working software using current setup, not rebuilding from scratch.**

## ⚠️ REMEMBER:
- The user wants **working software**
- The user wants you to use the **current setup**
- The user does **NOT** want new virtual environments
- The user does **NOT** want dependency management changes
- **FIX THE ACTUAL PROBLEMS, don't change the environment**

## 🔍 WHEN TESTING:
- Use `source venv/bin/activate && make test-unit`
- If tests fail, fix the test logic, not the environment
- If dependencies missing, use `make install`
- **NEVER** create new venvs or change Python paths

## 💡 EXAMPLE CORRECT COMMAND:
```bash
source venv/bin/activate && make test-unit
```

## ❌ EXAMPLE WRONG COMMAND:
```bash
python3 -m venv test_venv  # NEVER DO THIS
```

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
