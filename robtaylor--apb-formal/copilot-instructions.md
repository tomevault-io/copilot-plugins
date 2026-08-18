## apb-formal

> A formal protocol-compliance checker for the ARM AMBA APB bus (APB-lite + APB3),

# apb-formal — project conventions

A formal protocol-compliance checker for the ARM AMBA APB bus (APB-lite + APB3),
written in SystemVerilog and proven with SymbiYosys / Yosys.

## Orientation

- **What & why:** see `README.md` and `docs/adr/` (0001 toolchain, 0002 scope,
  0003 deliverable, 0004 waveforms).
- **Spec source of truth:** `docs/spec/IHI0024E.md` (machine-readable capture of the Arm
  spec), `docs/spec/property-catalog.md` (every protocol rule → spec citation → assertion),
  `docs/spec/waveforms/` (WaveDrom). The original PDF is **not** committed (licensing).
- **What's next:** `docs/plans/apb-formal-roadmap.md`.

## Toolchain

Yosys + SymbiYosys + SMT solver (boolector/yices). `nix develop` is the reproducible path;
`brew install yosys` is the lightweight local fallback. Proofs run via `make prove` /
`make cover` / `make all`.

## Working rules specific to this repo

- **Yosys does not parse concurrent SVA** (`property`/`sequence`/`bind`). Write checks as
  **immediate assertions** in a clocked `always` block using `$past/$rose/$stable/$fell`,
  and wire the checker to a DUT by **instantiation in an SBY wrapper**, not `bind`.
  (Rationale: ADR 0001.)
- Every assertion in `formal/fapb.sv` must trace to a row in `docs/spec/property-catalog.md`,
  which in turn cites a section/figure of IHI0024E. No orphan properties.
- A checker that can't fail on a known-bad design proves nothing: keep the negative test
  (the libfpga AHBL→APB bridge-bug reproducer) green-means-it-fails.

## Project memory discipline

This project uses the four-document discipline from
<https://robtaylor.github.io/claude-project-discipline/>:

| Doc kind | Lives in | Lifetime | Answers |
|---|---|---|---|
| **ADR** | `docs/adr/NNNN-*.md` | Forever; amended in place; never deleted | *Why* did we choose this? |
| **Plan** | `docs/plans/<topic>.md` | Long-lived; updated as work lands | *What's next, in what order?* |
| **Spike** | `docs/spikes/<topic>.md` | Forever, marked Resolved | *Did this idea work?* |
| **Handoff** | `docs/handoffs/<topic>-handoff.md` | Ephemeral — folded into the others, then deleted | *What's in flight now?* |

Load-bearing rule: **information has exactly one home, and the handoff is the only doc that
gets deleted.** Everything surviving a session migrates from the handoff into an ADR / plan /
spike / design doc / `CHANGELOG.md` *before* the handoff file is removed. The full migration
protocol and smell tests are in `docs/handoff-discipline.md`.

### Where things go

| About to write… | Belongs in… |
|---|---|
| "We chose X over Y because Z" | a new ADR (or an existing one's Decision/Consequences) |
| "What's next, in what order, with exit criteria" | the plan doc |
| "Validate <assumption> before committing to ADR NNNN" | a time-boxed spike |
| "What's in flight now / next session picks up" | a handoff |
| "How subsystem X works internally" | a design doc `docs/<topic>.md` |
| "What shipped in this version" | `CHANGELOG.md` |

---
> Source: [robtaylor/apb-formal](https://github.com/robtaylor/apb-formal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
