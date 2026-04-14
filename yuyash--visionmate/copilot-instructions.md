## visionmate

> Visionmate is a multi-modal assistant that continuously captures UVC video, screen content, or audio, streaming them to a VLM (Vision Language Model) to fuse visual and spoken context for near real-time conversational understanding.

# AGENTS.md

## About this project

Visionmate is a multi-modal assistant that continuously captures UVC video, screen content, or audio, streaming them to a VLM (Vision Language Model) to fuse visual and spoken context for near real-time conversational understanding.

## Dev environment tips

- This project uses `uv` to set up the Python environment. Python is installed in the `${workspaceFolder}/.venv` directory.
- Use `uv sync` to install dependencies synced with pyproject.toml.
- Use `uv pip install` to install additional libraries.
- Use `uv run ruff check .` and `uv run ty check .` for linting, formatting, and type checking.

## Project Structure

Key directories and files:

```
.
├── AGENTS.md                   # Agent instructions
├── CHANGELOG.md                # Release notes
├── LICENSE                     # License information
├── README.md                   # Project overview
├── pyproject.toml              # Packaging and tool configuration
├── release-please-config.json  # Release automation configuration
├── uv.lock                     # Locked dependencies for uv
├── src/
│   └── visionmate/             # Main package
│       ├── __main__.py         # Entry point
│       ├── core/               # Core domain logic
│       │   ├── capture/        # Audio and video capture pipeline
│       │   ├── multimedia/     # Buffering, detection, and segments
│       │   ├── recognition/    # VLM/STT clients and processing
│       │   ├── session/        # Session lifecycle management
│       │   ├── settings/       # Settings management
│       │   ├── logging.py      # Logging setup
│       │   └── models.py       # Shared domain models
│       ├── desktop/            # Desktop UI
│       │   ├── main.py         # App entry point
│       │   ├── styles.py       # UI styles
│       │   ├── dialogs/        # Modal dialogs
│       │   └── widgets/        # UI widgets
│       └── web/                # Web package (currently minimal)
└── tests/
    ├── unit/                   # Unit tests
    ├── integration/            # Integration tests
    └── e2e/                    # End-to-end tests
```

## Design instructions

- Always consider testability, readability, and disposability.
- Always consider separation of concerns so that each module, class, and function has a clear boundary.
- Use Enum instead of str, and create objects rather than using multiple primitive variables.

## Coding instructions

- Do not hesitate to restructure the existing code.
- Do not create duplicate models, logic, or functions.
- Always use type hint for attributes and variables and returns.
- Always consider splitting files if the file is longer than 500 lines.

## Testing instructions

- Use `uv run pytest` to execute tests.
- Create unit tests under the `${workspaceFolder}/tests/unit` directory.
- Create integration tests under the `${workspaceFolder}/tests/integration` directory.
- Create end-to-end tests under the `${workspaceFolder}/tests/e2e` directory.
- Always create a unit test for a new module. The module name must start with "test\_".

## PR instructions

- Always run `uv run pytest`, `uv run ruff check .`, and `uv run ty check .` before committing changes.
- Always create a new branch for pull requests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuyash) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
