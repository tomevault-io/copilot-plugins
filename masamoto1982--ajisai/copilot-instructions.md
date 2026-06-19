## ajisai

> <!-- GENERATED FILE — do not edit by hand.

<!-- GENERATED FILE — do not edit by hand.
     Regenerate: npm run generate:skill   (verified against the ajisai CLI)
     Source of truth for semantics: SPECIFICATION.html.
     Generator: scripts/generate-skill-md.mjs -->

# Ajisai — Agent Writing Protocol (SKILL.md)

How to *write working Ajisai on the first try*. Every code line below was
executed by the generator against the real interpreter; results shown are
actual outputs. **If a word is not in the §9 table, it does not exist — when
unsure, grep §9 before writing.**

## 1. Run loop

```sh
ajisai run program.ajisai --json     # exit 0 = ok, 1 = language error, 2 = usage
ajisai check program.ajisai --json   # parse + resolve only, no execution
```

Read the JSON in this order (contract: docs/dev/agent-cli-output-contract.md):
1. `status` / exit code. On ok: `stackDisplay` (final stack, bottom→top) and `output` (PRINT lines).
2. On error: `diagnosis.why` + `diagnosis.where` locate the failure; follow `diagnosis.nextChecks` in order; `aiDiagnostic.recoverability` says what kind of change fixes it (`fixProgram` / `fixInput` / `fixHost` ...).
3. Even on ok, scan `errorFlowTrace` for `nilProduced` events if a NIL surprised you.

## 2. Minimal syntax

