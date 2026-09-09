## shdoc

> shdoc is a single-file GNU awk program that turns shell comment annotations

# AGENTS.md

shdoc is a single-file GNU awk program that turns shell comment annotations
into GitHub-flavored Markdown. Treat the awk script as the product: the
README, examples, and tests are fixtures that describe the public annotation
language.

## Architecture

The parser is an ordered set of awk pattern/action rules over one input stream.
Most state is global and represents the current pending documentation block:
file metadata, section metadata, a function description, collected tag entries,
and mode flags such as `in_description` and `in_example`. Rule order is part of
the design. Moving a rule can change whether a line is consumed as annotation,
function declaration, continuation text, or plain source.

Rendering is separate from recognition but still lives in the same file. The
`styles` table maps logical Markdown elements to regex substitutions, and the
render helpers turn the collected docblock into headings, lists, code fences,
links, and definition-list-like bullet output. Keep parsing changes and
rendering changes mentally separate when reviewing diffs.

A docblock is emitted only when the parser reaches a function declaration with
pending documentation. File and section descriptions are special: the same
`@description` syntax feeds file overview text before the first function,
section text after `@section`, and function text once a documented function
follows.
`@internal` suppresses the next documented function by setting transient state;
do not make it a persistent file-wide flag.

Function detection supports both one-line declarations and declarations where
`{` appears on a following line. The parser intentionally ignores function-like
text inside `@example` blocks. When adding declaration forms, preserve those two
contracts and add regression coverage for examples and split braces.

## Annotation contracts

`@arg` is reserved for positional arguments such as `$1` and `$@`. Malformed
`@arg` entries are processed through the option parser for backward
compatibility, so changes to `@option` validation can affect old `@arg` input.
The rendered argument order depends on zero-padded numeric keys and awk array
sorting.

`@stdin`, `@stdout`, and `@stderr` share continuation handling. Their following
indented comment lines append to the last item until indentation no longer
matches. `@example` uses different continuation rules and then unindents the
whole captured block before rendering a bash code fence.

`@see` entries are rendered as Markdown links, anchors, or literal Markdown
links depending on their text. Anchor formatting follows GitHub-style rules in
`render_toc_link`; update tests before changing that behavior.

## Tests and fixtures

Run the suite with:

```sh
make test
```

The test harness comes from the vendored `import.bash` submodule. If the harness
is missing, initialize submodules before testing:

```sh
git submodule update --init --recursive
```

Each file under `tests/testcases/` builds an input document, an expected
Markdown output, then calls the shared `assert` helper. Prefer focused fixtures
over checking generated output by hand. Issue-named fixtures are regression
tests; keep the original behavior clear when editing them.

Regenerate README examples with:

```sh
make examples
```

## Change guidance

Keep `shdoc` portable to GNU awk and shell-based workflows. Avoid introducing a
build step, a second runtime, or dependencies outside the existing submodule
unless the project direction changes explicitly.

When changing parser behavior, add a test that shows the exact annotation input
and full Markdown output. When changing documentation only, avoid rewriting the
large README examples unless the rendered output or annotation language actually
changed.

---
> Source: [reconquest/shdoc](https://github.com/reconquest/shdoc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
