## hibit-ecs

> Prefer `ecs_serializer_write_bits(s, value, n_bits)` / `ecs_deserializer_read_bits(d, n_bits)` when:

# Project conventions

## Serializer

Prefer `ecs_serializer_write_bits(s, value, n_bits)` / `ecs_deserializer_read_bits(d, n_bits)` when:
- payload size is fixed at compile time
- payload is `<= 64 bits`
- caller does not require byte alignment after the write

`write_bytes` / `read_bytes` carry overhead (alignment fixup + memcpy machinery) that is wasted on tiny fixed-size fields. Using `write_bits` for single-byte headers (version, flags) also avoids GCC `-Wstringop-overread` false positives on stack-local 1-byte sources passed to the inlined memcpy path.

---
> Source: [dungeon2567/hibit_ecs](https://github.com/dungeon2567/hibit_ecs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
