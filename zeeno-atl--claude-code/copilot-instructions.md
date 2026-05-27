## claude-code

> - Build Docker image: `make all` or `make build`

# Claude Assistant Guide

## Build and Run Commands
- Build Docker image: `make all` or `make build`
- Run container with current directory: `make run` 
- Run container with specific directory: `make run-dir DIR=/path/to/directory`
- Manual container run: `docker run -it --rm -v "$(pwd):/app" claude-code`
- Install CLI locally: `npm install -g @anthropic-ai/claude-code`

## Directory Mapping
- Claude Code requires a directory to be mounted at `/app` inside the container
- The Makefile targets automatically map the current directory to `/app`
- To use a different directory, use `make run-dir DIR=/path/to/directory`

## Project Structure
- `Dockerfile`: Debian-based image with Node.js/npm
- `docker-entrypoint.sh`: Main container entry point
- `docker-entrypoint.d/*.sh`: Initialization scripts (installs Claude Code CLI)

## Code Style Guidelines
- **Language**: Bash scripting (shell scripts)
- **Formatting**: 2-space indentation, UNIX line endings
- **Error Handling**: Use `set -e` to stop on errors
- **Comments**: Document complex logic and script purpose
- **Naming**: Use descriptive lowercase filenames with appropriate extensions
- **Execution**: Ensure scripts have executable permissions

---
> Source: [Zeeno-atl/claude-code](https://github.com/Zeeno-atl/claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
