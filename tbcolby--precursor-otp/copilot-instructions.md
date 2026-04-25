## precursor-otp

> - 7-state machine: PadList, GeneratePad, ViewPad, Encrypt, Decrypt, Result, ConfirmDel

# One-Time Pad — Build Notes

## Architecture
- 7-state machine: PadList, GeneratePad, ViewPad, Encrypt, Decrypt, Result, ConfirmDel
- XOR-based OTP with hex encoding for display
- TRNG bulk byte generation for key material
- Use-once tracking prevents pad reuse

## Key Patterns
**Bulk TRNG bytes** — rng.bytes(n) pulls N bytes efficiently from u32 TRNG calls
**Hex encoding** — bytes_to_hex/hex_to_bytes for ciphertext transcription
**Pad lifecycle** — FRESH → USED after encryption, destroyable at any time
**Input filtering** — decrypt mode only accepts hex chars (0-9, A-F)

## Build
```bash
cargo build -p otp --target riscv32imac-unknown-xous-elf
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tbcolby) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
