## smartcash

> - **Always read `PLANNING.md` and `MODEL_ARC.md`** at the start of a new conversation to understand the project's architecture, goals, style, and constraints.

### 🔄 Project Awareness & Context
- **Always read `PLANNING.md` and `MODEL_ARC.md`** at the start of a new conversation to understand the project's architecture, goals, style, and constraints.
- **Check `TASK.md`** before starting a new task. If the task isn’t listed, add it with a brief description and today's date.
- **Use consistent naming conventions, file structure, and architecture patterns** as described in `PLANNING.md`.
- **Use venv_linux** (the virtual environment named 'venv-test') whenever executing Python commands, including for unit tests.

### 🧱 Code Structure & Modularity
- **Never create a file longer than 500 lines of code.** If a file approaches this limit, refactor by splitting it into modules or helper files.
- **Organize code into clearly separated modules**, grouped by feature or responsibility.
- **Use clear, consistent imports** (prefer relative imports within packages).
- **Use python_dotenv and load_env()** for environment variables.

### 🧪 Testing & Reliability
- **Always create Pytest unit tests for new features** (functions, classes, routes, etc).
- **After updating any logic**, check whether existing unit tests need to be updated. If so, do it.
- **Tests should live in a `/tests` folder** mirroring the main app structure.
  - Include at least:
    - 1 test for expected use
    - 1 edge case
    - 1 failure case

### ✅ Task Completion
- **Mark completed tasks in `TASK.md`** immediately after finishing them.
- Add new sub-tasks or TODOs discovered during development to `TASK.md` under a “Discovered During Work” section.

### 📎 Style & Conventions
- **Use Python** as the primary language.
- **Follow PEP8**, use type hints, and format with `black`.
- Write **docstrings for every function** using the Google style:
  ```python
  def example():
      """
      Brief summary.

      Args:
          param1 (type): Description.

      Returns:
          type: Description.
      """
  ```

## 📋 Development DO and DON'Ts

### ✅ DO - Best Practices

#### Logger Usage
- **DO** use instance loggers (`self.logger`) instead of module-level loggers
- **DO** use function-based logger access (`_get_logger()`) for module-level operations
- **DO** prevent logger propagation with `logger.propagate = False` in UI logger setup
- **DO** use operation container logging to redirect logs to UI components

#### Type Safety and Error Handling
- **DO** add comprehensive type checking with `isinstance(data, dict)` before calling `.get()`
- **DO** handle both dict and string inputs gracefully in formatting functions
- **DO** provide fallback values and error messages for invalid data types
- **DO** use try-catch blocks around all UI operations with proper error logging

#### Status Panel Integration
- **DO** store both header_container object and widget separately for access to methods
- **DO** implement `_update_status()` helper methods in UIModule classes
- **DO** provide real-time feedback for all major operations (start, success, error states)
- **DO** use consistent status types: 'info', 'success', 'warning', 'error'

#### UIModule Architecture
- **DO** follow the standardized container order: Header → Form → Action → Summary → Operation → Footer
- **DO** register components with clear, consistent naming conventions
- **DO** use SharedMethodRegistry for cross-module functionality
- **DO** implement proper cleanup in module destructors

#### Code Organization
- **DO** keep functions under 500 lines by splitting into helper methods
- **DO** use clear, descriptive function and variable names
- **DO** add comprehensive docstrings with Args and Returns documentation
- **DO** group related functionality into logical modules and classes

### ❌ DON'T - Common Pitfalls

#### Logger Anti-Patterns
- **DON'T** use module-level logger declarations like `logger = get_module_logger(__name__)`
- **DON'T** call logger objects directly in event handlers (causes "Logger object is not callable" errors)
- **DON'T** mix standard logging with UI container logging (causes duplicate logs)
- **DON'T** forget to suppress console output when using operation containers

#### Type and Data Handling
- **DON'T** assume data structures are always dictionaries without type checking
- **DON'T** call `.get()` methods on variables that might be strings
- **DON'T** ignore None or empty values in data formatting functions
- **DON'T** let exceptions in formatting functions crash the entire UI

#### UI Component Management
- **DON'T** store only widget containers when you need access to object methods
- **DON'T** create UI components without proper error handling
- **DON'T** forget to implement status updates for user feedback
- **DON'T** mix different UI frameworks or patterns within the same module

#### Architecture Violations
- **DON'T** bypass the BaseUIModule pattern for new modules
- **DON'T** duplicate common functionality across modules (use mixins instead)
- **DON'T** create direct dependencies between modules (use SharedMethodRegistry instead)
- **DON'T** implement complex multi-tab interfaces (use single-screen approach)

#### Performance and Memory
- **DON'T** create memory leaks by not cleaning up event handlers
- **DON'T** hold references to large objects in module-level variables
- **DON'T** initialize heavy operations during module import
- **DON'T** block the UI thread with long-running operations


### 📚 Documentation & Explainability
- **Update `ARCHITECTURE.md`** when new UI related features are added, dependencies change, or setup steps are modified.
- **Comment non-obvious code** and ensure everything is understandable to a mid-level developer.
- When writing complex logic, **add an inline `# Reason:` comment** explaining the why, not just the what.

### 🧠 AI Behavior Rules
- **Never assume missing context. Ask questions if uncertain.**
- **Never hallucinate libraries or functions** – only use known, verified Python packages.
- **Always confirm file paths and module names** exist before referencing them in code or tests.
- **Never delete or overwrite existing code** unless explicitly instructed to or if part of a task from `TASK.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masdevid)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/masdevid)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
