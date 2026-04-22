## jda-ai-portal

> Enforces the use of type hints for function signatures in Python code.


# Cursor Rule Definition: Python Type Hint Requirement

## I. Rule Metadata (Frontmatter)

```yaml
---
ruleId: "code-quality-typehints-python-001"
description: "All Python function definitions must include type hints for parameters and return types."
severity: "WARNING"
globs:
  - "**/*.py"
alwaysApply: false
tags:
  - python
  - code-quality
  - typing
  - readability
relatedRules:
  - dev-workflow-master-001
  - code-quality-pep8-python-001
autoFixable: "SUGGEST_ONLY"
version: "1.0"
lastUpdated: "[YYYY-MM-DD]"
---
```

## II. Rule Definition & Explanation

### A. Rule Overview & Rationale

**Purpose:**  
Improve code readability, maintainability, and enable effective static analysis in Python codebases.

**Rationale/Benefit:**  
Type hints clarify function inputs and outputs, catch type-related bugs earlier, and enhance tooling support (e.g., MyPy, IDEs, autocomplete).

---

### B. Detailed Requirements & Specifications

1. Every user-defined function/method must include type hints for **all parameters**.  
2. Every function/method must have a type hint for its **return value**.  
3. Use standard types from `typing` module when appropriate (e.g., `List`, `Dict`, `Optional`, `Any`, `Callable`).  
4. Functions that return nothing must specify `-> None`.

---

### C. Examples of Compliance (✅ DO)

**Example 1:** Simple parameter and return types  
```python
# ✅ DO: Clear type hints for parameters and return value
def greet(name: str, loud: bool = False) -> str:
    message = f"Hello, {name}!"
    return message.upper() if loud else message
```

**Example 2:** Using types from `typing` and `-> None`  
```python
# ✅ DO: Complex types with no return value
from typing import List, Dict, Any

def process_records(records: List[Dict[str, Any]]) -> None:
    for record in records:
        print(f"Processing record ID: {record.get('id')}")
```

---

### D. Examples of Non-Compliance (❌ DON'T / Anti-Patterns)

**Anti-Example 1:** Missing all type hints  
```python
# ❌ DON'T: No type hints at all
def add(x, y):
    return x + y
```

**Anti-Example 2:** Parameter types present, return type missing  
```python
# ❌ DON'T: Missing return type
def multiply(a: int, b: int):  # Missing '-> int'
    return a * b
```

**Anti-Example 3:** Return type present, parameter types missing  
```python
# ❌ DON'T: Missing parameter types
def create_user(username, email) -> Dict[str, str]:  # Missing parameter types
    return {"username": username, "email": email}
```

---

### E. Automated Correction Logic / Suggestions

**Detection Pattern:**  
Use Python AST to detect `FunctionDef` nodes with missing `arg.annotation` or `returns`. Tools like MyPy and Pylint can help automate this check.

**Suggested Fix Description:**  
"Function `[function-name]` is missing type hints for parameter `[param-name]` or return value. Please add appropriate annotations."

**Conceptual Logic:**  
- Suggest `: Any` for unknown parameter types.  
- Suggest `-> None` if function has no return.  
- Suggest `-> Any` or infer the return type based on analysis.

---

### F. File & Rule References

- Governed by: `@dev-workflow-master-001.mdc`  
- Tooling: Enforced with static analysis tools (MyPy, Pylint). Cursor should be configured accordingly.

---

## III. Rule Maintenance Log

[YYYY-MM-DD] - v1.0 - [Your Name/AI Session ID]: Initial rule creation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aljereau) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
