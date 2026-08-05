## unodb

> - Repository: `unodb-dev/unodb`

# Agent Bootstrap — unodb

- Repository: `unodb-dev/unodb`

## Repository Orientation

At the start of every session, read:

1. This file
2. `CONTRIBUTING.md` — code quality standards and build instructions
3. `README.md` — project overview, API reference, build options

## Git Rules

- **Never amend commits** — always make new ones.
- **Never `git add -A`** — stage specific files.

## Code Quality Rules (from CONTRIBUTING.md)

- `const` everywhere possible (except by-value params and movable fields).
- `constexpr` everywhere it is legal.
- `[[nodiscard]]` on all value-returning functions by default.
- `noexcept` on everything that cannot throw.
- Pass by const reference for non-trivial types.
- All testable paths must be tested; LCOV_EXCL only for genuinely dead code.
- Doxygen comments on all declarations including private members.
- A clean CI run is a prerequisite for merging.

## Build Rules

### TSan and OLC Fields

**Never use `DISABLE_TSAN` or `__attribute__((no_sanitize("thread")))`.** The
OLC protocol uses optimistic reads (read without lock, validate version after).
TSan cannot model this and reports false races. The correct fix is to wrap the
field in the existing `in_critical_section<T>` template (relaxed atomics that
TSan understands) — NOT to suppress TSan.

Pattern:

- `in_critical_section<T>` (from `optimistic_lock.hpp`) — for OLC/concurrent use
- `in_fake_critical_section<T>` (from `in_fake_critical_section.hpp`) — for single-threaded `db`
- Both provide `load()` and `operator=(T)` with identical interfaces
- Template parameterize via `ArtPolicy::template critical_section_policy`

Example (value_bitmask_field):

```cpp
template <bool Enabled, class Storage, template <class> class CritSec>
struct value_bitmask_field {
  CritSec<Storage> bits{};  // NOT plain Storage
  void set(std::uint8_t i) noexcept {
    bits = static_cast<Storage>(bits.load() | (Storage{1} << i));
  }
};
```

For array-based fields, wrap each element: `std::array<CritSec<T>, N> bits{}`.

---
> Source: [unodb-dev/unodb](https://github.com/unodb-dev/unodb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
