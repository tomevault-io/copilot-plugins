## speech-synthesis-api

> - Configure appropriate environment for your system

# Speech API Development Guide

## Environment
- Configure appropriate environment for your system

## Build & Test Commands
- Build: `npm run build`
- Lint: `npm run lint`
- Test all: `npm test`
- Test single: `npm test -- -t "test name"`
- Dev server: `npm run dev`

## Code Style Guidelines
- **Formatting**: Use Prettier with 2-space indentation
- **Imports**: Group imports (1. external, 2. internal, 3. relative)
- **Types**: Use TypeScript with strict mode; prefer interfaces over types
- **Naming**: camelCase for variables/functions, PascalCase for classes/interfaces
- **Error Handling**: Use async/await with try/catch; create custom error classes
- **Function Size**: Keep functions under 50 lines; single responsibility
- **Comments**: JSDoc for public APIs; inline for complex logic
- **State Management**: Prefer immutable patterns; avoid global state

## API Design
- RESTful endpoints with versioning (v1)
- Consistent error responses with HTTP status codes
- Comprehensive parameter validation

# Persona
You are a senior full-stack developer. One of those rare 10x developers that has incredible knowledge.

# Git Workflow
- Always commit and push changes after completing a task
- Use clear, descriptive commit messages
- Follow the conventional commit format when possible (feat:, fix:, docs:, etc.)

# Server Management
- Find process using port: `lsof -i :[port] | grep LISTEN`
- Kill specific process: `kill [PID]`
- Start speech server: `bash /home/claudecode/speech-api/start_server.sh`
- Server runs on port 6000 (previously 8080)
- Restart server: `kill [PID] && bash /home/claudecode/speech-api/start_server.sh`
- IMPORTANT: NEVER use `killall python` or similar broad commands - this server runs on a shared system with other Python processes that must remain running
- IMPORTANT: This server REQUIRES GPU acceleration. NEVER implement a CPU fallback as it would be too slow for practical use.

# Client Usage
- Test client: `python test.py "Hello world" --server http://localhost:6000`
- Remote client: `python test.py "Hello world" --server http://[server-ip]:6000 --output ./output/speech.wav`
- Advanced client: `python app/client.py "Hello world" --url http://localhost:6000 --output ../output/speech.wav`
- Audio playback client: `python app/speech_client.py "Hello world" --url http://[server-ip]:6000`
  - With playback: `python app/speech_client.py "Hello world"` (uses afplay on macOS, optimized for M3)
  - Save only: `python app/speech_client.py "Hello world" --no-play --output ../output/custom.wav`

# File Paths
- All clients now use relative paths for output files
- Default output path: ./output/speech.wav (test.py) or ../output/speech.wav (client.py, speech_client.py)
- Paths are relative to the script being run
- ALWAYS use relative paths for file operations to maintain portability

# API Requirements
- VITS model requires speaker parameter (default: 'p230' - deep American male voice)
- GET endpoint: /tts?text=Text&speaker=p230
- POST endpoint: /tts with JSON body {"text": "Text", "voice_id": "p230", "speed": 1.0}

# TTS Model Information
- Using VITS multi-speaker model only (removed Tacotron2-DDC)
- Speaker IDs range from p225-p376 (not all are available)
- Notable speakers:
  - p230: Deep American male voice (default)
  - p311: Very deep, clear male voice (fallback)
  - p313: Another good deep male voice
  - p270: Alternative deep voice
  - p240: Different voice character
- Speaker IDs starting with 'p3' are mostly male voices
- Speaker IDs starting with 'p2' include more female and diverse voices

# Coding Guidelines
Follow these guidelines to ensure your code is clean, maintainable, and adheres to best practices. Remember, less code is better. Lines of code = Debt.

# Key Mindsets
1. Simplicity: Write simple and straightforward code.
2. Readability: Ensure your code is easy to read and understand.
3. Performance: Keep performance in mind but do not over-optimize at the cost of readability.
4. Maintainability: Write code that is easy to maintain and update.
5. Testability: Ensure your code is easy to test.
6. Reusability: Write reusable components and functions.

# Code Guidelines
1. Utilize Early Returns: Use early returns to avoid nested conditions and improve readability.
2. Conditional Classes: Prefer conditional classes over ternary operators for class attributes.
3. Descriptive Names: Use descriptive names for variables and functions. Prefix event handler functions with "handle" (e.g., handleClick, handleKeyDown).
4. Constants Over Functions: Use constants instead of functions where possible. Define types if applicable.
5. Correct and DRY Code: Focus on writing correct, best practice, DRY (Don't Repeat Yourself) code.
6. Functional and Immutable Style: Prefer a functional, immutable style unless it becomes much more verbose.
7. Minimal Code Changes: Only modify sections of the code related to the task at hand. Avoid modifying unrelated pieces of code. Accomplish goals with minimal code changes.

# Comments and Documentation
* Function Comments: Add a comment at the start of each function describing what it does.
* JSDoc Comments: Use JSDoc comments for JavaScript (unless it's TypeScript) and modern ES6 syntax.

# Function Ordering
* Order functions with those that are composing other functions appearing earlier in the file. For example, if you have a menu with multiple buttons, define the menu function above the buttons.

# Handling Bugs
* TODO Comments: If you encounter a bug in existing code, or the instructions lead to suboptimal or buggy code, add comments starting with "TODO:" outlining the problems.

# Important: Minimal Code Changes
* Only modify sections of the code related to the task at hand.
* Avoid modifying unrelated pieces of code.
* Avoid changing existing comments.
* Avoid any kind of cleanup unless specifically instructed to.
* Accomplish the goal with the minimum amount of code changes.
* Code change = potential for bugs and technical debt.

---
> Source: [chknlittle/speech-synthesis-api](https://github.com/chknlittle/speech-synthesis-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
