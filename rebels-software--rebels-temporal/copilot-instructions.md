## rebels-temporal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Rebels.Temporal** is a high-performance C# library for temporal event matching and correlation, designed for IoT, telemetry processing, and event-driven architectures. The library prioritizes:
- **Zero-allocation hot paths** for predictable, high-throughput performance
- **Allen's Interval Algebra** for temporal relations
- **Pure domain logic** with no external dependencies beyond .NET BCL
- **Flat namespace** (`Rebels.Temporal`) for simple API discovery

## Build and Test Commands

### Building
```bash
# Build entire solution
dotnet build

# Build in Release mode
dotnet build -c Release

# Build specific project
dotnet build src/Rebels.Temporal/Rebels.Temporal.csproj
```

### Testing
```bash
# Run all tests
dotnet test

# Run tests with filter (by namespace, class, or method name)
dotnet test --filter "MatchingTests"
dotnet test --filter "FullyQualifiedName~PointMatching"

# Run specific test method
dotnet test --filter "MethodName=Should_Match_Points_With_Tolerance"

# Run tests with detailed output
dotnet test -v detailed
```

### Benchmarks
```bash
# Run all benchmarks
cd benchmarks
dotnet run -c Release

# Run specific benchmark filter
dotnet run -c Release -- --filter *Sorted*
dotnet run -c Release -- --filter *PointMatching*
```

### Code Formatting
```bash
# Auto-format code following .NET standards
dotnet format
```

## Architecture

### Domain Model Philosophy

Rebels.Temporal is a **pure domain library**—it contains NO application layer, NO infrastructure layer, NO persistence, and NO external dependencies (ADR-4, ADR-7). The entire codebase exists in the domain layer.

### Core Concepts

The library defines four fundamental temporal concepts (see README.md domain model table):

1. **Temporal Event** (`ITemporalPoint`) - Point-in-time occurrences with a single timestamp
2. **Temporal Period/Interval** (`ITemporalInterval`) - Duration-based occurrences with Start and End
3. **Time Window** (`TimeWindow`) - Analytical time ranges for correlation (not domain occurrences)
4. **Temporal Relations** (`TemporalRelation`) - Allen's 13 interval algebra relations

### Fluent API Design

The primary API uses a fluent, zero-allocation interface accessed through `MatchTemporal`:

```csharp
// Pattern: MatchTemporal.<AnchorType>.With.<CandidateType>
MatchTemporal.Points.With.Points(...)      // Point → Point
MatchTemporal.Points.With.Intervals(...)   // Point → Interval
MatchTemporal.Intervals.With.Points(...)   // Interval → Point
MatchTemporal.Intervals.With.Intervals(...) // Interval → Interval (with Allen relations)
```

See `src/Rebels.Temporal/Matching/Execution/MatchTemporal.cs:21-170` for the fluent API structure.

### Visitor Pattern API (ADR-10)

Unlike typical .NET APIs, matching methods **do not allocate or return collections**. Callers provide a visitor implementing `IMatchVisitor<TAnchor, TCandidate>`:

```csharp
var buffer = new MatchPair<SensorReading, CommandEvent>[100];
var visitor = new BufferVisitor<SensorReading, CommandEvent>(buffer);

MatchTemporal.Points.With.Points(
    anchors, candidates, policy, ref visitor);

// Process results 0..visitor.MatchCount-1 in the buffer
// Also available: visitor.UnmatchedCount for observability
```

This enables zero-allocation hot paths (INV-3), gives callers full memory control, and provides built-in observability through `OnMatch` and `OnUnmatchedAnchor` callbacks.

### Algorithm Selection via InputOrdering

Performance depends critically on `InputOrdering` in `MatchPolicy`:

| InputOrdering | Algorithm | Complexity | When to Use |
|---------------|-----------|------------|-------------|
| `Both` | Dual-pointer scan | O(n+m) | Both collections sorted by timestamp (~255x faster) |
| `Candidates` | Binary search | O(n log m) | Only candidates sorted (e.g., from DB with ORDER BY) |
| `None` | Nested loops | O(n×m) | Unsorted data or very small datasets |

**Always prefer sorted data when possible.** The library validates declared ordering at runtime (INV-10).

### Project Structure

```
src/Rebels.Temporal/
├── Matching/
│   ├── Concepts/        # Core interfaces: ITemporalPoint, ITemporalInterval
│   ├── Execution/       # MatchTemporal (fluent API), IMatchVisitor, BufferVisitor, MatchPair
│   └── Policies/        # MatchPolicy, TimeTolerance, InputOrdering, AllowedRelations
tests/Rebels.Temporal.Tests/
├── Matching/            # Core matching tests
├── EdgeCases/           # Boundary conditions
└── Reference/           # Reference implementations
benchmarks/              # BenchmarkDotNet performance tests
docs/
├── adr/                 # Architecture Decision Records (16 ADRs)
└── invariants/          # Non-negotiable system rules (10 invariants)
```

**All public types live in a single namespace:** `Rebels.Temporal` (ADR-9, INV-4). Folder structure is for organization only.

## Critical Constraints (Invariants)

When writing code, these invariants **MUST NEVER be violated**:

### INV-1: Interval Start-End Constraint
All intervals must satisfy `Start <= End`. The library validates this at runtime.

