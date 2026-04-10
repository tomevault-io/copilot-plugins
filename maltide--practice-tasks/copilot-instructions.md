## practice-tasks

> This file provides guidance to agents when working with code in this repository.

# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Testing Commands
- Single test: `go test ./sprints/sprint-01-find-smallest/ -run TestFindSmallestIntInMap`
- All tests: `go test ./...`
- Progress script: `go run scripts/progress.go` (updates README statistics)

## Project-Specific Conventions
- **Package naming**: ALL source files use `package problems` regardless of directory name
- **Import aliases**: Sprint tests use descriptive aliases like `findsmallestmap`, `basicsslice` when importing from subdirectories
- **Empty input handling**: Empty maps/slices return 0 (not error), following project convention in existing solutions
- **Test organization**: Tests are at sprint level (`sprint_test.go`), not individual exercise level
- **Mixed languages**: Comments mix Russian (function descriptions) and English (code comments)

## Directory Structure
- Sprint-based organization: `sprints/sprint-XX-name/category/exercise.go`
- Each sprint has single test file importing from all subdirectories
- Import paths use full module: `github.com/Maltide/Practice-Tasks/sprints/sprint-XX-name/subdir`

## Code Style
- Table-driven tests with comprehensive edge cases (empty inputs, negatives, single items)
- Functions have detailed Russian comments explaining algorithm and examples
- Use `math.MaxInt` for finding minimums in non-empty collections

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Maltide)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Maltide)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
