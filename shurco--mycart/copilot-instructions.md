## mycart

> PostgreSQL test databases, provisioned with

# internal/testutil/pgtest

PostgreSQL test databases, provisioned with
[pgtestdb](https://github.com/peterldowns/pgtestdb). `internal/testutil` and
`migrations/schema_conformance_test.go` both build on it, so there is one place
that knows how a test database is created.

## What it does

`MigratedDSN` / `FixturesDSN` / `EmptyDSN` each return the DSN of a database of
the caller's own, cloned from a template pgtestdb builds once per server:

| template | built by | contents |
| --- | --- | --- |
| plain | `goosemigrator` + `migrations.Embed()` | schema, empty tables |
| fixtures | the plain one, then the fixture set | schema + `fixtures/migration` |
| empty | `pgtestdb.NoopMigrator` | nothing, for tests of the migration path |

The migrations therefore run once per server — not once per test, and not once
per package, because the template survives in the server between test binaries.
Editing a migration or a fixture changes the template hash, so the next run
rebuilds it. A template is `datistemplate = true` and is never dropped by the
suite; to force a rebuild, `UPDATE pg_database SET datistemplate = false` and
`DROP DATABASE` it.

The suite connects through `database.Connect`, so every PostgreSQL test also
exercises the production session-timezone pin, the `SHOW timezone` check and the
pool sizing.

## TEST_POSTGRES_DSN

An **administrator** connection to a dedicated test server. pgtestdb creates the
role `pgtdbuser` (`NOSUPERUSER NOCREATEDB NOCREATEROLE`), one `testdb_tpl_*`
database per template and one `testdb_tpl_*_inst_*` database per test, and drops
the instances again. The user in the DSN needs `CREATEDB`, `CREATEROLE` and
`SUPERUSER`. **Never point it at a server holding data anybody wants to keep.**

Two constraints, both enforced with a message rather than a silent wrong
answer:

- The DSN must be a `postgres://` URL, and the database it names must already
  exist — it is only the connection pgtestdb administers the server through.
  `mycart_test` in the CI example is created by `POSTGRES_DB`; the local
  `pgtestdb` service in `docker/docker-compose_dev.yml` names the `postgres`
  maintenance database instead, which every server has.
- The session timezone is pinned to UTC here as well as in `database.Connect`,
  and the pin is **load-bearing**, not decoration: pgtestdb migrates the
  template on its own connection, and the fixtures are `INSERT`s with
  `DEFAULT CURRENT_TIMESTAMP`. Measured on a server running `Asia/Seoul`,
  dropping the pin stored the fixture template's timestamps `+32399s` out.

## Rules

- Never call `pgtestdb.New`/`Custom` directly. The config parsing, the timezone
  pin and the migrator choice live here so `EmptyDSN` and `FixturesDSN` cannot
  drift apart.
- Do not hard-code the template or instance names, and do not connect to a
  template database to poke at it from a test: ask for a DSN.
- A test that fails keeps its database on purpose — pgtestdb logs the connection
  string so the state can be inspected with `psql`. A test that passes has it
  dropped; a database that survives a green run means a leaked connection.

---
> Source: [shurco/mycart](https://github.com/shurco/mycart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
