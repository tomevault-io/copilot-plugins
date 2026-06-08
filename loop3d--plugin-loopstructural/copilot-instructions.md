## plugin-loopstructural

> This document outlines the architectural and development principles for contributors to the `plugin_loopstructural` QGIS plugin. The plugin provides a thin, modular interface to the `map2loop` and `LoopStructural` libraries, enabling geological modeling workflows within QGIS.

# AGENTS.md

## Purpose

This document outlines the architectural and development principles for contributors to the `plugin_loopstructural` QGIS plugin. The plugin provides a thin, modular interface to the `map2loop` and `LoopStructural` libraries, enabling geological modeling workflows within QGIS.

---

## Design Philosophy

### 🔹 Thin Interface Layer
- The plugin **must not** reimplement or duplicate functionality from `map2loop` or `LoopStructural`.
- All core logic and enhancements should be contributed upstream to the respective libraries.
- The plugin should focus on **UI integration**, **data flow orchestration**, and **user interaction**.

### 🔹 Modularity
- UI components (dialogs, panels, actions) live under the `loopstructural/gui/` package and should be encapsulated in their own classes.
- Business logic and orchestration are located in `loopstructural/main/` and `loopstructural/toolbelt/` where adapters and services wrap external libraries.
- Processing algorithms and QGIS provider integration are in `loopstructural/processing/` and should be isolated from UI code.
- Avoid tight coupling between components. Use signals/slots or event-driven patterns where appropriate.

### 🔹 Object-Oriented Design
- Use classes with clear responsibilities and interfaces.
- Prefer composition over inheritance unless subclassing is semantically appropriate.
- Encapsulate interactions with external libraries in dedicated adapter or service classes (e.g., `loopstructural.main.Map2LoopService`, `loopstructural.main.LoopStructuralRunner`).

---

## Development Guidelines

### ✅ Code Quality
- All code must pass the repository's pre-commit checks (formatting, linting, import sorting).
- Use type hints and docstrings for all public methods and classes.
- Follow PEP8 and QGIS plugin development best practices.

### 🧪 Testing
- All new code must include **unit tests** and, where applicable, **integration tests**.
- Tests live under the `tests/` package and are runnable with `pytest`.
- Mock external dependencies (`map2loop`, `LoopStructural`) in unit tests.

### 🧩 Current Plugin Structure

```
plugin_loopstructural/
├── loopstructural/           # plugin package
│   ├── __init__.py
│   ├── __about__.py
│   ├── plugin_main.py        # QGIS plugin entry and bootstrap
│   ├── gui/                  # UI dialogs, widgets, and panels
│   ├── main/                 # controllers, managers, adapters (service layer)
│   ├── processing/           # QGIS processing provider and algorithms
│   ├── toolbelt/             # utilities, env parsing, preferences, logging
│   ├── resources/            # icons, translations, help files
│   └── requirements.txt
├── docs/
├── requirements/
├── tests/
└── README.md
```

Notes on mapping older concepts:
- What used to be called `services/` and `controllers/` is implemented across `loopstructural/main/` and `loopstructural/toolbelt/`.
- UI remains in `loopstructural/gui/` (dialogs, `.ui` files, widget classes).
- Processing-specific code and QGIS provider live under `loopstructural/processing/`.

---

## Contribution Workflow

1. Fork the repository and create a feature branch.
2. Implement changes following the design and code quality guidelines.
3. Add or update tests under `tests/` and ensure they run with `pytest`.
4. Run pre-commit hooks (e.g. `pre-commit run --all-files`) and ensure all checks pass.
5. Submit a pull request with a clear description of the changes and rationale. Link to upstream libraries if behavior is moved upstream.

---

## Future Enhancements

- Support for asynchronous or background processing of long-running tasks (consider using QGIS background task framework).
- Improved error handling and user feedback.
- Internationalization (i18n) support and keeping `.ts`/.qm translation files in `loopstructural/resources/i18n/`.
- Better separation of concerns between UI, processing algorithms and adapters to facilitate unit testing.

---

## Contact

For questions or contributions to the upstream libraries:
- `map2loop`
- `LoopStructural`

---

---
> Source: [Loop3D/plugin_loopstructural](https://github.com/Loop3D/plugin_loopstructural) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
