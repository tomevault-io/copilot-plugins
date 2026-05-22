## quack-jdbc

> Project notes for Claude and other contributors.

# CLAUDE.md

Project notes for Claude and other contributors.

## What this is

A JDBC driver for [DuckDB's Quack remote protocol](https://duckdb.org/docs/current/quack/overview).
Lets JVM tools (DBeaver, IntelliJ, dbt, Spark, etc.) connect to a remote
DuckDB server with a familiar `jdbc:quack://host:port?token=…` URL.

## Package layout

```
com.gizmodata.quack.jdbc
├── codec/        BinaryReader, BinaryWriter, HugeIntParts, QuackConstants
├── type/         LogicalType, LogicalTypeId, PhysicalType, ExtraTypeInfo, codec
├── message/      MessageType, MessageHeader, QuackMessage (sealed), MessageCodec,
│                 DataChunk, DecodedVector, VectorCodec
├── transport/    QuackUri (URL parser), QuackHttpTransport (java.net.http)
├── sql/          QuackDriver, QuackConnection, QuackStatement,
│                 QuackPreparedStatement, QuackResultSet,
│                 QuackResultSetMetaData, QuackDatabaseMetaData,
│                 SqlLiteral (parameter interpolation),
│                 SkeletalConnection/Statement/ResultSet (throws-by-default bases)
└── Quack*Exception   error hierarchy
```

The `codec`, `type`, `message`, and `transport` packages are reusable for
an ADBC driver (Go or otherwise). The `sql` package is the JDBC-specific
surface.

## Wire format reference

Wire-format details ported clean-room from
[`@quack-protocol/sdk`](https://github.com/tobilg/quack-protocol) by Tobi
(MIT). The TypeScript reference lives at `/tmp/quack-ref/` after running:

```bash
mkdir -p /tmp/quack-ref
for f in binary constants messages logical-types vector protocol values \
         client errors sql builders index; do
  gh api repos/tobilg/quack-protocol/contents/src/$f.ts --jq .content \
    | base64 -d > /tmp/quack-ref/$f.ts
done
```

## Build & test

```bash
mvn package          # build
mvn test             # unit + integration (needs duckdb on PATH)
mvn -Dtest='!Quack*IntegrationTest' test    # unit only
```

Integration tests spawn a real DuckDB CLI as a Quack server. They auto-skip
when `duckdb` isn't on PATH; override with `QUACK_IT_DUCKDB=/path/to/duckdb`.

## Pinned Quack version

duckdb-quack has no release tags as of 2026-05-13, so we pin to commit
`daae4826f57986fbb6cc2116316f89c673814b23` (2026-05-10, current main).
Update the pin when the upstream tags a release for DuckDB 2.0 (planned
September 2026).

## DBeaver metadata reference

`QuackDatabaseMetaData` mirrors DuckDB's own JDBC driver
(`org.duckdb.DuckDBDatabaseMetaData`) query-for-query so DBeaver sees the
same shape it does from a native DuckDB connection. When updating queries,
cross-reference DuckDB's `src/main/java/org/duckdb/DuckDBDatabaseMetaData.java`.

## Conventions

- Java 17 floor — broadest runtime compatibility (DBeaver ships with JRE 17).
  No pattern-matching switches; use `instanceof` chains.
- No runtime dependencies. JDK `java.net.http.HttpClient` for transport.
- No emoji in code, javadocs, or commit messages.
- Throw `SQLFeatureNotSupportedException` for unimplemented JDBC surface
  (see the `Skeletal*` base classes).
- DBeaver compatibility is a hard requirement — when changing
  `QuackDatabaseMetaData`, verify against DuckDB's reference before
  diverging.

## Release process

1. Verify `mvn test` is green on all targets in CI.
2. Bump `version` in `pom.xml`, update `CHANGELOG.md`, commit.
3. Tag `vX.Y.Z` and push the tag. CI publishes to Maven Central on tagged
   commits to `main`.

## Roadmap

- APPEND_REQUEST encoder + a streaming `appendChunk()` API
- FSST vector decoding
- Full `java.sql.Array` / `java.sql.Struct` wrapping for nested types
- Native PREPARE_REQUEST parameter binding once the protocol surfaces it
- Companion ADBC driver in Go (same wire codec layer ported)

---
> Source: [gizmodata/quack-jdbc](https://github.com/gizmodata/quack-jdbc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
