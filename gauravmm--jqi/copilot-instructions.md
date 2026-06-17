## jqi

> Use this when the task is to understand unknown JSON well enough to write a correct `jq` filter, parser, transform, or extraction step.

# Skill: Understand Unfamiliar JSON Before Writing Queries or Parsers

Use this when the task is to understand unknown JSON well enough to write a correct `jq` filter, parser, transform, or extraction step.

Choose this skill when you need to:

- inspect a JSON file, JSON array, JSONL stream, or API response
- find real field paths, types, optional fields, or enum-like values
- detect structural splits such as records varying by `.type` or `.status`
- avoid guessing before writing code against the data

## Approach

Install `jqi`, run it on the data, then use the resulting structure summary to drive the next step.

```sh
TARGET=x86_64-unknown-linux-gnu
curl -L "https://github.com/gauravmm/jqi/releases/latest/download/jqi-${TARGET}.tar.gz" | tar xz
mv jqi /usr/local/bin/

jqi data.json
cat data.jsonl | jqi -
jq '.results[]' dump.json | jqi -
```

## What To Look For

- mixed types such as `int (30%) | float (70%)`
- optional fields shown as `(in 43%)`
- numeric ranges such as `min=1  max=500`
- enum-like values such as `"GET" | "POST" | "PUT"`
- string formats such as `uuid`, `iso8601`, `email`, `url`, `hex`, `base64`
- array shape and length ranges
- grouping by the best discriminant field
- deduped grouped output under `Common across all groups`

## Common Moves

If the input is a large nested object dump, extract the repeated subpath first instead of analyzing the whole outer wrapper:

```sh
jq '.items[]' dump.json | jqi -
```

```sh
# Machine-readable schema
jqi -j data.jsonl | jq '.fields[].path'

# Quick top-level shape only
jqi --nest-depth 1 data.jsonl

# Exact analysis instead of sampling
jqi --no-sample data.jsonl

# After inspection, write the real query
jq 'select(.status >= 400) | {ts: .timestamp, status, latency: .latency_ms}' data.jsonl
```

## Useful Flags

- `-j` for JSON output that can be piped into other tools
- `-q` for a quick type-only pass
- `--no-sample` to analyze all rows instead of a sample
- `--nest-depth N` to limit analysis depth when only outer structure matters

If the task needs an option not listed here, check `jqi --help`.

---
> Source: [gauravmm/jqi](https://github.com/gauravmm/jqi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
