## hexcheck

> Use when configuring or running hexcheck in a Go repo for hexagonal architecture boundaries, adapter business-logic warnings, and mock discipline.


# Hexcheck

Configure `.hexcheck.yaml` by mapping repo paths to roles. Folder names do not matter; roles do.

Roles:
- `core`: domain/core business logic
- `usecase`: application use cases/orchestration
- `ports`: interfaces/contracts
- `adapter`: infra, persistence, external services, outbound adapters
- `entrypoint`: CLI/HTTP/UI/bootstrap
- `ignore`: generated code, mocks, vendor

Minimal shape:

```yaml
version: 1
components:
  core: { role: core, paths: [internal/domain/**, internal/core/**] }
  usecases: { role: usecase, paths: [internal/application/usecase/**, internal/application/usecases/**, internal/usecases/**] }
  ports: { role: ports, paths: [internal/application/port/**, internal/domain/repository/**] }
  adapters: { role: adapter, paths: [internal/infrastructure/**, internal/adapters/**] }
  entrypoints: { role: entrypoint, paths: [cmd/**] }
  generated: { role: ignore, paths: ['**/mocks/**', '**/generated/**', '**/*_templ.go', '**/*_gen.go'] }
```

For repos using `boundaries`, map by role, e.g. `boundaries/in -> entrypoint`, `boundaries/out -> adapter`, `boundaries/ports -> ports`.

Recommended rules:

```yaml
rules:
  no-adapter-imports-in-core: error
  no-infra-imports-in-usecase: error
  no-framework-types-in-core: error
  no-infra-types-in-ports: error
  no-adapter-to-adapter-imports: warn
  suspicious-business-logic-in-adapter: warn
  no-local-fakes-for-ports: warn
  missing-generated-mock-for-port: warn
  prefer-generated-mocks: warn
```

Business-logic mode:

```yaml
heuristics:
  businessLogicMode: audit # audit|ci
  businessLogicMinConfidence: medium # low|medium|high
```

Use `audit` for refactor discovery. Use `ci` when heuristic findings must stay high-confidence. Prefer tuning confidence before adding broad `excludePaths`.

Mock config:

```yaml
mocking:
  generatedMockPaths: [internal/mocks/**, internal/application/mocks/**, internal/application/port/mocks/**]
  generatedMockNamePatterns: ['Mock{{Interface}}', '{{Interface}}Mock']
```

Run:

```bash
hexcheck -hexcheck.config .hexcheck.yaml -hexcheck.root . ./...
```

Agent checklist: read repo architecture docs, map paths to roles, ignore generated/mocks, configure mock paths, run once, tune `ruleSettings.*.excludePaths` only after inspecting examples.

---
> Source: [bnema/hexcheck](https://github.com/bnema/hexcheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