- Postfix, stack-based. Operands first, word last: `[ 1 ] [ 2 ] +`.
- Numbers are **exact rationals** (`1/3`, `3.14` → 157/50). No floats. Display shows `3/1` for 3.
- Data lives in vectors: `[ 1 2 3 ]`. Nest for tensors: `[ [ 1 2 ] [ 3 4 ] ]`. A lone number like `42` is allowed but `[ 42 ]` is the idiomatic scalar.
- Strings: `'single quotes'` (a codepoint vector with text role). Booleans: `TRUE` / `FALSE`. Absence: `NIL`.
- Code blocks: `{ ... }` — quoted programs passed to MAP / FILTER / FOLD / COND / DEF.
- User word: `{ body } 'NAME' DEF` then call `NAME`. Words are case-insensitive (canonicalized to upper case).
- Comments: `#` to end of line.
- Modifiers prefix the *next word only*: `,,` (KEEP: don't consume operands), `..` (STAK: apply to whole stack), `,` (EAT, default), `.` (TOP, default).
- One word does one thing to the stack; there are **no** DUP/SWAP-style shufflers (§8).

## 3. Control and iteration

- Branch: `value { guard } { body } { guard } { body } ... COND`. Guards see the value (it stays for each guard) and must leave TRUE/FALSE; use `{ TRUE }` as the final else-guard. The value remains on the stack after COND.
- Iterate data, not counters: `MAP` / `FILTER` / `FOLD` / `SCAN` / `UNFOLD` with `{ }` blocks (examples in §6). `FOLD` requires an explicit `[ init ]`.
- Predicates: `ANY` / `ALL` / `COUNT` with a `{ predicate }` block.
- Recursion is allowed in user words (execution-step and depth limits apply; exceeding them is a diagnosed error, not a hang).

## 4. NIL — absence is a value, not an exception

Failed partial operations *bubble*: `[ 1 ] [ 0 ] DIV` succeeds (exit 0) and
pushes `NIL` (reason: `divisionByZero`). The projection is recorded in
`errorFlowTrace` as a `nilProduced` event with a full diagnosis, and the NIL
value itself carries `semantics.absence.reason` on the stack.

- Provide a fallback with `^`: `[ 1 ] [ 0 ] DIV ^ [ 99 ]` → stack `[ 99/1 ]`.
- NIL flows through later operations (bubble rule); check for it where it matters instead of letting it propagate to the end.

## 5. UNKNOWN — the third truth value

Comparisons of lazy exact reals are *budgeted*. When the budget is exhausted
without a decision, the result is the logical `UNKNOWN`, not an error and not NIL:

```ajisai
'MATH' IMPORT
2 SQRT 8 SQRT 2 DIV 3 COMPARE-WITHIN   # √2 vs √8/2 within 3 partial quotients
```

→ stack `UNKNOWN` (exit 0). In JSON the value serializes as
`{ "type": "truthValue", "value": "unknown" }` and carries
`agreedPrefix: 3` (leading partial quotients that matched) in
`semantics.absence.diagnosis`. Raise the budget or restructure the comparison
to decide; AND/OR/NOT follow Kleene three-valued logic over UNKNOWN.

## 6. Canonical examples (all verified by the generator)

- Push a number (always inside a vector)
  `[ 42 ]` → stack: `[ 42/1 ]`
- Exact rational division — no floats, ever
  `[ 1 ] [ 3 ] /` → stack: `[ 1/3 ]`
- Elementwise vector arithmetic
  `[ 1 2 3 ] [ 4 5 6 ] +` → stack: `[ 5/1 7/1 9/1 ]`
- Scalar broadcast over a vector
  `[ 5 ] [ 1 2 3 ] *` → stack: `[ 5/1 10/1 15/1 ]`
- Remainder
  `[ 10 ] [ 3 ] %` → stack: `[ 1/1 ]`
- Comparison pushes a boolean
  `[ 1 ] [ 2 ] <` → stack: `TRUE`
- Range: one vector [ start end ] (inclusive)
  `[ 0 5 ] RANGE` → stack: `[ 0/1 1/1 2/1 3/1 4/1 5/1 ]`
- Range with step: [ start end step ]
  `[ 0 10 2 ] RANGE` → stack: `[ 0/1 2/1 4/1 6/1 8/1 10/1 ]`
- Fill a tensor: [ shape... value ]
  `[ 2 2 7 ] FILL` → stack: `[ [ 7/1 7/1 ] [ 7/1 7/1 ] ]`
- Reshape
  `[ 1 2 3 4 5 6 ] [ 2 3 ] RESHAPE` → stack: `[ [ 1/1 2/1 3/1 ] [ 4/1 5/1 6/1 ] ]`
- MAP with a { } code block
  `[ 0 4 ] RANGE { [ 2 ] * } MAP` → stack: `[ 0/1 2/1 4/1 6/1 8/1 ]`
- FILTER keeps matching elements
  `[ 0 10 ] RANGE { [ 5 ] > } FILTER` → stack: `[ 6/1 7/1 8/1 9/1 10/1 ]`
- FOLD needs an explicit initial value
  `[ 1 2 3 ] [ 0 ] { + } FOLD` → stack: `[ 6/1 ]`
- ANY / ALL / COUNT take predicate blocks
  `[ 1 2 3 ] { [ 1 ] > } ANY` → stack: `TRUE`
- Define a user word: { body } then name, then DEF
  `{ [ 1 ] [ 2 ] + } 'MY-SUM' DEF MY-SUM` → stack: `[ 3/1 ]`
- COND: value on stack, then { guard } { body } pairs (use { TRUE } as else-guard)
  `[ 4 ] { [ 0 ] >= } { 'non-negative' PRINT } { TRUE } { 'negative' PRINT } COND` → prints `'non-negative'`; stack: `[ 4/1 ]`
- Strings are bare '...' literals; CHARS/JOIN convert
  `'hello' CHARS REVERSE JOIN` → stack: `'olleh'`
- Cast a string to an exact number
  `'42' NUM` → stack: `42/1`
- PRINT pops and emits to output (not the stack)
  `[ 1 2 3 ] PRINT` → prints `[ 1/1 2/1 3/1 ]`
- Module words need IMPORT first
  `'ALGO' IMPORT [ 3 1 2 ] SORT` → stack: `[ 1/1 2/1 3/1 ]`
- KEEP modifier `,,` makes the next word non-consuming
  `[ 5 ] ,, PRINT` → prints `[ 5/1 ]`; stack: `[ 5/1 ]`

## 7. Common errors — actual CLI output, and the fix

- **Typo / unknown word** — `[ 1 ] ADDD`
  → exit 1, `message: "Unknown word: ADDD"`, `diagnosis: { when: "resolveWord", why: "typoOrUnknownName" }`,
  `aiDiagnostic.recoverability: "fixProgram"`, first nextCheck: "Check spelling".
  Fix: Grep §9 for the word you meant (here: `+` / `ADD`). Word names are upper-cased automatically.
- **Stack underflow: operands must be pushed first** — `+`
  → exit 1, `message: "Stack underflow"`, `diagnosis: { when: "executeWord", why: "stackShape" }`,
  `aiDiagnostic.recoverability: "fixProgram"`, first nextCheck: "Check arity".
  Fix: Push both operands before the operator: `[ 1 ] [ 2 ] +`. Ajisai is postfix; there is no infix form.
- **FOLD without an initial value** — `[ 1 2 3 ] { + } FOLD`
  → exit 1, `message: "Stack underflow"`, `diagnosis: { when: "executeWord", why: "stackShape" }`,
  `aiDiagnostic.recoverability: "fixProgram"`, first nextCheck: "Check arity".
  Fix: FOLD is `vector [ init ] { op } FOLD`: `[ 1 2 3 ] [ 0 ] { + } FOLD`.
- **COND blocks must come in { guard } { body } pairs** — `[ 5 ] { [ 3 ] > } { 'big' PRINT } { 'small' PRINT } COND`
  → exit 1, `message: "COND: expected even number of code blocks (guard/body pairs), got 3"`, `diagnosis: { when: "executeWord", why: "unknown" }`,
  `aiDiagnostic.recoverability: "inspectContext"`, first nextCheck: "Check error message".
  Fix: Give every body a guard; the else-branch is `{ TRUE } { ... }`: `[ 5 ] { [ 3 ] > } { 'big' PRINT } { TRUE } { 'small' PRINT } COND`.
- **COND guards must yield a boolean** — `TRUE { [ 1 ] } { [ 2 ] } COND`
  → exit 1, `message: "COND: guard must return TRUE or FALSE, got non-scalar"`, `diagnosis: { when: "executeWord", why: "unknown" }`,
  `aiDiagnostic.recoverability: "inspectContext"`, first nextCheck: "Check error message".
  Fix: The first block is a guard, not a value: it must leave TRUE/FALSE. Branch on a stack value with `[ x ] { predicate } { body } ... COND`.
- **Broadcast shape mismatch** — `[ 1 2 ] [ 1 2 3 ] +`
  → exit 1, `message: "Cannot broadcast shapes [2] and [3]"`, `diagnosis: { when: "executeWord", why: "unknown" }`,
  `aiDiagnostic.recoverability: "inspectContext"`, first nextCheck: "Check error message".
  Fix: Elementwise ops need equal or broadcastable shapes (scalar `[ 5 ]` broadcasts; `[2]` vs `[3]` does not).
- **NUM casts strings, not booleans** — `TRUE NUM`
  → exit 1, `message: "NUM: expected String, got Boolean"`, `diagnosis: { when: "executeWord", why: "unknown" }`,
  `aiDiagnostic.recoverability: "inspectContext"`, first nextCheck: "Check error message".
  Fix: NUM accepts strings: `'42' NUM`. There is no boolean→number cast.
- **Old two-vector RANGE form** — `[ 0 ] [ 5 ] RANGE`
  → exit 1, `message: "RANGE requires [start end] or [start end step]"`, `diagnosis: { when: "executeWord", why: "unknown" }`,
  `aiDiagnostic.recoverability: "inspectContext"`, first nextCheck: "Check error message".
  Fix: RANGE takes one vector: `[ 0 5 ] RANGE` (or `[ start end step ]`).
- **Vector-wrapped string passed to a cast** — `[ '42' ] NUM`
  → exit 1, `message: "NUM: expected String input"`, `diagnosis: { when: "executeWord", why: "unknown" }`,
  `aiDiagnostic.recoverability: "inspectContext"`, first nextCheck: "Check error message".
  Fix: String casts take the bare string: `'42' NUM`.

## 8. Forbidden patterns (each verified to fail)

- **DUP / SWAP / DROP / OVER / ROT** (`DUP` fails) — Forth-style stack shufflers do not exist. Use the modifiers instead: `,,` (KEEP: next word does not consume), `..` (STAK: next word applies to the whole stack).
- **IF / ELSE / THEN / WHILE** (`[ 1 ] IF` fails) — No structured keywords. Branch with COND guard/body pairs; iterate with MAP / FILTER / FOLD / UNFOLD or recursive user words.
- **Parentheses ( )** (`( 1 2 )` fails) — Reserved for the continued-fraction *display* form only. Vectors are `[ ]`, code blocks are `{ }`.
- **Double-quoted strings** (`"hello" PRINT` fails) — Strings use single quotes: 'hello'.
- **// line comments** (`// comment` fails) — Comments start with `#`.

## 9. Word quick reference

Generated from `docs/word-manifest.json` — the complete inventory. A word
absent here does not exist. Module words need `'MODULE' IMPORT` once per
program (then the short name works), or can be called fully qualified.

| word | category | summary |
|---|---|---|
| `TOP` | modifier | Set the operation target mode to the top of the stack. — e.g. `. +` |
| `STAK` | modifier | Set the operation target mode to the whole stack. — e.g. `.. +` |
| `EAT` | modifier | Set the consumption mode to consume operands. — e.g. `, +` |
| `KEEP` | modifier | Set the consumption mode to keep operands. — e.g. `,, +` |
| `GET` | vector | Extract one element of a vector by index. — e.g. `[ 10 20 30 ] [ 0 ] GET` |
| `INSERT` | vector | Insert a value at a given index in a vector. — e.g. `[ 1 3 ] [ 1 2 ] INSERT` |
| `REPLACE` | vector | Replace an element of a vector at a given index. — e.g. `[ 1 2 3 ] [ 0 9 ] REPLACE` |
| `REMOVE` | vector | Remove an element from a vector at a given index. — e.g. `[ 1 2 3 ] [ 0 ] REMOVE` |
| `LENGTH` | vector | Return the number of elements in a vector. — e.g. `[ 1 2 3 ] LENGTH` |
| `TAKE` | vector | Take the first N or last -N elements of a vector. — e.g. `[ 1 2 3 4 5 ] [ 3 ] TAKE` |
| `SPLIT` | vector | Split a vector into chunks at the specified sizes. — e.g. `[ 1 2 3 4 ] [ 2 2 ] SPLIT` |
| `CONCAT` | vector | Flatten and concatenate two vectors. — e.g. `[ 1 2 ] [ 3 4 ] CONCAT` |
| `REVERSE` | vector | Reverse the order of vector elements. — e.g. `[ 1 2 3 ] REVERSE` |
| `RANGE` | vector | Generate a numeric sequence from a [start, end] pair. — e.g. `[ 0 5 ] RANGE` |
| `REORDER` | vector | Reorder vector elements according to an index permutation. — e.g. `[ 'a' 'b' 'c' ] [ 2 0 1 ] REORDER` |
| `COLLECT` | vector | Collect N items off the stack into a new vector. — e.g. `1 2 3 3 COLLECT` |
| `TRUE` | constant | Push the boolean TRUE onto the stack. — e.g. `TRUE` |
| `FALSE` | constant | Push the boolean FALSE onto the stack. — e.g. `FALSE` |
| `NIL` | constant | Push the NIL value onto the stack. — e.g. `NIL` |
| `>CF` | conversion | Tag a numeric scalar for canonical continued-fraction serialization (SPEC 12.2). — e.g. `2 MATH@SQRT >CF` |
| `CHARS` | cast | Split a string into a vector of one-character strings. — e.g. `'hi' CHARS` |
| `JOIN` | cast | Join a vector of strings into a single string. — e.g. `[ 'h' 'i' ] JOIN` |
| `TRIM` | cast | Remove whitespace from both ends of a string. — e.g. `'  hi  ' TRIM` |
| `TRIM-LEFT` | cast | Remove whitespace from the start of a string. — e.g. `'  hi' TRIM-LEFT` |
| `TRIM-RIGHT` | cast | Remove whitespace from the end of a string. — e.g. `'hi  ' TRIM-RIGHT` |
| `TOKENIZE` | cast | Split a string into a vector of substrings using a separator. — e.g. `'a,b,c' ',' TOKENIZE` |
| `SUBSTITUTE` | cast | Replace every occurrence of a substring with another. — e.g. `'hello' 'l' 'L' SUBSTITUTE` |
| `STARTS-WITH?` | cast | Test whether a string begins with the given prefix. — e.g. `'hello' 'he' STARTS-WITH?` |
| `ENDS-WITH?` | cast | Test whether a string ends with the given suffix. — e.g. `'hello' 'lo' ENDS-WITH?` |
| `NUM` | cast | Parse text as a number; Bubble/NIL on parse failure. — e.g. `'42' NUM` |
| `STR` | cast | Convert a value to its string representation. — e.g. `42 STR` |
| `BOOL` | cast | Convert a value to a boolean by truthiness. — e.g. `1 BOOL` |
| `CHR` | cast | Convert a numeric character code to a single-character string. — e.g. `65 CHR` |
| `ADD` | arithmetic | Add two numeric values, element-wise with broadcasting. — e.g. `1 2 +` |
| `SUB` | arithmetic | Subtract two numeric values, element-wise with broadcasting. — e.g. `5 3 -` |
| `MUL` | arithmetic | Multiply two numeric values, element-wise with broadcasting. — e.g. `2 4 *` |
| `DIV` | arithmetic | Divide two numeric values exactly (fractional result). — e.g. `10 2 /` |
| `COMPARE-WITHIN` | comparison | Three-way compare two values within an explicit partial-quotient budget. — e.g. `a b 64 COMPARE-WITHIN` |
| `EQ` | comparison | Test equality of two values. — e.g. `1 1 =` |
| `LT` | comparison | Test less-than comparison. — e.g. `1 2 <` |
| `LTE` | comparison | Test less-than-or-equal comparison. — e.g. `1 1 <=` |
| `GT` | comparison | Test greater-than comparison. — e.g. `2 1 >` |
| `GTE` | comparison | Test greater-than-or-equal comparison. — e.g. `1 1 >=` |
| `NEQ` | comparison | Test inequality of two values. — e.g. `1 2 <>` |
| `AND` | logic | Logical AND with three-valued (Kleene) NIL handling. — e.g. `TRUE TRUE &` |
| `OR` | logic | Logical OR with three-valued (Kleene) NIL handling. — e.g. `TRUE FALSE OR` |
| `NOT` | logic | Logical negation. — e.g. `TRUE NOT` |
| `IDLE` | control | Pass control through unchanged (no-op). — e.g. `IDLE` |
| `COND` | control | Evaluate guard/body clauses in order, executing the first match. — e.g. `1 { TRUE | 'y' } { IDLE | 'n' } COND` |
| `FLOW` | modifier | Pipeline visual marker (no-op). — e.g. `xs ~ { ... } MAP` |
| `VENT` | modifier | Bubble/NIL fallback operator: substitute an alternative if value is NIL. — e.g. `NIL ^ [ 0 ]` |
| `MAP` | higher-order | Apply a code block to each element of a vector. — e.g. `[ 1 2 3 ] { [ 2 ] * } MAP` |
| `FILTER` | higher-order | Keep only the elements for which a predicate block returns TRUE. — e.g. `[ 1 2 3 ] { [ 2 ] = } FILTER` |
| `FOLD` | higher-order | Reduce a vector to a single value using an initial accumulator and combiner block. — e.g. `[ 1 2 3 ] [ 0 ] { + } FOLD` |
| `UNFOLD` | higher-order | Generate a sequence by repeatedly applying a state transition. — e.g. `[ 1 ] { ... COND } UNFOLD` |
| `ANY` | higher-order | TRUE if at least one element satisfies the predicate. — e.g. `[ 1 2 3 ] { [ 2 ] = } ANY` |
| `ALL` | higher-order | TRUE if every element satisfies the predicate. — e.g. `[ 2 4 ] { [ 2 ] MOD [ 0 ] = } ALL` |
| `COUNT` | higher-order | Count the elements that satisfy the predicate. — e.g. `[ 1 2 3 ] { [ 2 ] = } COUNT` |
| `SCAN` | higher-order | Return a vector of intermediate fold accumulators. — e.g. `[ 1 2 3 ] [ 0 ] { + } SCAN` |
| `PRINT` | io | Output a value to the display. — e.g. `42 PRINT` |
| `PRECOMPUTE` | Control / Staging | Definition-time staging marker (not a macro). — e.g. `{ ... } PRECOMPUTE` |
| `DEF` | dictionary | Define a user word from a body and a name. — e.g. `{ 2 * } 'DOUBLE' DEF` |
| `DEL` | dictionary | Delete a user word from the dictionary. — e.g. `'WORD' DEL` |
| `LOOKUP` | dictionary | Display the documentation for a named word. — e.g. `'ADD' ?` |
| `FORC` | control | Force destructive dictionary operations to apply. — e.g. `! 'WORD' DEL` |
| `SHAPE` | tensor | Return a vector describing the dimensions of a value. — e.g. `[ 1 2 3 ] SHAPE` |
| `RANK` | tensor | Return the number of dimensions of a value. — e.g. `[ [ 1 2 ] ] RANK` |
| `RESHAPE` | tensor | Reshape a vector to a target shape with the same total length. — e.g. `[ 1 2 3 4 ] [ 2 2 ] RESHAPE` |
| `TRANSPOSE` | tensor | Transpose the axes of a tensor. — e.g. `[ ( 1 2 ) ( 3 4 ) ] TRANSPOSE` |
| `FILL` | tensor | Fill a target shape with a constant value. — e.g. `[ 2 2 0 ] FILL` |
| `MOD` | arithmetic | Modulo (remainder) of two numeric values. — e.g. `7 3 %` |
| `FLOOR` | arithmetic | Round toward negative infinity. — e.g. `[ 7/3 ] FLOOR` |
| `CEIL` | arithmetic | Round toward positive infinity. — e.g. `[ 7/3 ] CEIL` |
| `ROUND` | arithmetic | Round to nearest integer (half-up). — e.g. `[ 5/2 ] ROUND` |
| `EXEC` | control | Execute a vector as Ajisai code. — e.g. `[ 1 2 + ] EXEC` |
| `EVAL` | control | Parse a string as Ajisai source code and execute it. — e.g. `'1 2 +' EVAL` |
| `IMPORT` | module | Load all public words of a module into the dictionary. — e.g. `'IO' IMPORT` |
| `IMPORT-ONLY` | module | Load only the listed public words of a module. — e.g. `'json' [ 'parse' ] IMPORT-ONLY` |
| `UNIMPORT` | module | Hide unused imported words from a module while keeping words referenced by user definitions. — e.g. `'IO' UNIMPORT` |
| `UNIMPORT-ONLY` | module | Hide only the listed imported module words. — e.g. `'json' [ 'parse' ] UNIMPORT-ONLY` |
| `SPAWN` | control | Spawn an isolated child runtime from a code block. — e.g. `{ 1 2 + } SPAWN` |
| `AWAIT` | control | Wait for a child runtime to finish and return its exit tuple. — e.g. `{ 1 2 + } SPAWN AWAIT` |
| `STATUS` | control | Read the current status of a child runtime. — e.g. `{ 1 2 + } SPAWN STATUS` |
| `KILL` | control | Forcibly terminate a child runtime. — e.g. `{ 1 2 + } SPAWN KILL` |
| `MONITOR` | control | Register a monitor on a child handle. — e.g. `{ 1 2 + } SPAWN MONITOR` |
| `SUPERVISE` | control | Run a code block under a one-for-one restart policy. — e.g. `{ 1 2 + } [ 3 ] SUPERVISE` |
| `MUSIC@SEQ` | music (module) | Set sequential playback mode — needs `'MUSIC' IMPORT` (or call as `MUSIC@SEQ`) |
| `MUSIC@SIM` | music (module) | Set simultaneous playback mode — needs `'MUSIC' IMPORT` (or call as `MUSIC@SIM`) |
| `MUSIC@SLOT` | music (module) | Set slot duration in seconds — needs `'MUSIC' IMPORT` (or call as `MUSIC@SLOT`) |
| `MUSIC@GAIN` | music (module) | Set volume level (0.0-1.0) — needs `'MUSIC' IMPORT` (or call as `MUSIC@GAIN`) |
| `MUSIC@GAIN-RESET` | music (module) | Reset volume to default (1.0) — needs `'MUSIC' IMPORT` (or call as `MUSIC@GAIN-RESET`) |
| `MUSIC@PAN` | music (module) | Set stereo position (-1.0 left to 1.0 right) — needs `'MUSIC' IMPORT` (or call as `MUSIC@PAN`) |
| `MUSIC@PAN-RESET` | music (module) | Reset pan to center (0.0) — needs `'MUSIC' IMPORT` (or call as `MUSIC@PAN-RESET`) |
| `MUSIC@FX-RESET` | music (module) | Reset all audio effects to defaults — needs `'MUSIC' IMPORT` (or call as `MUSIC@FX-RESET`) |
| `MUSIC@PLAY` | music (module) | Play audio — needs `'MUSIC' IMPORT` (or call as `MUSIC@PLAY`) |
| `MUSIC@SEQ-GROUP` | music (module) | Build an explicit sequential music group from a vector — needs `'MUSIC' IMPORT` (or call as `MUSIC@SEQ-GROUP`) |
| `MUSIC@SIM-GROUP` | music (module) | Build an explicit simultaneous music group from a vector — needs `'MUSIC' IMPORT` (or call as `MUSIC@SIM-GROUP`) |
| `MUSIC@CHORD` | music (module) | Build an explicit chord group (simultaneous) from a vector — needs `'MUSIC' IMPORT` (or call as `MUSIC@CHORD`) |
| `MUSIC@HZ` | music (module) | Build a music.pitch from a frequency in Hz (exact rational) — needs `'MUSIC' IMPORT` (or call as `MUSIC@HZ`) |
| `MUSIC@DUR` | music (module) | Build a music.duration from a number of seconds — needs `'MUSIC' IMPORT` (or call as `MUSIC@DUR`) |
| `MUSIC@NOTE` | music (module) | Combine a music.pitch and a music.duration into a music.note — needs `'MUSIC' IMPORT` (or call as `MUSIC@NOTE`) |
| `MUSIC@REST` | music (module) | Build a music.rest from a music.duration — needs `'MUSIC' IMPORT` (or call as `MUSIC@REST`) |
| `MUSIC@EDO` | music (module) | Build an equal-division-of-the-octave music.tuning — needs `'MUSIC' IMPORT` (or call as `MUSIC@EDO`) |
| `MUSIC@EDR` | music (module) | Build an equal-division-of-a-ratio music.tuning (non-octave) — needs `'MUSIC' IMPORT` (or call as `MUSIC@EDR`) |
| `MUSIC@STEP` | music (module) | Resolve a step within a music.tuning into a music.pitch — needs `'MUSIC' IMPORT` (or call as `MUSIC@STEP`) |
| `MUSIC@VOICE` | music (module) | Build a music group with the role of a single melodic voice — needs `'MUSIC' IMPORT` (or call as `MUSIC@VOICE`) |
| `MUSIC@TRACK` | music (module) | Build a music group with the role of an instrument track — needs `'MUSIC' IMPORT` (or call as `MUSIC@TRACK`) |
| `MUSIC@MEASURE` | music (module) | Build a music group with the role of a measure (bar) — needs `'MUSIC' IMPORT` (or call as `MUSIC@MEASURE`) |
| `MUSIC@PHRASE` | music (module) | Build a music group with the role of a phrase — needs `'MUSIC' IMPORT` (or call as `MUSIC@PHRASE`) |
| `MUSIC@WITH-TUNING` | music (module) | Bind a tuning over a body so bare integers become tuning steps — needs `'MUSIC' IMPORT` (or call as `MUSIC@WITH-TUNING`) |
| `MUSIC@EXPLAIN` | music (module) | Explain how MUSIC@PLAY would interpret a value — needs `'MUSIC' IMPORT` (or call as `MUSIC@EXPLAIN`) |
| `MUSIC@ADSR` | music (module) | Set ADSR envelope — needs `'MUSIC' IMPORT` (or call as `MUSIC@ADSR`) |
| `MUSIC@SINE` | music (module) | Set sine waveform — needs `'MUSIC' IMPORT` (or call as `MUSIC@SINE`) |
| `MUSIC@SQUARE` | music (module) | Set square waveform — needs `'MUSIC' IMPORT` (or call as `MUSIC@SQUARE`) |
| `MUSIC@SAW` | music (module) | Set sawtooth waveform — needs `'MUSIC' IMPORT` (or call as `MUSIC@SAW`) |
| `MUSIC@TRI` | music (module) | Set triangle waveform — needs `'MUSIC' IMPORT` (or call as `MUSIC@TRI`) |
| `JSON@PARSE` | json (module) | Parse JSON string to Ajisai value — needs `'JSON' IMPORT` (or call as `JSON@PARSE`) |
| `JSON@STRINGIFY` | json (module) | Convert Ajisai value to JSON string — needs `'JSON' IMPORT` (or call as `JSON@STRINGIFY`) |
| `JSON@GET` | json (module) | Get value by key from JSON object — needs `'JSON' IMPORT` (or call as `JSON@GET`) |
| `JSON@KEYS` | json (module) | Get all keys from JSON object — needs `'JSON' IMPORT` (or call as `JSON@KEYS`) |
| `JSON@SET` | json (module) | Set key-value in JSON object — needs `'JSON' IMPORT` (or call as `JSON@SET`) |
| `JSON@HAS` | json (module) | True if a JSON object contains the given key — needs `'JSON' IMPORT` (or call as `JSON@HAS`) |
| `JSON@VALUES` | json (module) | Get all values from a JSON object — needs `'JSON' IMPORT` (or call as `JSON@VALUES`) |
| `JSON@MERGE` | json (module) | Merge two JSON objects; right-hand keys win on conflict — needs `'JSON' IMPORT` (or call as `JSON@MERGE`) |
| `JSON@DELETE` | json (module) | Remove a key from a JSON object — needs `'JSON' IMPORT` (or call as `JSON@DELETE`) |
| `JSON@EXPORT` | json (module) | Export stack top as JSON file download — needs `'JSON' IMPORT` (or call as `JSON@EXPORT`) |
| `IO@INPUT` | io (module) | Read text from input buffer — needs `'IO' IMPORT` (or call as `IO@INPUT`) |
| `IO@OUTPUT` | io (module) | Write value to output buffer — needs `'IO' IMPORT` (or call as `IO@OUTPUT`) |
| `TIME@NOW` | time (module) | Get current Unix timestamp — needs `'TIME' IMPORT` (or call as `TIME@NOW`) |
| `TIME@DATETIME` | time (module) | Render an instant as civil [Y M D h m s] at a UTC offset (hours) — needs `'TIME' IMPORT` (or call as `TIME@DATETIME`) |
| `TIME@TIMESTAMP` | time (module) | Resolve a civil datetime to an instant at a UTC offset (hours) — needs `'TIME' IMPORT` (or call as `TIME@TIMESTAMP`) |
| `TIME@DATE` | time (module) | Extract the [Y M D] date from a datetime — needs `'TIME' IMPORT` (or call as `TIME@DATE`) |
| `TIME@TIME` | time (module) | Extract the [h m s] time-of-day from a datetime — needs `'TIME' IMPORT` (or call as `TIME@TIME`) |
| `TIME@YEAR` | time (module) | Year field of a date or datetime — needs `'TIME' IMPORT` (or call as `TIME@YEAR`) |
| `TIME@MONTH` | time (module) | Month field of a date or datetime — needs `'TIME' IMPORT` (or call as `TIME@MONTH`) |
| `TIME@DAY` | time (module) | Day field of a date or datetime — needs `'TIME' IMPORT` (or call as `TIME@DAY`) |
| `TIME@HOUR` | time (module) | Hour field of a time or datetime — needs `'TIME' IMPORT` (or call as `TIME@HOUR`) |
| `TIME@MINUTE` | time (module) | Minute field of a time or datetime — needs `'TIME' IMPORT` (or call as `TIME@MINUTE`) |
| `TIME@SECOND` | time (module) | Second field of a time or datetime — needs `'TIME' IMPORT` (or call as `TIME@SECOND`) |
| `TIME@WEEKDAY` | time (module) | ISO weekday of a date or datetime (Monday=1 .. Sunday=7) — needs `'TIME' IMPORT` (or call as `TIME@WEEKDAY`) |
| `TIME@ADD-DAYS` | time (module) | Shift a date or datetime by N whole days — needs `'TIME' IMPORT` (or call as `TIME@ADD-DAYS`) |
| `TIME@DIFF-DAYS` | time (module) | Whole-day difference a-b between two dates/datetimes — needs `'TIME' IMPORT` (or call as `TIME@DIFF-DAYS`) |
| `TIME@FORMAT` | time (module) | ISO-8601 text for a date (YYYY-MM-DD) or datetime (YYYY-MM-DDThh:mm:ss) — needs `'TIME' IMPORT` (or call as `TIME@FORMAT`) |
| `TIME@PARSE-ISO` | time (module) | Parse an ISO-8601 civil string into a datetime; Bubble/NIL if invalid — needs `'TIME' IMPORT` (or call as `TIME@PARSE-ISO`) |
| `TIME@ADD-MONTHS` | time (module) | Add N months to a date/datetime, clamping to the month end — needs `'TIME' IMPORT` (or call as `TIME@ADD-MONTHS`) |
| `TIME@ADD-YEARS` | time (module) | Add N years to a date/datetime, clamping Feb 29 in non-leap years — needs `'TIME' IMPORT` (or call as `TIME@ADD-YEARS`) |
| `CRYPTO@CSPRNG` | crypto (module) | Generate cryptographically secure random numbers — needs `'CRYPTO' IMPORT` (or call as `CRYPTO@CSPRNG`) |
| `CRYPTO@HASH` | crypto (module) | Compute hash value — needs `'CRYPTO' IMPORT` (or call as `CRYPTO@HASH`) |
| `ALGO@SORT` | algo (module) | Sort vector elements in ascending order — needs `'ALGO' IMPORT` (or call as `ALGO@SORT`) |
| `ALGO@UNIQUE` | algo (module) | Remove duplicate elements, preserving first-occurrence order — needs `'ALGO' IMPORT` (or call as `ALGO@UNIQUE`) |
| `ALGO@CONTAINS` | algo (module) | True if a vector contains an element equal to the given value — needs `'ALGO' IMPORT` (or call as `ALGO@CONTAINS`) |
| `ALGO@INDEX-OF` | algo (module) | Index of the first element equal to the value; Bubble/NIL if absent — needs `'ALGO' IMPORT` (or call as `ALGO@INDEX-OF`) |
| `MATH@SQRT` | math (module) | Square root. Exact rational roots stay exact; otherwise returns sound interval. — needs `'MATH' IMPORT` (or call as `MATH@SQRT`) |
| `MATH@SQRT-EPS` | math (module) | Square root with explicit interval width bound eps. — needs `'MATH' IMPORT` (or call as `MATH@SQRT-EPS`) |
| `MATH@INTERVAL` | math (module) | Create interval [lo, hi]. — needs `'MATH' IMPORT` (or call as `MATH@INTERVAL`) |
| `MATH@LOWER` | math (module) | Lower endpoint of number/interval. — needs `'MATH' IMPORT` (or call as `MATH@LOWER`) |
| `MATH@UPPER` | math (module) | Upper endpoint of number/interval. — needs `'MATH' IMPORT` (or call as `MATH@UPPER`) |
| `MATH@WIDTH` | math (module) | Interval width hi-lo. — needs `'MATH' IMPORT` (or call as `MATH@WIDTH`) |
| `MATH@IS-EXACT` | math (module) | True for exact number or degenerate interval. — needs `'MATH' IMPORT` (or call as `MATH@IS-EXACT`) |
| `MATH@ABS` | math (module) | Absolute value of a number. — needs `'MATH' IMPORT` (or call as `MATH@ABS`) |
| `MATH@NEG` | math (module) | Negate a number. — needs `'MATH' IMPORT` (or call as `MATH@NEG`) |
| `MATH@SIGN` | math (module) | Sign of a number: -1, 0, or 1. — needs `'MATH' IMPORT` (or call as `MATH@SIGN`) |
| `MATH@MIN` | math (module) | Smaller of two numbers. — needs `'MATH' IMPORT` (or call as `MATH@MIN`) |
| `MATH@MAX` | math (module) | Larger of two numbers. — needs `'MATH' IMPORT` (or call as `MATH@MAX`) |
| `MATH@POW` | math (module) | Integer-exponent exact power: base exp -- base^exp. — needs `'MATH' IMPORT` (or call as `MATH@POW`) |
| `MATH@GCD` | math (module) | Greatest common divisor of two integers. — needs `'MATH' IMPORT` (or call as `MATH@GCD`) |
| `MATH@LCM` | math (module) | Least common multiple of two integers. — needs `'MATH' IMPORT` (or call as `MATH@LCM`) |
| `SERIAL@LIST-PORTS` | serial (module) | Ask the host to enumerate available serial ports — needs `'SERIAL' IMPORT` (or call as `SERIAL@LIST-PORTS`) |
| `SERIAL@OPEN` | serial (module) | Open a serial port by id; leaves the port-id handle on the stack — needs `'SERIAL' IMPORT` (or call as `SERIAL@OPEN`) |
| `SERIAL@CONFIGURE` | serial (module) | Set the baud rate of an open serial port — needs `'SERIAL' IMPORT` (or call as `SERIAL@CONFIGURE`) |
| `SERIAL@WRITE` | serial (module) | Write a byte vector to an open serial port — needs `'SERIAL' IMPORT` (or call as `SERIAL@WRITE`) |
| `SERIAL@READ` | serial (module) | Drain received bytes from an open serial port; Bubble/NIL when none — needs `'SERIAL' IMPORT` (or call as `SERIAL@READ`) |
| `SERIAL@FLUSH` | serial (module) | Flush the outgoing buffer of an open serial port — needs `'SERIAL' IMPORT` (or call as `SERIAL@FLUSH`) |
| `SERIAL@CLOSE` | serial (module) | Close an open serial port — needs `'SERIAL' IMPORT` (or call as `SERIAL@CLOSE`) |
| `+` | symbol alias | shorthand for `ADD` |
| `-` | symbol alias | shorthand for `SUB` |
| `*` | symbol alias | shorthand for `MUL` |
| `/` | symbol alias | shorthand for `DIV` |
| `%` | symbol alias | shorthand for `MOD` |
| `=` | symbol alias | shorthand for `EQ` |
| `<` | symbol alias | shorthand for `LT` |
| `<=` | symbol alias | shorthand for `LTE` |
| `>` | symbol alias | shorthand for `GT` |
| `>=` | symbol alias | shorthand for `GTE` |
| `<>` | symbol alias | shorthand for `NEQ` |
| `!` | symbol alias | shorthand for `FORC` |
| `&` | symbol alias | shorthand for `AND` |
| `.` | syntax sugar | shorthand for `TOP` |
| `..` | syntax sugar | shorthand for `STAK` |
| `,` | syntax sugar | shorthand for `EAT` |
| `,,` | syntax sugar | shorthand for `KEEP` |
| `'` | input helper | shorthand for `STRING-QUOTE` |
| `?` | symbol alias | shorthand for `LOOKUP` |
| `~` | syntax sugar | shorthand for `FLOW` |
| `#` | source directive | shorthand for `COMMENT-LINE` |
| `\|` | control directive | shorthand for `COND-CLAUSE` |
| `[` | delimiter sugar | shorthand for `BEGIN-VECTOR` |
| `]` | delimiter sugar | shorthand for `END-VECTOR` |
| `{` | delimiter sugar | shorthand for `BEGIN-BLOCK` |
| `}` | delimiter sugar | shorthand for `END-BLOCK` |
| `'` | literal sugar | shorthand for `STRING-QUOTE` |
| `;` | modifier sugar | shorthand for `. ,` |
| `;;` | modifier sugar | shorthand for `.. ,,` |

---
> Source: [masamoto1982/Ajisai](https://github.com/masamoto1982/Ajisai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
