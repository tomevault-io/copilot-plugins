## cdc-connector

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# CDC Pipeline Project - CRITICAL INSTRUCTIONS

## 🚨 CDC PIPELINE ARCHITECTURE - READ THIS FIRST

**DATA FLOW:**
1. **Source Connector** registered first (captures binlog position)
2. **Python migration** (`migrate_v3.py`) loads initial data from MariaDB → PostgreSQL
3. **Sink Connector** registered (reads from earliest Kafka offset - ensures no data loss)
4. **CDC active**: Debezium streams binlog changes → Kafka → PostgreSQL

**CRITICAL RULES:**
- **Debezium hosted on Fly.io**: https://cdc-connector.fly.dev (NOT local)
- **Normal operation**: `snapshot.mode: "schema_only"` (source connector)
- **Adding new tables**: Set `snapshot.mode: "recovery"` temporarily, then revert
- **Removing tables**: Update topic prefix + schema history topic to force new binlog offset
- **Restart connectors**: Use `make restart-<database>-source/sink` (preserves state via PUT)
- **Initial data**: Use `migrate_v3.py`, NOT pgloader (legacy)

---

## 🚨 CURRENT PROJECT STRUCTURE

```
├── Makefile                              # Connector management commands
├── .env                                  # Environment variables (8 databases)
├── README.md                             # Complete user documentation
├── migrate_v3.py                         # Python migration script (CURRENT)
├── connectors/
│   ├── sources/mariadb/
│   │   ├── trading.json                  # Trading source connector
│   │   ├── finance.json                  # Finance source connector
│   │   ├── live.json                     # Live source connector
│   │   ├── chat.json                     # Chat source connector
│   │   ├── performance.json              # Performance source connector
│   │   ├── concontrol.json               # Concontrol source connector
│   │   ├── claim.json                    # Claim source connector
│   │   └── payment.json                  # Payment source connector
│   └── sinks/postgres/
│       ├── trading.json                  # Trading sink connector
│       ├── finance.json                  # Finance sink connector
│       ├── live.json                     # Live sink connector
│       ├── chat.json                     # Chat sink connector
│       ├── performance.json              # Performance sink connector
│       ├── concontrol.json               # Concontrol sink connector
│       ├── claim.json                    # Claim sink connector
│       └── payment.json                  # Payment sink connector
└── bootstrap/                            # LEGACY - pgloader (not used)
```

---

## Available Makefile Commands

**Registration:**
- `make register-<database>-source` - Register source connector
- `make register-<database>-sink` - Register sink connector

**Restart (Preserves Connection - Uses PUT):**
- `make restart-<database>-source` - Update source config without losing binlog position
- `make restart-<database>-sink` - Update sink config without losing Kafka offset

**Unregister:**
- `make unregister-<database>-source` - Delete source connector
- `make unregister-<database>-sink` - Delete sink connector

**Monitoring:**
- `make connectors` - List all connectors and their status
- `make connector-status C=<name>` - Get detailed connector status with error messages

**Available databases**: `trading`, `finance`, `live`, `chat`, `performance`, `concontrol`, `claim`, `payment`

**Example:**
```bash
make connector-status C=mariadb-trading-connector
make connector-status C=postgres-sink-concontrol
```

---

## Common User Workflows

### 1. NORMAL WORKFLOW - New Database Setup

When user wants to set up CDC for a new database:

**Step 1: Verify connector configs exist**
- Check `connectors/sources/mariadb/<database>.json` exists
- Check `connectors/sinks/postgres/<database>.json` exists
- Verify `snapshot.mode: "schema_only"` in source config

**Step 2: Suggest workflow**
```bash
# 1. Register source connector (establishes binlog position)
make register-<database>-source

# 2. Migrate initial data
python3 migrate_v3.py --database <database> --tables all

# 3. Register sink connector (reads from earliest Kafka offset)
make register-<database>-sink
```

**Step 3: Verify**
```bash
make connector-status C=mariadb-<database>-connector
make connector-status C=postgres-sink-<database>
```

---

### 2. ADDING NEW TABLES to Existing Database

When user adds a table to allowlist in `.env`:

**Step 1: Check current config**
- Read `connectors/sources/mariadb/<database>.json`
- Verify current `snapshot.mode` setting

**Step 2: Update source config**
- Change `snapshot.mode: "schema_only"` → `"recovery"`
- Verify `table.include.list` references the env var with new tables

