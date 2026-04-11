## cross-solve

> This document provides instructions for interacting with the `cross-solve` project using Gemini.

# Gemini Instructions for the cross-solve Project

This document provides instructions for interacting with the `cross-solve` project using Gemini.

## General Instructions

- When making changes to the codebase, please follow the existing coding style and conventions.
- After any major modification, please update the `Documentation.md` file to reflect the changes. This includes changes to the high-level overview, file-by-file documentation, and class/method descriptions.
- After any major modification, please update the `CHANGELOG.txt` file to reflect the changes (summary only).
- Do not stage changes for commit unless explicitly asked for.

## Building the Project

To build the project, you can use the following command. Remember to replace `/path/to/or-tools_install_dir` with the actual path to your OR-Tools installation.

```bash
mkdir build
cd build
cmake -DCMAKE_PREFIX_PATH=/path/to/or-tools_install_dir ..
make
```

## Running Tests

To build and run the tests, use the following commands:

```bash
mkdir build
cd build
cmake -DBUILD_TESTS=ON -DCMAKE_PREFIX_PATH=/path/to/or-tools_install_dir ..
make
ctest --output-on-failure
```

**Note:** The `ScipWorkerInjectionTest.SolutionInjection` test is expected to fail. This is a known issue with the test's threading model and can be safely ignored.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krooonal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/krooonal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
