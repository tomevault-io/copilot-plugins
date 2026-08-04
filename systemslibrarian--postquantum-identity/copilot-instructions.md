## postquantum-identity

> Conventions and guardrails for working in this repository. Read before making

# CLAUDE.md — PostQuantum.Identity

Conventions and guardrails for working in this repository. Read before making
changes. This project follows the same discipline as the rest of the
PostQuantum.* family.

## What this is

A post-quantum integration for **ASP.NET Core Identity**. Two capabilities:

1. **Argon2id password hashing** — a secure-by-default `IPasswordHasher<TUser>`
   producing PHC strings, on `net8.0`/`net9.0`/`net10.0`. Argon2id comes from
   `Konscious.Security.Cryptography.Argon2`.
2. **Post-quantum hybrid token issuance** — turns an authenticated Identity user
   into a [PostQuantum.Jwt](https://github.com/systemslibrarian/postquantum-jwt)
   token (ML-DSA-65 signature, optional X-Wing encryption). **`.NET 10 only`**,
   gated behind `#if NET10_0_OR_GREATER` plus a conditional `PackageReference`.

## Engineering discipline

- **Honesty over polish.** If something is incomplete, unproven, or risky, say so
  — in code comments, `SECURITY.md`, and `KNOWN-GAPS.md`. Never overstate what the
  crypto provides. The tokens are non-interoperable; the package is preview; it is
  unaudited — all stated plainly.
- **Fail-closed, always.** A malformed stored hash never verifies. Token
  validation throws on any failure. No silent downgrade.
- **Don't roll your own crypto.** Argon2id from Konscious; ML-DSA/ML-KEM from the
  BCL via PostQuantum.Jwt; constant-time compare via `FixedTimeEquals`. If you
  reach for hand-written primitives, stop.
- **Native BCL first.** Prefer `System.Security.Cryptography`. A new third-party
  dependency needs a written justification in `SECURITY.md`.
- **Keep the surface small.** No speculative features. The Argon2id core stays
  dependency-light so the package does not require the sibling
  `argon2id-passwordhasher` package.

## Code conventions

- **Targets:** library = `net8.0;net9.0;net10.0`; sample = `net10.0`.
- **Nullable** and **implicit usings** on. `LangVersion` is `latest`.
- **Warnings:** compiler warnings are errors (`TreatWarningsAsErrors`); analyzer
  (`CAxxxx`) suggestions stay warnings (`CodeAnalysisTreatWarningsAsErrors=false`).
  Don't suppress an analyzer without a comment explaining why.
- **Public API is documented.** XML doc comments on every public member
  (`GenerateDocumentationFile` is on; CS1591 would otherwise fail the build).
- **Deterministic builds** are enabled repo-wide; don't add nondeterminism.
- **net10-only code** lives behind `#if NET10_0_OR_GREATER` and, for whole files,
  the entire file is wrapped so it compiles away cleanly on net8/net9.
- **Naming:** `Argon2id*` for the hashing surface; `PostQuantum*`/`PqJwt*` for the
  token surface; DI helpers in `PostQuantum.Identity.DependencyInjection`;
  internal helpers in `PostQuantum.Identity.Internal`.
- **Secrets:** zero derived password bytes and candidate hashes with
  `CryptographicOperations.ZeroMemory`.

## Layout

```
src/PostQuantum.Identity/
  Argon2idOptions / VerifyResult / Argon2idPasswordHasher(+<TUser>)   core hashing
  Internal/PhcString.cs                                              PHC parse/format
  Tokens/                                                            net10 token service
  DependencyInjection/                                              Identity builder + DI
tests/PostQuantum.Identity.Tests/                                    xUnit tests
samples/PostQuantum.Identity.Demo/                                   minimal-API demo
docs/                                                                design notes
```

## Build & test

```bash
dotnet build
dotnet test
```

The token tests touch native ML-KEM/ML-DSA and **skip with a reason** when the
host lacks support. **In this dev container** the system OpenSSL (3.0.x) predates
ML-DSA, so run the full suite against conda's OpenSSL 3.5+:

```bash
LD_LIBRARY_PATH=/opt/conda/lib dotnet test
```

(This is an environment quirk, not a library requirement — the library only needs
*some* OpenSSL 3.5+.)

## Tests must stay honest

- A test that can't run its crypto should **skip with a reason**
  (`[SkippableFact]` + `Skip.IfNot(MLDsa.IsSupported, …)`), never silently pass.
- Keep the fail-closed tests: malformed PHC string, wrong password, wrong
  audience, tampered token.

## Faith statement

This project is built in gratitude to God. Documentation ends with:

> *To God be the glory — 1 Corinthians 10:31.*

Keep that footer on the README and the security docs.

---
> Source: [systemslibrarian/postquantum-identity](https://github.com/systemslibrarian/postquantum-identity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
