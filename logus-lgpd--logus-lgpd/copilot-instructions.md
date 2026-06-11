## logus-lgpd

> This repository is **logus-lgpd**: LGPD-compliant data privacy, PII masking,

# GitHub Copilot Instructions — logus-lgpd

This repository is **logus-lgpd**: LGPD-compliant data privacy, PII masking,
and AES-256-GCM encrypted storage for tabular data in Python.

## Import

```python
import logus as lg
import os
SALT = os.environ["LOGUS_SALT"]
KEY  = os.environ["LOGUS_KEY"]
```

## Core patterns

```python
df = lg.read("file.csv")                        # any format -> pl.DataFrame
df = lg.read("file.lgs", key=KEY)               # decrypt .lgs
df = lg.read("big.parquet", head=100_000)       # partial read, no OOM
df = lg.read(banco, "table")                    # from lg.db() connection
reports  = lg.scan(df)                          # detect PII columns
df_safe  = lg.mask(df, salt=SALT)               # mask PII, preserves type
lg.store(df_safe, "out.lgs", key=KEY)           # encrypt and save
result = lg.process("in.csv", salt=SALT, key=KEY, output="out.lgs")
```

## Rules

- NEVER open("file.lgs") -- use lg.read("file.lgs", key=KEY)
- NEVER same value for salt and key -- raises ValueError
- ALWAYS end lg.when() chains with .otherwise(default)
- lg.col IS polars.col -- all Polars methods work
- All manipulation functions preserve input type (pd->pd, pl->pl, lazy->lazy)

---
> Source: [logus-lgpd/logus-lgpd](https://github.com/logus-lgpd/logus-lgpd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
