## ivg

> To run the test suite use the helper script with up to ten minutes allowed for execution:

# Repository Guidelines

To run the test suite use the helper script with up to ten minutes allowed for execution:

```bash
timeout 600 ./build.sh
```

Always execute this command before committing changes to verify that the build and regression tests succeed.

A complete run ends with the line:

```
=== ALL BUILDS AND TESTS COMPLETED SUCCESSFULLY ===
```

If this message does not appear, the build has not finished correctly.

## Repository layout
The project uses a consistent folder structure. Build output is written to `output/` and no source files live there. Useful locations:

- `tools/` – scripts for building and maintaining the code and documentation.
- `projects/` – Xcode and Visual Studio project files.
- `docs/` – documentation.
- `externals/` - projects and source code from other repositories (only touch this content when explicitly asked to).
- `src/` – C++ source code for the library. The library is distributed as source rather than prebuilt binaries.
- `tests/` – regression tests.
- `examples/` – small sample programs.
- `benchmarks/` – JavaScript performance tests.
- `output/` – contains only build artifacts (and any runtime dependencies), no source files.

Root-level `build.sh` and `build.cmd` (mirrored implementations) should build and test both the beta and release targets.

BuildCpp.sh and BuildCpp.cmd are copied from another repository. Only make changes to them if there is no other solution.

## Formatting rules
Key style points:
- Tab characters for indentation, *not spaces*. A tab character equals four spaces.
- Opening braces stay on the same line as the control statement and closing braces are on their own line.
- Maximum line width is 120 characters. End-of-line comments may start at column 120.
- Line continuations should start with the operator and be indented two tabs from the original line.
- `#if`/`#endif` blocks should appear one tab *left* of the current indentation level.
- Class comment – put a plain C-style block comment immediately above each class, *not* Doxygen.  
	```
	/**
		One-sentence summary of what the class does.
		Extra details if truly needed.
	**/
	```
	* The two asterisks open/close the block; everything inside is indented with one tab.  
- Small method comment – use a single end-of-line comment:  
	void blahblah(int blah);	/// brief description of `blahblah`
- Inside comment text, wrap any variable, parameter, class or function names in back-ticks, e.g. `blah` is the temporary buffer.

When handling files with command-line tools:
- Always run `expand -t 4` on the file before processing.
- Always run `unexpand -t 4` on the file after processing.

## Script portability
All user-facing `.sh` and `.cmd` files must work when launched from any directory. They should start by changing
to their own folder (or the repository root) so that relative paths resolve correctly.

`.sh` scripts must be runnable without requiring `chmod +x`; always invoke them with `bash path/to/script.sh` (do
**not** rely on the system-default `sh`).  Each script must start with a portable she-bang:

```
#!/usr/bin/env bash
set -e -o pipefail -u
```

Every `.sh` script must have a corresponding `.cmd` implementation with identical behavior. Use `.cmd` files rather than `.bat`.

```
# example for a shell script
cd "$(dirname "$0")"/..
```

REM example for a .cmd script  
```
CD /D "%~dp0\.."
```

For robust error handling, `.sh` scripts should begin as shown above, and `.cmd`
scripts normally use a simple error check:

```
CALL buildAndTest.cmd %target% || GOTO error
EXIT /b 0
:error
EXIT /b %ERRORLEVEL%
```

---
> Source: [malstrom72/IVG](https://github.com/malstrom72/IVG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
