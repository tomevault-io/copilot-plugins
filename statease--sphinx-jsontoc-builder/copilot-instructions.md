## sphinx-jsontoc-builder

> `sphinxcontrib-jsontoc` is a Sphinx extension that provides a custom builder (`jsontoc`) which extends the standard JSON HTML builder to include a `toctree` key in every `.fjson` output file. This key contains the full site-wide table of contents (similar to the ReadTheDocs HTML theme), as opposed to the default `toc` key which only has the current page's TOC.

# CLAUDE.md

## Project overview

`sphinxcontrib-jsontoc` is a Sphinx extension that provides a custom builder (`jsontoc`) which extends the standard JSON HTML builder to include a `toctree` key in every `.fjson` output file. This key contains the full site-wide table of contents (similar to the ReadTheDocs HTML theme), as opposed to the default `toc` key which only has the current page's TOC.

## Structure

- `sphinxcontrib/jsontoc/builders.py` — `JSONTOCHTMLBuilder`, the single builder class. Extends `JSONHTMLBuilder` from `sphinxcontrib.serializinghtml`, overrides `get_doc_context` to inject the full toctree.
- `sphinxcontrib/jsontoc/__init__.py` — Sphinx extension entry point (`setup(app)`), registers the builder.
- `sphinxcontrib/__init__.py` — Namespace package boilerplate.
- `setup.py` — Package metadata. Version is `0.01`, requires `Sphinx>=1.0`.

## Build and install

```
pip install -e .
```

## Usage

In a Sphinx project's `conf.py`:
```python
extensions = ['sphinxcontrib.jsontoc']
```

Then build with:
```
sphinx-build -b jsontoc -c . build/jsontoc
```

## Key dependencies

- `Sphinx` (>=1.0)
- `sphinxcontrib.serializinghtml` (provides `JSONHTMLBuilder`)

## Notes

- Apache 2.0 licensed.
- Uses the `sphinxcontrib` namespace package pattern.
- No tests currently exist in the repo.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/statease) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
