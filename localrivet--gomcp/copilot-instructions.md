## be-very-stingy-with-locks

> When writing golang be very stingy and sparing with mutex locks. locks ofter produce more problems than not. where possible use atomics and avoid locks.

When writing golang be very stingy and sparing with mutex locks. locks ofter produce more problems than not. where possible use atomics and avoid locks.

Locks are hard to debug because child process could be trying to get a lock or release a lock of a parent lock.

it's better to avoid them with better designs.

---
> Source: [localrivet/gomcp](https://github.com/localrivet/gomcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