**Step 3: Suggest workflow**
```bash
# 1. Migrate the new table
python3 migrate_v3.py --database <database> --tables T_NEW_TABLE

# 2. Restart connectors (preserves binlog position)
make restart-<database>-source
make restart-<database>-sink

# 3. Monitor for completion
make connector-status C=mariadb-<database>-connector

# 4. After snapshot completes, revert snapshot.mode to "schema_only"
# Edit connectors/sources/mariadb/<database>.json
# Then: make restart-<database>-source
```

**What to check:**
- Source connector should show "RUNNING" state
- Sink connector should show "RUNNING" state
- No errors in connector status

---

### 3. REMOVING TABLES from Allowlist

When user removes tables from allowlist:

**Step 1: Identify required changes**
- Schema history topic needs versioning (force new binlog offset)
- Topic prefix needs versioning (create new Kafka topics)
- Topics regex in sink needs updating

**Step 2: Update connector configs**

Source connector (`connectors/sources/mariadb/<database>.json`):
```json
{
  "config": {
    "topic.prefix": "xchange_trading_v2",
    "schema.history.internal.kafka.topic": "xchange_trading-v2.schema-history",
    "table.include.list": "${TRADING_TABLE_ALLOWLIST}"
  }
}
```

Sink connector (`connectors/sinks/postgres/<database>.json`):
```json
{
  "config": {
    "topics.regex": "xchange_trading_v2\\.xchange_trading\\..*"
  }
}
```

**Step 3: Suggest workflow**
```bash
# 1. Update .env to remove tables from allowlist
# 2. Edit source connector config (topic.prefix + schema.history topic)
# 3. Edit sink connector config (topics.regex)
# 4. Restart both connectors
make restart-<database>-source
make restart-<database>-sink
```

**What to check:**
- New Kafka topics with new prefix are created
- Old topics still exist (not deleted automatically)
- Connectors reading from correct topics

**Example reference**: `connectors/sources/mariadb/concontrol.json` uses `topic.prefix: "xchange_concontrol_v4"`

---

### 4. DEBUGGING Connector Issues

When user reports connector errors:

**Step 1: Check connector status**
```bash
make connectors                           # Overview of all connectors
make connector-status C=<connector-name>  # Detailed error messages
```

**Step 2: Common issues to look for**

**Type mismatch errors (bit(1) → boolean):**
- Check sink connector for Cast transform
- Should have: `"transforms": "route,castBits"`
- Should have: `"transforms.castBits.type": "org.apache.kafka.connect.transforms.Cast$Value"`
- Should have: `"transforms.castBits.spec": "active:boolean,deleted:boolean,..."`
- **IMPORTANT**: If `castBits` config exists but not in transforms list, transform won't apply

**Source connector not capturing changes:**
- Verify `snapshot.mode: "schema_only"` (unless adding new tables)
- Check binlog position hasn't expired (MariaDB retention)
- Verify tables are in allowlist env var

**Sink connector failing:**
- Check `topics.regex` matches source `topic.prefix`
- Verify primary keys exist in target PostgreSQL tables
- Check schema compatibility between Kafka and PostgreSQL

**Step 3: Suggest fixes**

For type mismatches:
```json
// Edit connectors/sinks/postgres/<database>.json
{
  "transforms": "route,castBits",  // Add castBits to the chain
  "transforms.castBits.type": "org.apache.kafka.connect.transforms.Cast$Value",
  "transforms.castBits.spec": "field1:boolean,field2:boolean"
}
```

Then restart:
```bash
make restart-<database>-sink
```

---

## Configuration File Locations & Settings

### Source Connector (`connectors/sources/mariadb/<database>.json`)

**Key settings:**
```json
{
  "name": "mariadb-<database>-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",

    // Database connection
    "database.hostname": "${<DATABASE>_HOST}",
    "database.user": "${<DATABASE>_USER}",
    "database.password": "${<DATABASE>_PASS}",

    // Snapshot mode
    "snapshot.mode": "schema_only",        // Normal operation
    "snapshot.mode": "recovery",           // When adding new tables

    // Table filtering
    "table.include.list": "${<DATABASE>_TABLE_ALLOWLIST}",

    // Kafka topics
    "topic.prefix": "xchange_<database>",
    "schema.history.internal.kafka.topic": "xchange_<database>.schema-history",

    // Type conversions
    "converters": "boolean",
    "boolean.type": "io.debezium.connector.mysql.converters.TinyIntOneToBooleanConverter"
  }
}
```

### Sink Connector (`connectors/sinks/postgres/<database>.json`)

