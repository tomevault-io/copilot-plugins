## cppsystemdesignnotes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a C++ knowledge base containing technical notes, reference implementations, and educational materials covering:
- Modern C++ features (C++11 through C++23)
- Audio programming and real-time systems
- STL containers and algorithms
- Memory management and function optimization
- System design patterns

All content is in Markdown format with occasional C++ code examples. This is primarily a documentation repository, not a software project.

## Repository Structure

### Core Directories

- **Modern C++/**: Comprehensive notes on modern C++ features organized by standard version (C++11, C++14, C++17, C++20, C++23). Each feature has its own markdown file with detailed explanations, code examples, and best practices. The overview file `現代 C++ 核心特性深度解析：從 C++11 到 C++23.md` provides a comprehensive synthesis.

- **Audio Programming/**: Deep-dive materials on low-latency audio processing in C++. Covers real-time programming constraints, lock-free algorithms, multi-buffering strategies (double/triple buffering), SIMD optimization, and producer-consumer patterns. Files are numbered (00-13) for sequential reading. The `Multi-Buffering/` subdirectory contains detailed implementation notes and a working triple buffer implementation in C++.

- **CppDesignConcepts/**: Core C++ design concepts and patterns, organized into specialized subdirectories:
  - **Algorithm/**: Structured algorithm reference following a consistent template (Core Concept, Use Cases, C++ Example, Complexity Analysis). Organized into subdirectories: `Sorting/`, `Searching/`, `Numeric/`, `String/`.
  - **Container/**: Notes on STL container usage patterns, particularly `std::vector` operations like `push_back` vs `emplace_back`, `resize` vs `assign`.
  - **Memory/**: Advanced memory management topics including placement new and custom memory handling.
  - **Function/**: Function-level optimization topics like tail call optimization.
  - Top-level files cover cross-cutting design concepts like aggregate initialization and random generation best practices.

- **Software Testing/**: Notes and resources related to software testing methodologies and practices.

- **Reading/**: Reference materials including books and papers on embedded systems design and software testing.

- **memory-bank/**: Project context management system (legacy). Contains historical project goals, decisions, and progress tracking. This directory tracks the evolution of the knowledge base and can be referenced for understanding the original design intent.

### Key Characteristics

- **Language**: Content is primarily in Traditional Chinese (繁體中文), with code examples and technical terms in English.
- **Format**: All documentation is in Markdown with embedded C++ code blocks.
- **No Build System**: This is a documentation repository with no compilation targets, tests, or build scripts.

## Project Goals and Context

Based on the memory-bank records, this repository aims to:
- Create a well-organized collection of notes on common C++ algorithms
- Focus on core concepts with concise explanations (not overly complex)
- Include practical examples frequently seen in contexts like LeetCode
- Cover modern C++ features from C++11 to C++23
- Document real-time audio programming patterns and constraints

### Potential Expansion Areas
- Graph algorithms
- String algorithms
- Dynamic Programming patterns
- Additional STL algorithms (std::find, std::reverse, std::unique, etc.)

## Working with This Repository

### Documentation Standards

**Writing Style - Succinct and Focused**:
- **Target length**: Keep documents under 150 lines when possible
- **Prioritize main concepts**: Focus on core ideas and design patterns, not exhaustive coverage
- **Minimal code examples**: Include only essential code to support concepts
- **Remove verbosity**: Eliminate redundant explanations, excessive examples, and tangential information
- **Playbook/mindset approach**: Documents should be quick reference guides, not textbooks

When creating or modifying algorithm notes in `CppDesignConcepts/Algorithm/`, follow this template structure:
1. **Core Concept**: A brief explanation of the algorithm
2. **Common Use Cases**: When to use it
3. **C++ Example**: A clear, concise C++ code snippet
4. **Complexity Analysis**: Time and space complexity in Big O notation

### Code Examples

- Use modern C++ (C++11 or later) in examples
- Include necessary headers explicitly
- Add clear comments explaining non-obvious behavior
- Follow the existing code style in similar files

### Real-Time Audio Programming Guidelines

When working with audio programming content, observe these critical constraints documented in the `Audio Programming/` materials:
- **Never** allocate/deallocate memory in real-time threads (no `new`, `delete`, `malloc`, `free`)
- **Never** use locks or mutexes in audio callbacks
- **Never** perform I/O operations in real-time contexts
- Use lock-free data structures for inter-thread communication
- Emphasize deterministic execution time over average-case performance

### Git Workflow

This repository uses simple git workflow:
- Main branch: `master`
- Recent commits show pattern: `category: brief description` (e.g., "vector: resize with default value", "audio programming: triple buffering concept")
- Keep commit messages concise and descriptive

### File Naming

- Use descriptive English names for new files in `CppDesignConcepts/Algorithm/`, `CppDesignConcepts/Container/`, `CppDesignConcepts/Memory/`, `CppDesignConcepts/Function/`
- Use Traditional Chinese for conceptual overview documents when continuing existing patterns
- Number files sequentially when order matters (as in `Audio Programming/`)
- Use hyphens or underscores for multi-word names consistently with existing files in that directory

---
> Source: [romanticamaj/CppSystemDesignNotes](https://github.com/romanticamaj/CppSystemDesignNotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
