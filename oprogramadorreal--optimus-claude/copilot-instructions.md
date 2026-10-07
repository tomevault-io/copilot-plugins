## optimus-claude

> This directory belongs to invoice-ledger.

# Partner feed importer

This directory belongs to invoice-ledger.

Run `python -m unittest discover -s ../tests -p test_importer.py` here.

The partner pads IDs with spaces in transit. Strip this padding only at the
import boundary; preserve case and do not broaden the rule to catalog lookup.
See `../.claude/docs/testing.md` when changing tests.

---
> Source: [oprogramadorreal/optimus-claude](https://github.com/oprogramadorreal/optimus-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
