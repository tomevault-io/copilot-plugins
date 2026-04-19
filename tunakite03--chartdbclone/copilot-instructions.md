## chartdbclone

> Performance guidelines

# Performance

- Correctness first. Optimize only when measured to be slow.
- Avoid N+1 queries. Batch database operations.
- Paginate list responses. No unbounded result sets.
- Lazy load routes and heavy components.
- Debounce user input handlers.
- Use connection pooling for databases.
- Stream large responses instead of buffering.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tunakite03) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