### INV-2: DateTimeOffset Only
ALL temporal values must be `DateTimeOffset`. `DateTime` is forbidden everywhere (ADR-11). This prevents ambiguous timezone bugs.

### INV-3: No Allocations in Hot Path
Core matching algorithms must NOT allocate heap memory during execution. Forbidden:
- `new` for reference types
- Boxing value types
- LINQ operations
- `yield return`
- String operations in hot paths
- Collections (`List<T>`, `Dictionary<T>`)

Allocations in error/exception paths are permitted.

### INV-4: Single Namespace
All public types must be in `Rebels.Temporal` namespace. No sub-namespaces.

### INV-5: No External Dependencies
No NuGet packages beyond .NET BCL (ADR-7). Exception: test projects and benchmarks can use test/benchmark frameworks.

### INV-6: Allen Relations Exhaustive
Any two intervals relate by exactly one of the 13 Allen relations. See `MatchTemporal.DetermineAllenRelation()` at src/Rebels.Temporal/Matching/Execution/MatchTemporal.cs:499.

### INV-7: Single Anchor-Candidate Pair
Each matching operation operates on exactly one anchor type and one candidate type. Multi-source correlation requires multiple separate match invocations.

### INV-8: MatchPair Relation Consistency
`MatchPair.Relation` is **required** if and only if `MatchType == Interval`. Point-based matches do NOT include relation data.

### INV-9: TimeTolerance Non-Negative
Both `Before` and `After` components of `TimeTolerance` must be >= 0. Negative tolerance values are forbidden.

### INV-10: Input Ordering Validation
If `InputOrdering.Both` or `InputOrdering.Candidates` is declared, the library validates sorting at runtime and throws if violated.

## Testing Philosophy (ADR-13)

This project follows the **Chicago School (classicist)** testing approach:

- **Use real implementations** - no mocks for domain logic
- **Test behavior, not implementation** - assert on final state, not method calls
- **Minimal test doubles** - only for custom visitor implementations in tests
- **Refactoring-resistant** - tests survive internal restructuring

Example:
```csharp
// Good: Test with real data and assert results
var matches = new MatchPair<SensorReading, SensorReading>[100];
var visitor = new BufferVisitor<SensorReading, SensorReading>(matches);
MatchTemporal.Points.With.Points(anchors, candidates, policy, ref visitor);
Assert.Equal(expectedCount, visitor.MatchCount);
Assert.Equal(expectedValue, matches[0].Anchor.Value);

// Bad: Mock MatchTemporal or verify method calls
```

## Common Development Tasks

### Adding a New Matching Algorithm Variant

1. Add private static method in `MatchTemporal` class (src/Rebels.Temporal/Matching/Execution/MatchTemporal.cs)
2. Ensure zero allocations (use `ReadOnlySpan<T>`, no LINQ, no yield)
3. Use `DateTimeOffset` for all temporal values
4. Update algorithm selection logic in public API methods
5. Add tests in `tests/Rebels.Temporal.Tests/Matching/`
6. Add benchmark if performance-critical

### Adding a New Temporal Concept

1. Define interface in `src/Rebels.Temporal/Matching/Concepts/`
2. Use `namespace Rebels.Temporal;` (no sub-namespaces)
3. Ensure concept uses `DateTimeOffset`, not `DateTime`
4. Document ADR if introducing significant design decision
5. Add invariant if introducing non-negotiable constraint

### Working with ADRs and Invariants

- **ADRs** (Architecture Decision Records) explain "why" design choices were made
- **Invariants** are non-negotiable rules that must always hold
- Before making architectural changes, check if an ADR exists: `docs/adr/`
- Use `/why <question>` command to understand design decisions
- Violating an invariant requires core maintainer approval and explicit documentation

### Performance Considerations

From ADR-3 (Performance Design Principles):
- Prefer `ReadOnlySpan<T>` and `Span<T>` for data access
- Avoid LINQ in all performance-critical paths
- Prefer `struct` enumerators and minimal abstractions
- Document algorithmic complexity (O(n), O(n log n), etc.)
- Use BenchmarkDotNet to validate performance claims

## LLM Commands

This repository includes custom commands for AI assistants. See `docs/COMMANDS.md` for full documentation.

- `/init` - Load full project context (ADRs, invariants, source code)
- `/why <question>` - Explain design decisions (e.g., `/why no DateTime`)
- `/benchmark [filter]` - Run performance benchmarks

## Key Files to Read Before Contributing

1. `README.md` - Library overview and usage examples
2. `docs/adr/3-performance-design-principles.md` - Performance-first philosophy
3. `docs/invariants/README.md` - What are invariants and how to handle violations
4. `docs/invariants/3-no-allocations-in-hot-path.md` - Zero-allocation requirement
5. `docs/adr/13-chicago-style-unit-testing.md` - Testing approach
6. `src/Rebels.Temporal/Matching/Execution/MatchTemporal.cs` - Core matching API

## Notes for AI Assistants

- The codebase is intentionally small and focused - do not suggest adding application/infrastructure layers
- Performance is not premature optimization here - it's fundamental to the library's purpose (IoT/telemetry at scale)
- When suggesting changes, verify they respect all invariants
- The flat namespace is intentional - do not create sub-namespaces
- Visitor-based APIs may seem low-level, but they're required for zero-allocation guarantee and observability

---
> Source: [rebels-software/rebels-temporal](https://github.com/rebels-software/rebels-temporal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
