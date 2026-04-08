## r-pipeline

> This document provides an AI-centric overview of the project, enabling Gemini and other large language models to understand the codebase, its conventions, and how to effectively assist in its development.

# GEMINI.md - Your AI Assistant's Guide to this Repository

This document provides an AI-centric overview of the project, enabling Gemini and other large language models to understand the codebase, its conventions, and how to effectively assist in its development.

## Project Overview

This repository contains a sophisticated, polyglot (PowerShell, Python, TypeScript) framework for AI-assisted code quality enforcement. The system, referred to as "AIUOKEEP," is designed to function as a deterministic, policy-driven guardrail platform. It mediates AI-generated code changes through a custom "Model Context Protocol (MCP)," a "SafePatch" validation pipeline, and a comprehensive audit and governance layer.

The architecture is highly modular, consisting of 24 distinct pipeline modules as outlined in the `MASTER PLAN` document. These modules handle everything from goal normalization and discovery to code synthesis, validation, and knowledge reuse. The project is designed to be built in phases, starting with a streamlined file-watcher and progressively adding more complex capabilities.

### Key Technologies

*   **PowerShell:** The primary language for orchestration, automation, and scripting. It uses `Invoke-Build` as a task runner, `PSScriptAnalyzer` for linting, and `Pester` for testing.
*   **Python:** A first-class language for tooling, MCP servers, and application logic. It uses `ruff` for formatting and linting, `pyright` for type checking, and `pytest` for testing. `pyinvoke` is available as an alternative task runner.
*   **TypeScript:** Supported for front-end or other components, with `prettier` for formatting.
*   **Model Context Protocol (MCP):** A key architectural pattern for exposing development tools (linters, test runners, etc.) as services that AI agents can consume.
*   **Open Policy Agent (OPA):** Used for enforcing policies on code and configuration.
*   **Semgrep:** Used for Static Application Security Testing (SAST).
*   **GitHub Actions:** The foundation for all CI/CD and automated validation workflows.

## Repository Layout

The repository is organized into the following key directories:

*   `./SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-/`: The core of the project, containing the main application logic, configuration, and documentation.
    *   `./.github/workflows/`: Contains all GitHub Actions workflows for CI/CD, including `powershell-verify.yml`.
    *   `./.mcp/`: Scripts for initializing and managing the Model Context Protocol (MCP) environment.
    *   `./docs/`: Detailed documentation on architecture, conventions, guardrails, and more.
    *   `./mcp-servers/`: Implementations of the MCP servers for different languages and tools.
    *   `./policy/`: OPA policies and JSON schemas that define the project\'s guardrails.
    *   `./scripts/`: A collection of PowerShell and Python scripts for validation, auditing, and other tasks.
    *   `./tools/`: Configuration files and scripts for development tools like `PSScriptAnalyzer`, `ruff`, and the main `Verify.ps1` script.
*   `./PIPELINE_MODS/`: Placeholder directories for the 24 modular components of the AI pipeline.
*   `./`: The root directory contains high-level planning documents like `MASTER PLAN_...` and `quick_start_guide.md`.

## Getting Started

To set up a local development environment, follow these steps:

1.  **Install Dependencies:**
    *   PowerShell 7.4+
    *   Python 3.12+
    *   Node.js (for TypeScript tooling)
2.  **Install PowerShell Modules:**
    ```powershell
    Install-Module -Name InvokeBuild, PSScriptAnalyzer, Pester -Scope CurrentUser -Force
    ```
3.  **Install Python Packages:**
    ```bash
    pip install ruff pyright pytest invoke watchdog
    ```
4.  **Initialize the MCP Environment:**
    ```powershell
    ./SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-/.mcp/Initialize-McpEnvironment.ps1
    ```
5.  **Verify the Environment:**
    ```powershell
    ./SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-/.mcp/Test-McpEnvironment.ps1
    ```

## Building and Running

This project uses a combination of scripts and task runners for building, testing, and validation.

*   **Local Validation:** The primary entry point for running all local checks is the `Verify.ps1` script.
    ```powershell
    ./SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-/tools/Verify.ps1
    ```
*   **CI/CD:** The `.github/workflows/powershell-verify.yml` workflow is the main CI pipeline for PowerShell code. It is triggered on every push and pull request.
*   **Task Runners:** The project is set up to use both `Invoke-Build` (PowerShell) and `invoke` (Python) for running tasks. See the `quick_start_guide.md` for detailed examples.
    *   **PowerShell (Invoke-Build):**
        ```powershell
        # In the 'watcher' directory from the quick start guide
        # Check a single file
        Invoke-Build check.one -Path path/to/your/file.ps1

        # Run checks on recently modified files
        Invoke-Build dev
        ```
*   **File Watcher:** A file watcher is available to automatically run checks whenever a file is saved.
    ```powershell
    # In the 'watcher' directory from the quick start guide
    ./watch.ps1
    ```

## Development Conventions

*   **Strict Naming Conventions:** The project plans to use a "Two-ID" naming system for modules and files, which will be enforced by a `nameguard` tool.
*   **Code Style:**
    *   **PowerShell:** Code style is defined in `tools/PSScriptAnalyzerSettings.psd1`. Key rules include using `Verb-Noun` naming, 4-space indentation, and a max line length of 140 characters.
    *   **Python:** Code style is enforced by `ruff`, with configuration in `tools/ruff.toml`.
*   **Testing:** All code should be accompanied by tests. PowerShell tests use `Pester`, and Python tests use `pytest`.
*   **Pre-commit Hooks:** The project intends to use pre-commit hooks to automatically enforce conventions before code is committed.

## Key Files

*   `C:\Users\Richard Wilks\R_PIPELINE\MASTER PLAN_AI-Operated Pipeline with Streamlined Watcher_Two-ID Modular Structure.md`: The comprehensive plan for the entire project.
*   `C:\Users\Richard Wilks\R_PIPELINE\quick_start_guide.md`: A guide to quickly setting up a streamlined file-watcher workflow.
*   `C:\Users\Richard Wilks\R_PIPELINE\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\README.md`: The main README for the `SPEC-1` project.
*   `C:\Users\Richard Wilks\R_PIPELINE\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\docs\ARCHITECTURE.md`: A detailed explanation of the system architecture.
*   `C:\Users\Richard Wilks\R_PIPELINE\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\.github\workflows\powershell-verify.yml`: The main CI workflow for PowerShell.
*   `C:\Users\Richard Wilks\R_PIPELINE\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\tools\Verify.ps1`: The main script for local validation.
*   `C:\Users\Richard Wilks\R_PIPELINE\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\tools\PSScriptAnalyzerSettings.psd1`: PowerShell code style rules.
*   `C:\Users\Richard Wilks\R_PIPELINE\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\SPEC-1-AI-Upkeep-Suite-v2-Guardrails-MCP-\tools\ruff.toml`: Python code style rules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DICKY1987)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DICKY1987)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
