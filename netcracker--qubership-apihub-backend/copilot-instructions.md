## sql-performance

> SQL performance review for repository changes


# SQL Performance

When adding or changing non-trivial SQL in repositories:

- Consider required indices for filters, joins, and sort columns.
- Avoid N+1 query patterns; prefer joins or batch loads where appropriate.
- Note expected cardinality (rows scanned/returned) for hot paths.
- Flag full table scans, missing indices, and unbounded result sets.
- Document performance assumptions in the PR or commit message when risk is non-obvious.

---
> Source: [Netcracker/qubership-apihub-backend](https://github.com/Netcracker/qubership-apihub-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
