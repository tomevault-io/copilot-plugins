## laravel-ddd

> - Inspect the worktree before changing code. Existing code and tests are the

# Repository instructions

- Inspect the worktree before changing code. Existing code and tests are the
  source of truth.
- Read `docs/internal/STATUS.md` and `docs/internal/DECISIONS.md`.
- Preserve public command names and `Context:ClassName` syntax.
- Keep the new-installation defaults at project-root `contexts/` and namespace
  `Contexts`; preserve support for applications that configure another path
  and namespace.
- Reuse `ClassGenerator`, `ContextPathResolver`, and
  `ComponentPathResolver`; do not create parallel generator or path systems.
- Use the glossary in `docs/architecture.md`. Use Laravel Listener, Job,
  Mailable, Form Request, Resource, and Service Provider for framework
  adapters.
- Keep DTOs in Application. Keep Laravel Listeners, Jobs, and Mailables in
  Infrastructure.
- Keep Domain free of Laravel, Symfony, Application, Infrastructure, and
  Presentation, except that classes in `Domain/Events` may use Laravel's
  event `Dispatchable` trait. Do not create a generic `Domain/Services`
  directory.
- `ddd:make-context` always creates the five standard Domain directories.
  `--with-aggregate` may add only the empty same-name Aggregate and Repository,
  and Context force repair must never overwrite those business classes.
- Keep Context config at `contexts/{Context}/Config/config.php`, loaded by the
  root Context provider under the Context's snake-case key.
- Keep Laravel Listeners, Jobs, and Mailables directly in
  `Infrastructure/Listeners`, `Infrastructure/Jobs`, and
  `Infrastructure/Mail`. Listener discovery is automatic; providers must not
  maintain event-to-listener maps.
- Keep Context views and assets in lowercase `resources`, and Context Pest or
  PHPUnit suites in lowercase `tests`.
- `--with-vite` creates only `vite.config.js`, using root Node dependencies
  and Context-specific hot/build paths. Never create a Context
  `package.json`.
- Generate Context Service Providers only at
  `contexts/{Context}/Providers/{Context}ServiceProvider.php`. Legacy
  `Infrastructure/Providers` support is discovery-only, deprecated, and must
  never cause both providers to be registered.
- Keep repository documentation committed, but exclude `/docs` and other
  development-only material from Composer archives. Keep runtime source,
  package config, stubs, Composer metadata, license, and README.
- Update public docs after public behavior changes. Update `STATUS.md`,
  `HANDOFF.md`, `ROADMAP.md`, and `DEVELOPMENT_LOG.md` after meaningful work.
- Run Pint, PHPStan, Pest, and Testbench command discovery after changes.
- Never claim a test, command, or archive result unless it was run and
  inspected.

---
> Source: [ahmedsaed27/laravel-ddd](https://github.com/ahmedsaed27/laravel-ddd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
