## json-simple

> Guidance for Claude Code and other AI agents working in this repository. Humans

# Working agreement

Guidance for Claude Code and other AI agents working in this repository. Humans
should read it too — the constraints below are not AI-specific.

## What this project is

`json-simple` is a ~27 KB, dependency-free JSON encoder/decoder for Java 8+,
first released in 2006 and depended on by a large number of downstream projects,
many of which pin old versions and never update. **Backward compatibility
outranks correctness, elegance, and performance.** When those conflict, keep the
old behaviour and document the wart.

## Build and test

```
mvn verify          # compile, test, and build jar + sources + javadoc
mvn test            # tests only
```

Requires JDK 9+ to build (`maven.compiler.release=8`); the jar runs on Java 8.

Ant is gone. If you find a reference to `build.xml` or `test.xml`, it is stale —
those files specified `source="1.2"`, which no supported JDK accepts.

## Yylex.java is generated — do not edit it in isolation

`src/main/java/org/json/simple/parser/Yylex.java` is produced by
[JFlex](https://jflex.de/) 1.4.2 from `doc/json.lex`.

This is the single easiest way to silently break this repository. Editing only
the generated scanner works — it compiles, the tests pass — and the change is
then lost the next time anyone regenerates it.

**To change lexical rules or their action code:**

1. Edit `doc/json.lex` — it is the source of truth.
2. Mirror the change into `Yylex.java`. Only *action bodies* may be hand-mirrored;
   if you touched a regular expression, the DFA tables are now wrong and the file
   must be regenerated with JFlex.
3. Say in the commit message that both files were changed.

Wiring JFlex into the Maven build (so the file becomes a real build artifact) is
planned for 1.2.0, where a regenerated scanner's behavioural differences are
acceptable.

## Checking compatibility

Before proposing any change to `src/main/java`, run:

```
tools/compat-check.sh
```

It builds the last published release (`tag_release_1_1_1`) alongside the working
tree and compares them four ways: public API surface, binary compatibility
(probes compiled against the baseline, then run against both builds), observable
behaviour across ~120 inputs, and serialization including a cross-version round
trip. Any difference not recorded in `tools/compat/expected-diff.txt` fails the
check. It also runs in CI.

If a difference is intentional, review it and record it:

```
tools/compat-check.sh tag_release_1_1_1 --accept
```

Never run `--accept` to make a red failure go away. The recorded file is the
release's statement of what it deliberately changes for downstream users, and
every line in it should be one you can justify.

Three binary-compatibility regressions reached master after 1.1.1 shipped and
were fixed in 1.1.2. Do not undo them:

* `JSONArray.toJSONString(List)` and `writeJSONString(List, Writer)` exist as
  deprecated delegates to the `Collection` forms. Widening a parameter is
  source-compatible but **not** binary-compatible - already-compiled callers
  look up the exact descriptor and get `NoSuchMethodError`. Whenever you widen
  a public parameter type, leave the old signature behind as a delegate.
* `ParseException.toString()` returns the plain description, as it did on 1.1.1.
* `ParseException.serialVersionUID` is `-7880698968187728548L`, its 1.1.1 value.

`RegressionTest` pins all three.

## Compatibility red lines

Do not "fix" any of these. Each looks like a bug and is load-bearing for
downstream users.

* **`escape()` emits `\/` for a forward slash**, and `\uXXXX` for U+0000–U+001F,
  U+007F–U+009F and U+2000–U+20FF. Legal but unusual; a great many downstream
  tests hard-code this exact output.
* **Integers always decode to `Long`; fractional numbers always to `Double`.**
  Never "optimise" to `Integer`, `Float`, or `BigDecimal` — every downstream cast
  would break at once.
* **`JSONValue.parse()` swallows all exceptions and returns `null`.** Deprecated,
  but it may not be deleted or made to throw.
* **`ContainerFactory.creatArrayContainer()` is missing a letter.** It is a
  published interface method. Renaming it breaks every implementation. A
  correctly spelled method may be *added* in 2.0; the typo stays.
* **Public method signatures**, and the fact that `JSONObject extends HashMap`
  and `JSONArray extends ArrayList`.
* **Parser leniency** (trailing commas, optional separators, leading zeros,
  invalid escapes passing through). Documented in README.md. Document it further
  if you like; do not change the defaults. A strict mode is a 2.0 item.

Generics are also a red line *for the 1.x line*. Parameterising the collection
types is binary-compatible but **source-incompatible** — existing code calling
`obj.put(1, "x")` stops compiling. That change belongs in 2.0.

## Known behaviour that is not a red line

* Key order in a `JSONObject` is **not** preserved (it is a `HashMap`). Never
  assert on a serialised string when the object has more than one key — compare
  parsed structures instead. A test was already broken once by this when Java 8
  changed `HashMap` iteration order (commit `a8b94b7`).
* `JSONValue.toJSONString` recurses once per nesting level, so a deep enough
  structure exhausts the stack. There is no fixed safe depth — it follows the
  thread's stack size: roughly 12,000 levels on a default 1 MB stack, under 2,000
  on a 512 KB stack. Decoding is iterative and is safe at any depth (verified to
  500,000). A configurable encode-depth limit is a 1.2.0 item.

## Code style

* **Tabs** for indentation in hand-written Java; braces on the same line. See
  `.editorconfig`. `Yylex.java` is generator-formatted with spaces — leave it.
* Most files use **CRLF** line endings. `.gitattributes` sets `* -text` so Git
  never converts them. Do not normalise line endings; it produces whole-file
  diffs and destroys `git blame`.
* Do not reformat files you are not otherwise changing.
* No new runtime dependencies. Ever. The zero-dependency property is a large part
  of why this library is still used.

## Testing

Tests live in `src/test/java`. `Test.java` uses JUnit 3 (`junit.framework.TestCase`)
and prints heavily to stdout; `JSONArrayTest` and `JSONValueTest` use JUnit 4.
Migrating everything to JUnit 5 with real assertions is a 1.2.0 item.

`ConformanceTest` is the one test that does not check json-simple against
itself: it compares against JSON-java (`org.json`), an independently written
implementation, on a corpus of strictly valid documents. The comparison is
semantic, because the two libraries produce different but equally valid text —
json-simple's `\/` and its upper-case u-escapes among them. Keep json-simple's
documented leniency out of that corpus: the reference rejects it, and the
disagreement would say nothing about correctness.

`RandomConformanceTest` generates documents rather than listing them, in two
shapes: one that combines value types freely, and one that makes nesting depth
the variable so that object-in-array-in-object chains are the normal case rather
than a lucky one. The seed is fixed, so a green build always means the same
thing; every failure message carries the seed, and the depth, needed to replay
it. Keep generation inside what both libraries accept - no integers wider than a
long, no duplicate keys, no unpaired surrogates - or the noise buries the signal.
Nesting stays well under sixty-four levels: both libraries recurse somewhere, and
the test should not turn into a measurement of the runner's stack size.

A differential test only sees what the other implementation rejects or reads
differently, and the reference is more lenient than RFC 8259 in places. It
accepts every raw control character inside a string except NUL, LF and CR, so an
encoder that stopped escaping U+0001 would still pass the comparison. That is
why `testEncodedOutputHasNoRawControlCharacters` checks the emitted text
directly. Reach for that kind of structural assertion wherever the reference is
laxer than the spec.

When you fix a bug, add an assertion that would have failed before the fix. When
you touch anything on the red-line list — even to prove you did not change it —
add an assertion that pins the current output.

## Release lines

* **1.1.x** — bug fixes and metadata only. No observable behaviour changes.
* **1.2.x** — additive: new opt-in features, defaults unchanged.
* **2.0.0** — the first release allowed to break source compatibility: generics,
  strict parsing mode, ordered maps, the `creatArrayContainer` rename.

Do not land a 2.0 change on the 1.x line because it seems small.

---
> Source: [fangyidong/json-simple](https://github.com/fangyidong/json-simple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
