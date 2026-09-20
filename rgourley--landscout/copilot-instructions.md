## landscout

> Due diligence for raw land. APN in, sourced report out.

# parcel-diligence

Due diligence for raw land. APN in, sourced report out.

## Setup

```bash
pip install -e '.[dev]'
pytest        # 64 tests, none touch the network
```

## Non-obvious things

**Facts are three-valued.** `facts.py` — a value, `UNKNOWN` (looked, couldn't
tell), or `UNAVAILABLE` (couldn't look). `Missing.__bool__` raises on purpose:
if you find yourself wanting `if fact.value:` you are about to collapse unknown
into false, which is the bug the whole design exists to prevent.

**Rule conditions are parsed, not eval'd.** `rules/expr.py` walks an AST over a
whitelist. Adding a helper means adding it to `_FUNCTIONS`, not reaching for
`eval`. Kleene logic throughout: unknown propagates, but a confirmed false still
settles an `and`.

**Providers report, rules interpret.** A provider never decides what a value
means — that lives in `jurisdictions/*.yaml`. Adding a state should be adding a
file. If you're editing a provider to support a jurisdiction, something is wrong.

**Every flag needs a `source`.** Enforced in `rules/loader.py`; an uncited flag
fails to load. Don't work around it.

**Sources are cached by request hash.** `PD_MODE=live|cached|replay`
(`http.py`). Tests run under `replay`, so they never reach the network — if a
test starts making requests, that's a bug in the test.

`PD_RULES_DIR` / `PD_DATA_DIR` relocate the YAML; they default to this checkout.

## State

Providers were written against each service's documented response contract and
tested against stubbed payloads, but **no request has hit a live endpoint** —
this was built where egress was blocked. County parcel endpoints in
`data/counties.yaml` are unconfirmed guesses (`verified: false`), and rule
content is encoded from the project brief rather than read from the primary
statutes (`verification: unverified` in each pack). Both surface in the report
rather than being hidden. Verify before trusting output for a real transaction.

See README.md § "Status, honestly".

---
> Source: [rgourley/landscout](https://github.com/rgourley/landscout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
