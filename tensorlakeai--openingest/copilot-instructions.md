## openingest

> Project conventions for AI assistants working in this repo.

# CLAUDE.md

Project conventions for AI assistants working in this repo.

## Deployment

The deploy entry `src/workflow.py` is **not** shipped via
PyPI. Always direct users to clone, not `pip install git+...`:

```bash
git clone https://github.com/tensorlakeai/openingest
cd openingest
pip install -e .
tl deploy src/workflow.py
```

The entry file must sit ONE LEVEL ABOVE the `tensorlake_docai/` package
(i.e. at `src/workflow.py`, **not** inside `src/tensorlake_docai/`).
`tl deploy` ships the directory containing the entry file as the zip
root, so placing `workflow.py` at `src/` keeps `tensorlake_docai/` as
a sibling and preserves the package name inside the executor — without
that, absolute imports like `from tensorlake_docai.vlm.cloud import ...`
inside bundled submodules fail with `ModuleNotFoundError`.

The SDK's recursive check (`import_file_path.startswith(code_dir_path)`)
still requires every `@function()`/`@application()` source file to live
under the entry file's directory; `src/` satisfies that because all
functions are defined inside `src/tensorlake_docai/...`. Don't move
`workflow.py` into an unrelated sibling folder.

## Contributions

- Small, focused PRs. One change per PR.
- Big refactors or new model integrations: open an issue first.

## Lint checks

After editing any Python file under `src/`, `tests/`, or `examples/`, run
both formatters before reporting the task complete:

```bash
black src/ tests/ examples/
ruff check src/ tests/ examples/
```

Fix any `ruff` errors before handing back. Don't silence rules with
`# noqa` unless the violation is intentional and worth a brief comment.

## Dev commands

```bash
pytest tests/ -q
black src/ tests/ examples/
ruff check src/ tests/ examples/
```

---
> Source: [tensorlakeai/OpenIngest](https://github.com/tensorlakeai/OpenIngest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
