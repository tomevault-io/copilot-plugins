## data-detector

> **Data Detector** is a high-performance engine designed for detecting, redacting, and generating sensitive data (PII). It supports multiple languages (including NLP-enhanced detection for CJK languages), provides a CLI, a library for Python integration, a REST/gRPC server, and a Chrome Extension for real-time monitoring.

# Data Detector

## Project Overview

**Data Detector** is a high-performance engine designed for detecting, redacting, and generating sensitive data (PII). It supports multiple languages (including NLP-enhanced detection for CJK languages), provides a CLI, a library for Python integration, a REST/gRPC server, and a Chrome Extension for real-time monitoring.

### Key Features
*   **Detection:** Finds PII using Regex and NLP (Korean, Chinese, Japanese).
*   **Redaction:** Masks or tokenizes sensitive data.
*   **Generation:** Creates fake PII data for testing.
*   **Verification:** Validates detected data (e.g., checksums for ID numbers).
*   **Chrome Extension:** Real-time PII monitoring in the browser.

### Tech Stack
*   **Language:** Python 3.8+
*   **Core Libraries:** `fastapi`, `grpcio`, `google-re2`, `click`, `pydantic`.
*   **NLP:** `konlpy`, `jieba`, `sudachipy`.
*   **Testing/Quality:** `pytest`, `ruff`, `black`, `mypy`.
*   **Extension:** JavaScript, HTML, CSS (Manifest V3).

## Building and Running

### Installation
```bash
# Standard installation
make install
# or
pip install -e .

# Development installation (includes dev, test, nlp deps)
make install-dev
# or
pip install -e ".[dev,test,nlp]"
```

### Testing
```bash
# Run tests with coverage
make test
# or
pytest --cov=.
```

### Linting and Formatting
```bash
# Check code style and types
make check

# Fix formatting issues
make format
```

### CLI Usage
The `data-detector` command is the primary interface.
```bash
# Find PII
data-detector find --text "My phone is 010-1234-5678" --ns kr

# Redact file
data-detector redact --in input.log --out redacted.log

# Start Server
data-detector serve --port 8080
```

### Server
```bash
# Start development server
make serve
```

### Docker
```bash
# Build image
make docker-build

# Run container
make docker-run
```

## Chrome Extension
The extension source is located in `chrome-extension/`.

1.  **Build/Prepare:** Ensure the API server is running (`make serve`).
2.  **Load:** Open `chrome://extensions/` in Chrome, enable **Developer mode**, and select **Load unpacked**. Point to the `chrome-extension` directory.

## Directory Structure

*   `src/datadetector/`: Core Python package source.
*   `chrome-extension/`: Source code for the Chrome browser extension.
*   `tests/`: Unit and integration tests.
*   `docs/`: Extensive documentation (Architecture, API, Guides).
*   `pii-pattern-engine/`: Submodule/Directory containing detection patterns.
*   `examples/`: Usage examples.
*   `docker/`: Docker configuration files.

## Development Conventions

*   **Code Style:** Adheres to `black` and `ruff`. Types are enforced with `mypy`.
*   **Testing:** All features must include unit tests. Run `make check` before committing.
*   **Patterns:** New PII patterns should include validation logic and test cases.

---
> Source: [zafrem/Data-detector](https://github.com/zafrem/Data-detector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
