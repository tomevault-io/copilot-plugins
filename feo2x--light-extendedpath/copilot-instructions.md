## light-extendedpath

> This library is about extending the existing `System.IO.Path` class in .NET. The `Path` class has hardcoded values for separator chars, file system rooting, case-sensitivity etc. While this makes the class easy to use, you cannot process Windows paths on UNIX. The overall goal is to provide a new class called `PathX` which has many of the `Path` methods, but provides their functionality in a host-agnostic way.


# Production Code Rules

This library is about extending the existing `System.IO.Path` class in .NET. The `Path` class has hardcoded values for separator chars, file system rooting, case-sensitivity etc. While this makes the class easy to use, you cannot process Windows paths on UNIX. The overall goal is to provide a new class called `PathX` which has many of the `Path` methods, but provides their functionality in a host-agnostic way.

## Principles

- Use rules-driven design (PathRules) instead of single separator parameters.
- Do not hardcode host-OS behavior in string logic; derive from PathRules.
- Prefer allocation-efficient APIs (ReadOnlySpan/Span, ValueStringBuilder).
- v1 is string-focused; avoid filesystem side effects unless explicitly requested.
- Document differences from System.IO.Path clearly (case sensitivity, root rules, device/UNC, trailing separators, full-path semantics).
- For guard clauses, use the assertions of Light.GuardClauses.

## Performance

- Provide span-based overloads when practical and reuse buffers with ValueStringBuilder.
- Avoid unnecessary allocations; prefer deterministic passes over strings, spans, memory or arrays.

## Documentation notes

- Write XML comments for all public members.
- State explicitly: PathX uses the provided rules (or preset), not the host OS, unless `Rules.Host` is chosen.
- Call out differences from System.IO.Path where behavior diverges (especially case, root semantics, normalization).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/feO2x) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
