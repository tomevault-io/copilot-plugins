## synapse

> Associative structured local memory store. Use to remember and recall anything and retrieve linked memories.


# Synapse

Use the `synapse` CLI as a structured local memory and graph store.

## Core model

Synapse has three main concepts:

1. **Schemas**
   - runtime JSON Schemas
   - define the allowed structure of node payloads
   - archived schemas remain in history

2. **Nodes**
   - typed records validated against a schema
   - have payload, keywords, and archive state
   - payload updates replace the entire payload

3. **Links**
   - connections between node pairs
   - links are bidirectional
   - used for graph traversal

IDs are prefixed and human-readable:

- `schema_<...>`
- `node_<...>`
- `event_<...>`

## Command map

### Initialize

```bash
synapse init
```

Use `--db-path` to point at a non-default SQLite file:

```bash
synapse --db-path /path/to/synapse.db init
```

Default DB path is `.synapse/db`.

### Config

Read current config:

```bash
synapse config get
```

Update config from JSON argument or stdin:

```bash
cat config.json | synapse config update
```

Print config schema:

```bash
synapse config schema
```

### Schemas

Add a schema:

```bash
cat schema.json | synapse schemas add --name task
```

List schemas:

```bash
synapse schemas list
synapse schemas list --archived
```

Archive a schema:

```bash
synapse schemas archive <schema-id>
```

### Nodes

Add a node:

```bash
cat payload.json | synapse nodes add --schema-id <schema-id>
```

List nodes for a schema:

```bash
synapse nodes list --schema-id <schema-id>
synapse nodes list --schema-id <schema-id> --archived
```

Update a node payload:

```bash
cat payload.json | synapse nodes update <node-id>
```

Archive a node:

```bash
synapse nodes archive <node-id>
```

Search node IDs:

```bash
synapse nodes search '{"query":["keyword phrase"]}' --limit 20
```

### Keywords

Read node keywords:

```bash
synapse nodes keywords get <node-id>
```

Replace node keywords:

```bash
cat keywords.json | synapse nodes keywords update <node-id>
```

### Links and graph

Create or remove links:

```bash
synapse links add <from-node-id> <to-node-id>
synapse links remove <from-node-id> <to-node-id>
```

Traverse from explicit node IDs:

```bash
synapse graph get '["<node-id>"]' --depth 2 --breadth 10
```

Search first, then traverse:

```bash
synapse graph search '{"query":["keyword phrase"]}' --search-limit 5 --depth 2 --breadth 10
```

### Projections and replication

Run projections explicitly:

```bash
synapse projections run
```

Run replication explicitly:

```bash
synapse replication run
```

Restore from configured replicator into an empty event store:

```bash
synapse replication restore
```

## JSON input and query shapes

Commands that accept JSON can read it either:

- as a positional argument
- from `stdin`

Structured query commands use JSON too. `query` is always an array: each array element is OR-ed, and the space-separated words inside one element are AND-ed.

Examples:

```bash
synapse schemas add --name note '{"type":"object","properties":{"title":{"type":"string"}},"required":["title"]}'
synapse nodes add --schema-id "$SCHEMA_ID" '{"title":"Write README","status":"todo"}'
synapse nodes update "$NODE_ID" '{"title":"Write README","status":"done"}'
synapse nodes search '{"query":["readme"]}'
synapse nodes search '{"query":["readme docs","ops"]}'
synapse graph get '["node_01..."]'
synapse graph search '{"query":["engineering"]}' --depth 2
synapse graph search '{"query":["engineering sqlite","golang"]}' --depth 2
```

`stdin` form:

```bash
cat schema.json | synapse schemas add --name note
cat payload.json | synapse nodes add --schema-id "$SCHEMA_ID"
cat payload.json | synapse nodes update "$NODE_ID"
cat keywords.json | synapse nodes keywords update "$NODE_ID"
```

## jq workflow patterns

Use `jq` heavily. Synapse returns JSON, and updates are full replacement.

### Filter nodes

```bash
synapse nodes list --schema-id "$TASK_SCHEMA_ID" \
  | jq -r '.[] | select(.payload.status == "todo") | .payload.title'
```

### Patch-like payload update

Do not send partial payloads unless the partial object is valid as the full payload.

Preferred pattern:

```bash
synapse nodes list --schema-id "$TASK_SCHEMA_ID" \
  | jq -c --arg id "$NODE_ID" '
      .[]
      | select(.id == $id)
      | .payload
      | .status = "done"
    ' \
  | synapse nodes update "$NODE_ID"
```

### Append keywords safely

```bash
synapse nodes keywords get "$NODE_ID" \
  | jq -c '. + ["new-keyword"] | unique' \
  | synapse nodes keywords update "$NODE_ID"
```

### Select due tasks

If task payloads use ISO dates:

```bash
TODAY=$(date +%F)

synapse nodes list --schema-id "$TASK_SCHEMA_ID" \
  | jq -r --arg today "$TODAY" '
      .[]
      | select(.payload.status != "done")
      | select(.payload.due != null and .payload.due <= $today)
      | .id
    '
```

## Usage best practice
The guidelines in this section are IMPORTANT. Make sure to follow them and help your user to follow them as well.

### Schemas
Each schema should have a top-level description (JSON schema `description` field) of how its nodes can be used and what situations they can be used in. For example, for a task schema the description can be like this:
"Task tracks the state of a work in progress. Use whenever working on a long-running task or a follow-up. When the task is done, archive the node. To select all tasks that need to be reviewed, filter by 'check_after' ISO date field with jq."

Each field of the schema should also have a clear and concise description of how to use it.

Schema design should not imply hundreds of node updates because node aggregate will become slow. A task node that will get updated maybe a few dozen times maximum over its lifetime is fine. A single project node that can live for years and get hundreds of updates is bad. Instead, create separate nodes (e.g. tasks) and link them to the project.

Design schemas for future use. Schemas are immutable so make sure it is flexible enough but still concrete enough to be useful. Check with your user before creating a schema to make sure it's correct.

### Nodes
Don't be afraid to create many nodes. If you encounter something that fits a schema, create a node for it. Nodes are your memory. It's essential to document everything that happends so that a new session can remember important details and you become smarter with time.

Whenever you create a node, add 5-20 keywords for it depending on complexity. Do not add irrelevant keywords to avoid keyword polluting. Think about keywords that you would search with when looking for this node.

When you retrieve nodes, make use of `jq` and `--limit` to protect your context window from pollution. For example, don't fetch all tasks and then decide which ones to follow on. Filter by task status instead. If you only want to work on a single task, limit selection to one node. If you only need information from certain node fields, filter out irrelevant fields with `jq`.

### Links
Two nodes should be linked if they are likely to be useful together when retrieved from memory. For example:
- A task node may link to a learning node if that learning can help achieve that task.
- A person node may link to a place node if that person loves in that place.
- A project node can be linked to many task nodes if they move that project forwards.

What should not be linked:
- Two tasks just because they were created at the same time.
- Two learnings just because they help working with the same tool. Link them both to the tool node instead, otherwise you'll have to link n^2 learnings about each tool.

### Preferred style

Prefer simple shell pipelines over wrappers.

Good:
- `synapse ... | jq ...`
- JSON payloads via stdin
- explicit `--db-path` when scripting against multiple databases

Avoid:
- hidden partial updates
- assuming search is semantic/vector-based

### Sharp edges
- `nodes update` replaces the full payload
- `nodes keywords update` replaces the full keyword array
- archived nodes are excluded from normal graph traversal

---
> Source: [aromancev/synapse](https://github.com/aromancev/synapse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
