## endweave

> Bedrock protocol translation plugin (Endstone). Lets clients on protocol N talk

# endweave

Bedrock protocol translation plugin (Endstone). Lets clients on protocol N talk
to servers on protocol M by rewriting packets in flight.

## Sources of truth (in order)

1. **EndstoneMC/protocol-docs** -- one branch per BDS release (`r26_u1`,
   `r26_u2`, ...). Authoritative for wire-format JSON.
2. **Mojang/bedrock-protocol-docs** -- branches `r/26_u1`, `r/26_u2`. Useful
   `.dot` graphs and `json/` schemas; same data, slightly different shape.
3. **gophertunnel** -- handy for cross-checking field names and discovering
   what changed between versions, but **do not trust** for wire formats. It
   is reverse-engineered and has been wrong before (e.g. `ActionFlag`
   uvarint vs uint8 for LocatorBar; chemistry recipe `RecipeUnlockingRequirement`).
4. **CloudburstMC/Protocol** / pmmp -- same caveat as gophertunnel.
5. **bedrock-headers** at `LiteLDev/bedrock-headers` --
   reference only for enums/struct layouts. Never `#include` directly.

## Architecture

- `src/endstone_endweave/codec/` -- field types (`BOOL`, `UVAR_INT`, ...) and
  the `PacketWrapper` (`passthrough` / `read` / `write` / `passthrough_all`).
  Always go through the wrapper API in handlers; do not reach into
  `wrapper.reader` / `wrapper.writer` directly.
- `src/endstone_endweave/codec/types/enums.py` -- IntEnums mirroring BDS
  protocol enums. **Add new enums here** when a handler needs variant tags
  -- do not define ad-hoc `_PAYLOAD_X = 0` constants in handler files.
- When a handler grows local `_passthrough_foo(wrapper, ...)` helpers, the
  field is non-trivial enough to deserve its own `Type` subclass in
  `codec/types/`. Add the type, expose it as a singleton (e.g.
  `ENVIRONMENT_ATTRIBUTE_DATA`), and call `wrapper.passthrough(THE_TYPE)`.
- Names for new enums and types come **verbatim** from
  EndstoneMC/protocol-docs (`name` field on packets/types/enums). Don't
  invent shorter names or borrow gophertunnel/CloudburstMC names. Example:
  `EAS::AttributeLayerSettings` -> class `AttributeLayerSettings`,
  not `EnvAttrSettings`.
- `src/endstone_endweave/protocol/vAAA_to_vBBB/` -- one folder per direction.
  `protocol.py` registers handlers; `handlers/<packet>.py` does the rewrite.

## Wire-format conventions

- Game rule int values are `VAR_INT` in every version (not `UVAR_INT`).
- Recipe results: `NETWORK_ITEM_INSTANCE_DESCRIPTOR` (no `HasNetID`), not
  `ITEM_INSTANCE`.
- `ShapedChemistry` and `ShapelessChemistry` recipes do **not** carry
  `RecipeUnlockingRequirement` on the wire in v944. CloudburstMC was right
  here; gophertunnel and pmmp were wrong.
- "Optional[T]" in the JSON docs maps to: bool prefix + (T if true).

## Style

- No double-dashes (`--`) in user-facing prose. They look LLM-generated.
- No `Co-Authored-By: Claude` trailer on commits.
- Changelog entries: state what is broken and what is fixed, not why.
- Don't fabricate test fixtures from thin air -- prefer roundtripping a
  hand-built payload through the handler and checking byte deltas.

## Useful commands

```sh
uv run pytest -q                # full test suite
uv run python -c "..."          # ad-hoc roundtrip checks (not python3)
gh api repos/.../contents/...   # fetch protocol docs
```

---
> Source: [EndstoneMC/endweave](https://github.com/EndstoneMC/endweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
