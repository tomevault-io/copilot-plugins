## aipark

> > The "how we work" brief for **AIPARK**. The stack, the architecture and the

# CLAUDE.md — Agent Operating Context

> The "how we work" brief for **AIPARK**. The stack, the architecture and the
> agent framework live in [sherpa](https://github.com/brij-coda/sherpa); this
> file holds only what is true of this product.

## What this is

A vehicle register for parking gates. Guards record entries and exits on a
phone; the register is an append-only event log and everything else is a
projection re-derived from it.

## ⚠️ The one architectural fact

`session` and `vehicle` are **projections**, not tables anyone writes to. They
are re-derived from `event` by `rebuildForPlates`. A correction is a new event
that supersedes an old one — never an edit. If you find yourself updating a
session row directly, you are working against the grain of the whole system.

## Auth

**Method:** Firebase phone OTP.

The provider proves the number once and we mint our own 90-day session JWT.
Match on a stable subject id, never a mutable handle — carriers recycle phone
numbers. No password auth. Local dev uses `/auth/dev-login`.

⚠️ Any `VITE_*` value is build-time and public. A provider secret never gets a
`VITE_` prefix.

## Gates — all three before a commit is green

```bash
cd backend  && deno task check
cd frontend && pnpm build
cd android  && ./gradlew :app:assembleFieldtest :app:testFieldtestUnitTest
```

And `deno task prove-gates` when you touch a gate. Sherpa's rule: you do not
know a check works until you have seen it fail.

⚠️ The Android tests were not on this list once, and four of them were red for
three releases because nobody ran them. The task is `testFieldtestUnitTest` —
`testFieldtestDebugUnitTest` does not exist and fails in a way that reads like a
Gradle problem rather than a typo.

## Migration before code

Drizzle's `select()` pulls every column, so a migration must reach an
environment before or with the code that reads it. That holds for a migration
that changes what an existing column *means*, too.

## The traps

**Room compares migration indices BY NAME.** An index created as
`idx_known_fold` is one Room did not expect, so it throws and the app dies on
launch — for everyone who already had it installed, while a fresh install works
perfectly. The convention is `index_<table>_<col>`. `NearMissTest` checks every
`CREATE INDEX` against it.

**The device and the server both fold plates, and the two folds must agree
character for character.** One extra rule on either side and every plate
containing that letter stops matching, silently. `PlateCandidatesTest` asserts
the device's set against the server's.

**A JDK that is too NEW breaks the build as obscurely as none at all.** Gradle
8.11 tops out at Java 23 and dies with `Could not determine java version from
'25.0.2'`, which reads like a corrupt toolchain. Use JDK 17.

---
> Source: [brij-coda/aipark](https://github.com/brij-coda/aipark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