**Key settings:**
```json
{
  "name": "postgres-sink-<database>",
  "config": {
    "connector.class": "io.debezium.connector.jdbc.JdbcSinkConnector",

    // Database connection
    "connection.url": "${SINK_DB_URL}&currentSchema=mp_cdc",
    "connection.username": "${SINK_DB_USER}",
    "connection.password": "${SINK_DB_PASSWORD}",

    // Topic matching
    "topics.regex": "xchange_<database>\\.<database>\\..*",

    // Upsert mode
    "insert.mode": "upsert",
    "primary.key.mode": "record_key",
    "delete.enabled": "true",

    // Schema evolution
    "schema.evolution": "basic",

    // Kafka consumer
    "consumer.override.auto.offset.reset": "earliest",

    // Transforms
    "transforms": "route,castBits",
    "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
    "transforms.route.regex": "xchange_<database>\\.<database>\\.(.*)",
    "transforms.route.replacement": "<schema>_$1",

    // Type casting (if needed for bit(1) fields)
    "transforms.castBits.type": "org.apache.kafka.connect.transforms.Cast$Value",
    "transforms.castBits.spec": "active:boolean,deleted:boolean"
  }
}
```

---

## When User Asks For...

### "Set up CDC for <database>"
1. Check if source/sink configs exist in `connectors/`
2. Suggest the 3-step workflow (register source → migrate → register sink)
3. Provide monitoring commands

### "Add a new table to <database>"
1. Check current `snapshot.mode` in source config
2. Suggest changing to `"recovery"`, migrating, restarting, then reverting
3. Remind to update `.env` allowlist first

### "Remove a table from <database>"
1. Explain need to version topic prefix and schema history
2. Show example changes to source and sink configs
3. Suggest restart workflow

### "Why is connector failing?"
1. Suggest `make connector-status C=<name>` to see error
2. Look for common patterns (type mismatch, topic regex, primary keys)
3. Suggest specific fixes based on error

### "How do I check connector health?"
1. `make connectors` for overview
2. `make connector-status C=<name>` for details
3. Check both source and sink connectors

---

## CRITICAL RULES - READ CAREFULLY

### 🚫 DO NOT:
1. **Suggest `make migrate`** - Legacy pgloader, use `migrate_v3.py` instead
2. **Change `snapshot.mode` permanently** - Only `"recovery"` temporarily for new tables
3. **Create new connector configs** - 8 databases already configured
4. **Suggest local Debezium** - Hosted on Fly.io
5. **Use DELETE+POST for restart** - Use `make restart-*` (preserves state via PUT)

### ✅ DO:
1. **Check README.md first** - Contains complete workflow documentation
2. **Use `make` commands** - Don't suggest raw curl commands
3. **Verify connector configs** - Read JSON files before suggesting changes
4. **Check transforms chain** - Ensure `castBits` is in `"transforms"` list if config exists
5. **Monitor connector status** - Use `make connector-status` to verify changes

---

## Environment Variables (FROM .env)

**8 MariaDB Source Databases:**
1. `trading` - xchange_trading
2. `finance` - xchange_finance
3. `live` - xchangelive
4. `chat` - xchange_chat
5. `performance` - xchange_performance
6. `concontrol` - xchange_concontrol
7. `claim` - xchange_claim
8. `payment` - xchange_payment

**PostgreSQL Target:**
- `SINK_DB_URL` - JDBC connection string
- `SINK_DB_USER` - Username
- `SINK_DB_PASSWORD` - Password
- Target schema: `mp_cdc`

**Kafka (Confluent Cloud):**
- `KAFKA_BOOTSTRAP_SERVERS`
- `CONFLUENT_API_KEY`
- `CONFLUENT_API_SECRET`

**Debezium:**
- `DEBEZIUM_URL=https://cdc-connector.fly.dev`

---

# Important Reminders

**File Creation Policy**:
- NEVER create files unless absolutely necessary for the goal
- ALWAYS prefer editing existing files over creating new ones
- NEVER proactively create documentation files (*.md) or README files unless explicitly requested
- **MANDATORY**: Check existing files/scripts/commands BEFORE creating anything new

**Development Policy**:
- Do what has been asked; nothing more, nothing less
- Use existing Makefile commands instead of raw docker/curl commands
- Check `.env` for all configuration values
- Check README.md for complete workflow documentation before suggesting workflows
- Verify connector configs before suggesting changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Container-xChange)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Container-xChange)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
