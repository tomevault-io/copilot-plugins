## k7-tests

> K7 testing conventions (NUnit, FluentAssertions, NSubstitute)


# Testing

Stack: NUnit, FluentAssertions, NSubstitute. AAA structure.

Naming: `{ClassUnderTest}Tests`, `{Method}_Should{Expected}_When{Condition}`.

Functional: `CustomWebApplicationFactory`. Integration: Testcontainers + Respawn.
Critical Blazor components: bUnit in `tests/Clients.ComponentTests` (components live under `Clients/Shared/UI/Components/`).

---
> Source: [kaybi-gh/K7](https://github.com/kaybi-gh/K7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
