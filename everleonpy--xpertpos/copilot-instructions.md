## xpertpos

> ./scripts/setup-maven-local.sh

# x-pertPOS AGENTS.md

## Build & Run

```bash
# Setup Maven dependencies (requires JRE 8)
./scripts/setup-maven-local.sh

# Build
mvn clean package

# Run
./run.sh  # Linux
./run.bat # Windows
```

**Output JAR:** `target/x-pertpos-23.1.1-all.jar`

---

## Architecture

- **Entry point:** `pos.business.LogonWindowLauncher` (main)
- **Config:** `resources/Application.properties` → `resources/Company*.properties` (multi-tenant)
- **GUI:** Swing-based, 70+ window classes in `pos/gui/`
- **Data layer:** 60+ DAOs in `pos/dao/`
- **Models:** 120+ POJOs in `pos/pojos/`

**Multi-company support:** Companies 1-99 configured in `resources/Company*.properties`

---

## Critical Dependencies (Local Maven)

Install via `scripts/setup-maven-local.sh`:

| Artifact | Location |
|----------|----------|
| `local.oracle:classes12:12.0` | `lib/classes12.zip` |
| `local.oracle:ocijdbc:10` | `lib/ocijdbc.jar` |
| `local.javapos:javapos:1.13` | `lib/jpos113.jar` |
| `local.ncr:*:3.13` (3 artifacts) | `lib/NCR*.jar` |
| `local.rxtx:rxtxcomm:2.2` | `lib/RXTXcomm.jar` |

---

## System Requirements

- **Java:** 1.8 (JRE)
- **OS:** Windows/Linux/macOS (configured per company)
- **Database:** PostgreSQL (primary), Oracle (legacy)
- **Swing:** Requires display (NOT headless)

---

## Database

- **Driver:** PostgreSQL 9.4-1201-jdbc4
- **Connection:** Via Proxool connection pool
- **Schema:** 200+ tables (see `resources/*.sql` in scripts/)

---

## Special Notes

1. **App context** loads from property bundles: `Application`, `Company{ID}`, `{OS}` (Windows/Linux/MacOs)
2. **Single-instance enforcement** via `LockApp` (file-based mutex)
3. **No tests** defined in this codebase
4. **Legacy codebase** with 1109 Java files; no package restructuring yet
5. **Native libraries:** JavaPOS, NCR JavaPOS, RXTX serial comm

---

## File Organization

```
src/          # Source code (pos/ namespace)
resources/    # Properties, configs, icons, fonts
lib/          # Local jars for Maven install
scripts/      # Setup scripts, SQL upgrade files
target/       # Build output (ignored)
```

---

## Common Mistakes

- ❌ Running without `./scripts/setup-maven-local.sh` first (missing local deps)
- ❌ Missing `jre/` directory (app won't start from scripts)
- ❌ Assuming headless mode works (Swing GUI requires display)
- ❌ Ignoring company-specific config in `resources/Company{ID}.properties`
- ❌ Modifying legacy package structure without migration plan (see `PLAN_MEJORAS.md`)

---
> Source: [everleonpy/xpertpos](https://github.com/everleonpy/xpertpos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
