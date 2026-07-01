## zbc

> provides it even under `ZBC_NO_STDINT`). There is no portability

# ZBC Engineering Principles

## Width neutrality is the product

ZBC exists so that one protocol serves any CPU — 8-bit through 128-bit
and beyond, any pointer width, any file size. Therefore:

**Any part of this system that assumes a specific architectural bit
width, file size, or address width is an anti-feature and must be
stamped out at all costs.**

What this means in practice:

- The only legitimate sources of a width are: (a) a wire format's own
  field definition (a 9P offset is 64 bits, a RIFF chunk size is 32
  bits, a virtio descriptor address is 64 bits — because their specs
  say so), and (b) the guest's *declared* parameters (`int_size`,
  `ptr_size` from CNFG or platform config). Never the implementer's
  habits, never "32 bits is plenty", never the build machine's word
  size.
- Internal state that mirrors a wire field carries the **full wire
  width**. Narrowing between the wire and internal bookkeeping (e.g.
  tracking a 64-bit file offset in a `uint32_t`) silently caps what the
  protocol can do and is exactly the failure mode this rule exists to
  prevent. A 16-bit guest streaming sequentially through a file larger
  than its own address space is a *core use case*, not an edge case.
- `uint64_t` is always available in this codebase (`zbc_protocol.h`
  provides it even under `ZBC_NO_STDINT`). There is no portability
  excuse for narrowing.
- Where an ABI boundary genuinely limits width (a RETN result is
  `int_size` bytes because the spec says so), the limit belongs to that
  boundary alone — it must not leak inward and shrink transports,
  drivers, or state machines behind it.
- Shifts wider than a type's guaranteed width are not a reason to
  narrow: split values or use defined double-shifts
  (`(x >> 16) >> 16`), as the existing code does.

If honoring this rule appears impossible somewhere, that is a design
question for the maintainer — not a license to pick a width.

## Other repo conventions

- C sources are C90, zero heap allocation (statically checked in
  `test/CMakeLists.txt`); callers provide all buffers.
- Every external symbol and every header-visible macro/type is
  `zbc_`/`ZBC_`-prefixed; static helpers are kept globally unique
  across library sources so amalgamated builds stay safe.
- The wire format's single source of truth is
  `docs/source/specification.rst`; shared wire constants live in
  `include/shared/` so implementations cannot drift.
- Nothing in the libraries may hang: probe failures and unsupported
  operations fail deterministically (`-1` / `ENOSYS`), never spin.

---
> Source: [johnwbyrd/zbc](https://github.com/johnwbyrd/zbc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
