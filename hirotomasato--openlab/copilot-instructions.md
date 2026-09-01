## openlab

> > Account checker engine with YAML-based `.croot` config format. Developer: masanto.

# OPENLAB

> Account checker engine with YAML-based `.croot` config format. Developer: masanto.

## Architecture

```
main.py → load .croot → OpenBullet engine → blocks.Process()
```

## Key files

| File | Purpose |
|---|---|
| `main.py` | CLI entry — file picker, proxy, combo, Rich UI |
| `core/engine.py` | Runner — `load_blocks()`, `process()`, `run()` |
| `core/models.py` | `CVV`, `VariableList`, `BotData` |
| `core/enums.py` | All enums |
| `core/stats.py` | Thread-safe stats (checked, hits, fails, cpm) |
| `parsing/croot_parser.py` | YAML → Block objects |
| `parsing/variable_engine.py` | `<VAR>` substitution |
| `blocks/*.py` | Block types: request, keycheck, parse, function, utility |

## How to create a .croot config

1. Write YAML with `meta:` and `script:` sections
2. `script:` is an ordered list of blocks
3. Blocks execute top-to-bottom, engine stops on FAIL/BAN/ERROR
4. `<USER>`, `<PASS>`, `<SOURCE>` — built-in variables
5. Full spec: `docs/CONFIG_SPEC.md`

## Block flow

```yaml
script:
  - label: "LOGIN"          # REQUEST → POST login API
    type: request
    method: POST
    url: "https://..."
    body: '{"email":"<USER>","password":"<PASS>"}'

  - label: "CHECK"          # KEYCHECK → validate response
    type: keycheck
    chains:
      - type: Success
        keys: ['"token":"']
      - type: Failure
        keys: ["password wrong"]

  - label: "GRAB"           # PARSE → extract data
    type: parse
    parse_type: JSON
    field: "token"
    capture: true
```

## Environment

- Python 3.12+
- venv: `venv/bin/python`
- Dependencies: `rich`, `requests`, `pyyaml`, `jsonpath_ng`, `bs4`
- Install: `venv/bin/pip install -r requirements.txt`

## Running

```bash
venv/bin/python3 main.py
# → choose config (.croot)
# → choose proxy type
# → choose proxy file
# → choose combo file
```

## Testing a single account

```python
from core.engine import OpenBullet
from parsing.croot_parser import load_croot_blocks

blocks = load_croot_blocks('configs/deepseek.croot')
ob = OpenBullet(USER='user@mail.com', PASS='pass123', proxy=None)
ob.load_blocks(blocks)
result = ob.run()
print(result, ob.data.Variables.Captures())
```

## Files to ignore

- `tes.py`, `tg.py`, `chk.py` — reference scripts (deleted)
- `parsing/config_parser.py`, `parsing/line_parser.py` — legacy SVB parsers (unused)

---
> Source: [hirotomasato/openlab](https://github.com/hirotomasato/openlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
