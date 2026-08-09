## cssengine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

The entire codebase is `engine.py`, pure stdlib, Python 3.11. There is no README, no test suite, no build config, no dependency manifest, and no git repository. The `.venv` contains only pip/setuptools.

## Environment

The virtualenv lives at `.venv` (Windows layout, PowerShell is the primary shell):

```powershell
.venv\Scripts\Activate.ps1                     # activate
.venv\Scripts\python.exe -c "import engine"    # run without activating
```

There is no test runner installed. If tests are added, install pytest into `.venv` first (`.venv\Scripts\pip.exe install pytest`) and run a single test with `.venv\Scripts\python.exe -m pytest path::test_name`.

## Design intent

This is a CSS-*shaped* engine, not a CSS implementation. The engine has no built-in knowledge of any property. It never contains `if property == "margin"`. Its entire job is:

1. Parse selectors
2. Parse property names
3. Parse property values, by dispatching to a **registered** parser
4. Match selectors against nodes
5. Return a plain bag of properties

Given `margin: 2;` it produces `{"margin": 2}` and stops. It does not know what a margin is, and it has no layout, box model, flexbox, or grid. A consuming framework (Textual, say) decides what the values mean — `layout.spacing = css["margin"]` — or ignores them.

Semantics enter only through registration:

```python
engine.css_property("margin", parse_int)
engine.css_class("button", ...)
engine.css_pseudo("hover", ...)
```

A property with no registered parser is **not** an error and **not** a special case — it is reported as an unknown property. This is the load-bearing constraint of the project: when adding a feature, the test is whether the engine gained knowledge of a specific CSS concept. If it did, that knowledge belongs in a registered callable on the caller's side instead.

## Architecture

The pipeline is `parse()` → rules → `style()` → dict.

- `CSSNode` — a default node. Exposes read-only `tag`, `classes`, `parent`, `children`. Nothing requires it: matching is duck-typed on those four attributes, so a host framework's own widget classes are styleable without adapters. Keep that contract intact — `parent`/`children` just need to default to `None`/empty on anything that doesn't model a tree, which is why combinators quietly never match rather than erroring on such objects.
- `CSSSelector` — one compound clause (`tag`, `classes`, `pseudo_classes`, `pseudo_element`), each optional (`None` = "don't constrain on this axis"), plus an optional `combinator`/`left` pointing at the selector to its left in a chain (`a > b`, `a b`, `a + b`, `a ~ b`). A selector with no `left` is just a compound, exactly as before. `matches(obj)` alone is a purely structural match; `matches(obj, engine)` additionally consults registered matchers. Without an engine, a selector carrying pseudo-classes cannot be verified and returns `False`. `specificity` sums over the whole chain.
- `Rule` / `Declaration` — a selector (possibly a whole chain) plus its already-parsed values, tagged with source order.
- `Diagnostic` — `(kind, message, line)`. Kinds: `syntax`, `unknown-property`, `invalid-value`, `unknown-pseudo-class`, `unknown-pseudo-element`.
- `CSSEngine` — registration, parsing, matching. `_Source` is the cursor used by the hand-written recursive-descent parser.

Registration: `css_property(name, parser)`, `css_class(name, matcher)`, `css_pseudo_class(name, matcher)` (aliased `css_pseudo`), `css_pseudo_element(name, matcher=None)`. All accept a decorator form when the callable is omitted.

### Invariants worth preserving

- **Parsing never raises.** Malformed input produces diagnostics and the parser recovers (to the next `;` or `}`). A caller's value parser raising is caught and turned into an `invalid-value` diagnostic — a parser rejecting its input is not the engine's crash.
- **Values are parsed eagerly at `parse()` time**, so a property's parser must be registered *before* the stylesheet using it is parsed.
- **Registered classes are virtual.** `css_class("button", matcher)` makes `.button` ask the matcher instead of checking the node's set. Unregistered class names are *not* unknown — they fall back to set membership. This asymmetry is deliberate: pseudo-classes and pseudo-elements have no structural fallback, so unregistered ones are diagnosed and their rules dropped.
- **Cascade order** is specificity then source order, where specificity is `(classes + pseudo-classes, tag + pseudo-element)`. `!important` declarations are applied over all normal ones. This is the one place the engine owns policy, because it is about *rules*, not about any particular property.
- **Pseudo-elements get their own bag**: `style(node)` excludes them, `style(node, "before")` returns only theirs.
- **Combinators are relations, checked structurally, never registered.** `>`, ` ` (descendant), `+`, `~` walk `.parent`/`.children` on the node itself, not on the engine — there is no `css_combinator`. Only the rightmost compound is checked against the target node directly; earlier compounds in the chain are checked against ancestors/siblings found by walking the tree. Adjacency with no explicit combinator token means descendant, same as real CSS.

---
> Source: [youssefahmed2017/CSSEngine](https://github.com/youssefahmed2017/CSSEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
