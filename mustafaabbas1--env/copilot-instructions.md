## env

> This document provides context for AI agents and assistants to effectively assist with the development of this project.

# Agent Workspace Context

This document provides context for AI agents and assistants to effectively assist with the development of this project.

## Project Overview

This project provides a set of scripts to automate the setup of a development environment on a new machine. It is designed to be modular and extensible, allowing for the easy addition or removal of tools.

The main entry point is the `setup.sh` script in the root directory. This script is responsible for orchestrating the setup of all the tools defined in the `tools` directory.

## Project Structure

-   `setup.sh`: The main script that executes the setup for all the tools. It is designed to be run from any directory and will determine its own location to correctly reference the other scripts.
-   `tools/`: This directory contains subdirectories for each tool that can be installed.
    -   `tools/<tool_name>/setup.sh`: Each tool subdirectory contains a `setup.sh` script that is responsible for installing and configuring that specific tool. These scripts are designed to be idempotent, meaning they can be run multiple times without causing issues.
-   `changelog/`: This directory contains markdown files that document the changes made to the project. Each commit should have its own changelog file, named with the date and a short description of the change (e.g., `YYYY-MM-DDTHH:MM:SS_description.md`).
-   `AGENTS.md`: This file provides context to AI agents.

## Tool Setup Workflow

The `setup.sh` script uses a predefined array called `TOOL_ORDER` to determine the order in which the tools are installed. This ensures a consistent and predictable setup process. The script iterates through this array and executes the `setup.sh` script for each tool in the corresponding `tools/<tool_name>` directory.

The script also includes a verification step to ensure that all the tool directories specified in the `TOOL_ORDER` array exist before attempting to run their setup scripts. This prevents errors from missing directories.

## Development Guidelines

-   **Modularity:** When adding a new tool, create a new subdirectory in the `tools` directory and add a `setup.sh` script to it.
-   **Idempotency:** Ensure that all `setup.sh` scripts are idempotent.
-   **Clarity:** The `AGENTS.md` file should be kept up-to-date with any changes to the project structure or development workflow.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mustafaabbas1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mustafaabbas1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
