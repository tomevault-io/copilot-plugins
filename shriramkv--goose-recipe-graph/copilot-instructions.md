## goose-recipe-graph

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this is

A tool that maps and validates sub_recipe dependencies across a library of goose
recipes. It discovers recipe files under a directory, builds the dependency graph,
detects broken references and cycles, checks the values passed between recipes,
and renders the graph as Mermaid or DOT. Pure Python, standard library only for
the core. Read only against the recipes under test. No network access.

## Layout

- `goose_recipe_graph/loader.py` reads JSON and the YAML subset recipes use,
  including block scalars. Uses PyYAML when installed. Vendored from the sibling
  goose-recipe-lint project; keep the two in sync if you change parsing.
- `goose_recipe_graph/discover.py` finds and parses recipe files into Recipe
  objects (title, parameters, required keys, sub_recipe references).
- `goose_recipe_graph/graph.py` builds the graph, resolves sub_recipe paths,
  detects cycles, and runs the cross-recipe checks that produce findings.
- `goose_recipe_graph/render.py` renders Mermaid and Graphviz DOT.
- `goose_recipe_graph/report.py` renders console, JSON, and badge output.
- `goose_recipe_graph/model.py` holds the finding, result, and Recipe types.
- `goose_recipe_graph/cli.py` is the argparse entry point and CI gates.
- `samples/cookbook` is a clean library; `samples/broken` has a missing
  reference, a cycle, and a bad value key. Both are used by tests.

## Rules for changes

- Keep the core dependency free. PyYAML is an optional convenience, never required.
- sub_recipe paths resolve relative to the referencing recipe's own directory.
- A path that resolves on disk but sits outside the scanned set is 'external', not
  broken. Only a path that resolves to nothing is broken.
- Be tolerant of unknown recipe keys. Never raise on fields you do not recognise.
- Do not use em dashes in prose or comments.
- Run `python3 -m unittest discover -s tests` before finishing.

---
> Source: [shriramkv/goose-recipe-graph](https://github.com/shriramkv/goose-recipe-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
