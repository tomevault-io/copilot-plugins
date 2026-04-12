## go-base

> - Treat `pkg` as a **generic, reusable core library**: avoid importing application-specific code (`cmd/*`, service-specific business logic) into `pkg`.

## `pkg` design (core reusable package)
- Treat `pkg` as a **generic, reusable core library**: avoid importing application-specific code (`cmd/*`, service-specific business logic) into `pkg`.
- Keep **public APIs stable and minimal**: prefer small, composable interfaces (e.g. `logger.ILogger`, `pubsub.Interface`) and avoid leaking concrete types when not necessary.
- Use **generics** for app bootstrap only where it clearly improves ergonomics (e.g. `App[EnvInterface any]`), not for domain models.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luat111)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/luat111)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
