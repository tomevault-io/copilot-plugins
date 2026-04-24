## skill-interpreter

> A Python-based implementation of a SKILL programming language interpreter.

# SKILL Interpreter

A Python-based implementation of a SKILL programming language interpreter.

SKILL is a programming language commonly used in the electronic design automation (EDA) industry, particularly for custom IC layout and design.

## Features

- Modular and extensible architecture with clean interfaces
- Parser for SKILL syntax and expressions
- Runtime environment with built-in procedures
- Support for defining and calling SKILL procedures
- Python integration: register Python functions as SKILL procedures
- IC layout capabilities: create cells, rectangles, and other shapes
- Compatibility layer for legacy code


1. **Modular Architecture**: We've organized the code into clean, well-separated modules:
   - Interface definitions
   - Parser implementation
   - Runtime environment
   - Main interpreter
   - Mock implementations
   - Compatibility layer
   - Utility functions

2. **Clean API**: The package provides a simple and intuitive API:
   - Execute SKILL code
   - Load SKILL files
   - Register procedures
   - Create multiple interpreters
   - Debug options

3. **Packaging**: We've created all the necessary files for a proper Python package:
   - setup.py
   - pyproject.toml
   - README.md
   - Proper import structure

4. **Development Tools**: We've set up configurations for common development tools:
   - pytest for testing
   - black and isort for formatting
   - mypy for type checking
   - flake8 for linting

5. **Compatibility**: We've ensured backward compatibility with legacy code through:
   - A compatibility layer
   - System module patching
   - Consistent API across implementations

6. **Documentation**: We've provided comprehensive documentation:
   - Detailed README with examples
   - Docstrings for all classes and functions
   - Usage examples

7. **Examples**: We've included a usage example that demonstrates key features

## Installation

### Using pip

```bash
pip install skill-interp
```

### From Source

```bash
git clone https://github.com/jerryzhao173985/skill_interp.git
cd skill_interp
pip install -e .
```

## Quick Start

```python
import skill_interp as skill

# Execute simple SKILL code
result = skill.execute_code("(+ 5 10)")
print(result)  # 15

# Define a procedure
skill.execute_code("""
procedure(square (x)
    x * x
)
""")

# Call the procedure
result = skill.execute_code("(square 5)")
print(result)  # 25

# Register a Python function as a SKILL procedure
@skill.register_procedure('pythonFunction')
def python_function(a, b):
    return a * b + a + b

# Call the Python function from SKILL
result = skill.execute_code('(pythonFunction 3 4)')
print(result)  # 19

# Load a SKILL file
skill.load_file("path/to/your/file.il")
```

## Architecture

The SKILL interpreter is designed with modularity in mind, separating concerns into distinct components:

### Core Components

1. **Interface Module** (`skill_interp.interface`)
   - Defines the core interfaces for all components
   - Provides exception classes for error handling

2. **Parser Module** (`skill_interp.parser`)
   - Parses SKILL code into a simplified AST
   - Handles procedure definitions, function calls, variables, etc.

3. **Runtime Module** (`skill_interp.runtime`)
   - Manages procedures and variables
   - Provides built-in SKILL procedures
   - Maintains execution context

4. **Interpreter Module** (`skill_interp.interpreter`)
   - Connects the parser and runtime
   - Provides the main API for executing SKILL code

5. **Mock Module** (`skill_interp.mock`)
   - Provides mock implementations for testing
   - Fallback implementation when needed

6. **Utils Module** (`skill_interp.utils`)
   - Helper functions for common tasks
   - File handling and procedure extraction

### Design Principles

- **Clean Interfaces**: All components implement clearly defined interfaces
- **Separation of Concerns**: Parser, runtime, and interpreter are decoupled
- **Extensibility**: Easy to extend with custom procedures and functionality
- **Compatibility**: Support for legacy code through backward compatibility layers

## Advanced Usage

### Multiple Interpreters

You can create multiple independent interpreter instances:

```python
import skill_interp as skill

# Create two independent interpreters
interp1 = skill.create_interpreter()
interp2 = skill.create_interpreter()

# They maintain separate environments
interp1.execute_code("x = 10")
interp2.execute_code("x = 20")

print(interp1.execute_code("x"))  # 10
print(interp2.execute_code("x"))  # 20
```

### Debug Mode

You can enable debug mode to see more information about parsing and execution:

```python
import skill_interp as skill

# Enable debug mode
skill.set_debug_mode(True)

# Now code execution will print debug information
skill.execute_code("(+ 1 2)")
```

### Working with IC Layout

The interpreter includes basic support for IC layout concepts:

```python
import skill_interp as skill

# Create a cell view
cell = skill.execute_code('(ddGetObj "mylib" "mycell")')

# Create a rectangle
rect = skill.execute_code('(rodCreateRect "metal1" 1.0 2.0)')

# Create a label
label = skill.execute_code('(dbCreateLabel cv "metal1" "Label" [1.0 1.0])')
```

### Utility Functions

The package includes utility functions for working with SKILL files:

```python
from skill_interp.utils import (
    find_skill_files,
    extract_procedures_from_file,
    batch_load_skill_files,
    time_execution
)

# Find all SKILL files in a directory
files = find_skill_files("./designs")

# Extract procedures from a file
procs = extract_procedures_from_file("./designs/main.il")

# Load multiple files
results = batch_load_skill_files(files)

# Time the execution of SKILL code
result, exec_time = time_execution("(+ 1 2 3 4 5)", iterations=1000)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jerryzhao173985) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
