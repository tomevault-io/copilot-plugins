## litebus

> Require XML docs on all LiteBus src constructs including private/internal


# XML documentation

When editing C# under `src/`:

1. Add or update `///` XML comments on every type, member, and field (including `private` and `internal`).
2. Follow `AGENTS.md` (summary indentation, `cref`, `param`, `returns`, `typeparam`, `value`).
3. Use `<inheritdoc />` for explicit interface implementations when the implemented member is already documented.
4. Run `dotnet build LiteBus.slnx` and fix SA16xx StyleCop documentation warnings before completing the task.

Do not skip documentation because a member is internal or private.

---
> Source: [litenova/LiteBus](https://github.com/litenova/LiteBus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
