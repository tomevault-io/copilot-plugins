## safescript

> Write and run safescript programs. Safescript is a sandboxed language with static analysis — before code runs, you can see exactly which hosts it contacts, which secrets it reads/writes, and its resource bounds.


# Safescript Language Reference

Safescript is a small, sandboxed language designed for secure automation. Every
program is statically analyzed before execution, so you always know what network
hosts it contacts, what secrets it reads or writes, and its resource bounds.
There are no escape hatches.

## Quick Start

Use `analyze_safescript` to inspect a script before running it. Use
`run_safescript` to execute a function from a script. Community skills may also
expose individual safescript functions as callable tools directly.

## Program Structure

A program is zero or more imports and zero or more function definitions. Imports
and functions can appear in any order at the top level.

```
import helperFn from "https://example.com/lib.ss" perms { hosts: ["api.example.com"] } hash "sha256:abc123..."

myFunction = (name: string, count: number): string => {
  result = stringConcat({ parts: [name, " x", jsonStringify({ value: count }).text] })
  return result.result
}
```

## No Semicolons

Safescript does not use semicolons. Statements are separated by newlines or
whitespace.

## Function Definitions

```
functionName = (param1: Type1, param2: Type2): ReturnType => {
  // body statements
  return expression
}
```

Every function must end with `return expression`. There is no early return —
`return` cannot appear inside `if`/`else` blocks, only as the final item in the
function body. Return type annotation is optional.

## Types

Primitives: `string`, `number`, `boolean`

Arrays: `string[]`, `number[]`, `boolean[]`, or nested: `string[][]`

Objects: `{ fieldName: Type, otherField: Type }`

Nested: `{ items: { name: string, score: number }[], total: number }`

Array of objects: `{ name: string }[]`

## Expressions

### Literals

- Strings: `"hello"` (double quotes only, no single quotes, no template
  literals)
- Numbers: `42`, `3.14` (no scientific notation, no hex/octal/binary)
- Booleans: `true`, `false`
- Arrays: `[1, 2, 3]`, `["a", "b"]`, `[]`
- Objects: `{ name: "alice", age: 30 }`

### String Escape Sequences

`\n` (newline), `\t` (tab), `\\` (backslash), `\"` (double quote)

### Object Shorthand

`{ name }` is equivalent to `{ name: name }`. String keys are also allowed:
`{ "content-type": value }`.

### Operators (by precedence, lowest first)

1. Ternary: `condition ? thenExpr : elseExpr`
2. Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`
3. Additive: `+` (numbers or string concatenation), `-`
4. Multiplicative: `*`, `/`, `%`
5. Unary: `-x`
6. Dot access: `obj.field`, chainable: `a.b.c`

String concatenation with `+` only works when BOTH operands are strings. Use
`stringConcat` for joining multiple parts.

### Field Access

Access object fields with dot notation: `response.status`, `result.body`.
Chainable: `identity.keys.signingPublicKey`. Keywords are allowed as field
names: `obj.hash`, `obj.map`, `obj.return`.

## Statements

Three kinds of statements:

### Assignment

```
x = someExpression
```

### Void Call (calling an op without capturing the result)

```
writeSecret({ name: "my-key", value: newValue })
```

### If/Else

```
if condition {
  x = "yes"
} else {
  x = "no"
}
```

No `else if` — nest instead: `else { if ... { } }`. Variables assigned inside
blocks are visible after the block.

## Calling Built-in Ops

Ops are the only way to perform I/O, crypto, and data transformation. They are
called with named arguments in an object:

```
result = httpRequest({ host: "api.example.com", method: "GET", path: "/data" })
parsed = jsonParse({ text: result.body })
```

Some ops support a shorthand (unary sugar) when they take a single argument:

```
h = sha256("hello")           // same as sha256({ data: "hello" })
t = jsonStringify(myObj)       // same as jsonStringify({ value: myObj })
parts = stringConcat(myArray)  // same as stringConcat({ parts: myArray })
```

You CANNOT call user-defined functions directly. They can only be invoked
through `map`, `filter`, or `reduce`.

## Iteration with map, filter, reduce

No loops exist. Use these instead:

### map(functionName, array)

Applies a 1-parameter function to each element. Runs in parallel.

```
double = (n: number): number => {
  return n * 2
}
main = (nums: number[]): number[] => {
  return map(double, nums)
}
```

### filter(functionName, array)

Keeps elements where a 1-parameter function returns truthy. Runs in parallel.

```
isPositive = (n: number): boolean => {
  return n > 0
}
main = (nums: number[]): number[] => {
  return filter(isPositive, nums)
}
```

### reduce(functionName, initial, array)

Folds an array with a 2-parameter function (accumulator, element). Runs
sequentially.

```
add = (acc: number, n: number): number => {
  return acc + n
}
main = (nums: number[]): number => {
  return reduce(add, 0, nums)
}
```

The first argument is always a function NAME (identifier), not an inline
expression.

## Override (DAG composition with substitution)

`override(target, { name: replacement, ... })` returns a first-class DAG value
that behaves like the user function `target`, but with every reference to `name`
(a builtin op label or another user-fn name) rewritten to `replacement` (a
user-fn name). Substitution is **transitive** — callees of `target` are
rewritten too.

```
fetchExample = (): string => {
  return httpRequest({ host: "example.com", path: "/" })
}

inner = (): string => {
  return httpRequest({ host: "original.com", path: "/" })
}

useFetcher = (): string => {
  return inner()
}

main = (): string => {
  f = override(useFetcher, { inner: fetchExample })
  return f()
}
```

Three calling forms are supported:

- Inline: `override(target, {...})(arg1, arg2)` or
  `override(target, {...})({k: v})`
- Local-bound: `f = override(...); f({k: v})` or `f()`
- As `map`/`filter`/`reduce` fn argument

Replacement keys can be op labels (e.g. `httpRequest`) or user-fn names.
Replacement values must be user-fn names. The runtime executes the rewritten
DAG; signatures (hosts, secrets, env, complexity) reflect the substitution.

## Static Field Constraint

Some op arguments must be string literals — they cannot be variables or
expressions. This is what makes static analysis possible.

Static fields:

- `host` in `httpRequest`

```
// CORRECT — host is a string literal
response = httpRequest({ host: "api.example.com", method: "GET", path: myPath })

// WRONG — host cannot be a variable
response = httpRequest({ host: someVar, method: "GET", path: myPath })
```

## Comments

Line comments only: `// comment text`

## Imports

```
import functionName from "https://example.com/module.ss" perms {
  hosts: ["api.example.com"],
  secretsRead: ["api-key"]
} hash "sha256:abc123..."
```

Optional alias:
`import functionName as myAlias from "..." perms { ... } hash "..."`

The `perms` object declares what the imported function is allowed to do. The
`hash` is a SHA-256 content hash for integrity verification. Both are required.

## Built-in Ops Reference

### I/O Ops

#### httpRequest

Makes HTTPS requests. The `host` field is static (must be a literal).

```
response = httpRequest({
  host: "api.example.com",
  method: "POST",
  path: "/data",
  headers: { "content-type": "application/json", "authorization": authHeader },
  body: jsonBody
})
// response.status (number), response.body (string)
```

Methods: `"GET"`, `"POST"`, `"PUT"`, `"DELETE"`, `"PATCH"`. The `headers` and
`body` fields are optional. Always uses HTTPS.

#### doc

Attaches markdown documentation to a module or function. Runtime no-op — only
affects `safescript skill` output.

```
doc({ text: "Module-level documentation in markdown.\n\nCan span multiple lines." })
doc({ target: myFunction, text: "Describes what myFunction does.\n\nReturns a result object." })
```

`target` is optional (omit for module-level docs). Must appear at the top level
(module docs) or as a void call inside a function body (function docs).

#### assert

Errors if condition is false, otherwise returns `{ ok: true }`. Used for
testing.

```
assert({ condition: x == 42, message: "expected 42" })
```

Unary shorthand: `assert(x == 42)` — same as `assert({ condition: x == 42 })`

### Pure Ops (no side effects)

#### jsonParse

Parses a JSON string into a value.

```
parsed = jsonParse({ text: someJsonString })
// parsed.value (the parsed object/array/primitive)
```

Unary shorthand: `jsonParse(someJsonString)` — same as
`jsonParse({ text: someJsonString })`

#### jsonStringify

Serializes a value to a JSON string.

```
str = jsonStringify({ value: myObject })
// str.text (string)
```

Unary shorthand: `jsonStringify(myObject)`

#### stringConcat

Joins an array of strings into one string.

```
result = stringConcat({ parts: ["hello", " ", "world"] })
// result.result (string: "hello world")
```

Unary shorthand: `stringConcat(myPartsArray)`

#### sha256

Computes a SHA-256 hash, returned as base64url.

```
h = sha256({ data: "hello" })
// h.hash (string, base64url-encoded)
```

Unary shorthand: `sha256("hello")`

#### base64urlEncode

Encodes a string to base64url.

```
enc = base64urlEncode({ text: "hello" })
// enc.encoded (string)
```

Unary shorthand: `base64urlEncode("hello")` — same as
`base64urlEncode({ text: "hello" })`

#### base64urlDecode

Decodes a base64url string.

```
dec = base64urlDecode({ encoded: someB64 })
// dec.text (string)
```

Unary shorthand: `base64urlDecode(someB64)` — same as
`base64urlDecode({ encoded: someB64 })`

#### merge

Merges two objects (b overrides a).

```
merged = merge({ a: obj1, b: obj2 })
// merged.result (object)
```

#### pick

Picks specific keys from an object.

```
subset = pick({ obj: myObj, keys: ["name", "email"] })
// subset.result (object with only those keys)
```

### Crypto Ops

#### generateEd25519KeyPair

Generates an Ed25519 signing key pair.

```
keys = generateEd25519KeyPair()
// keys.publicKey (string), keys.privateKey (string)
```

#### ed25519Sign

Signs data with an Ed25519 private key.

```
sig = ed25519Sign({ data: message, privateKey: privKey })
// sig.signature (string)
```

#### generateX25519KeyPair

Generates an X25519 key exchange key pair.

```
keys = generateX25519KeyPair()
// keys.publicKey (string), keys.privateKey (string)
```

#### x25519DeriveKey

Derives a symmetric key from X25519 key exchange.

```
derived = x25519DeriveKey({
  myPrivateKey: myPriv,
  theirPublicKey: theirPub,
  salt: saltString,
  info: "context-label"
})
// derived.derivedKey (string)
```

#### aesGenerateKey

Generates a random AES-GCM-256 key.

```
k = aesGenerateKey()
// k.key (string)
```

#### aesEncrypt

Encrypts plaintext with AES-GCM.

```
enc = aesEncrypt({ plaintext: "secret data", key: aesKey })
// enc.ciphertext (string), enc.iv (string)
```

#### aesDecrypt

Decrypts AES-GCM ciphertext.

```
dec = aesDecrypt({ ciphertext: enc.ciphertext, iv: enc.iv, key: aesKey })
// dec.plaintext (string)
```

### Source Ops (non-deterministic)

#### timestamp

Returns the current Unix timestamp in milliseconds.

```
t = timestamp()
// t.timestamp (number)
```

#### randomBytes

Generates cryptographically random bytes, returned as base64url.

```
r = randomBytes({ length: 32 })
// r.bytes (string, base64url-encoded)
```

Unary shorthand: `randomBytes(32)`

## Complete Example

This function takes an API key and userId, fetches data from an API, and returns a processed result:

```
fetchUserName = (userId: string, apiKey: string): string => {
  path = stringConcat({ parts: ["/users/", userId] })
  response = httpRequest({
    host: "api.example.com",
    method: "GET",
    path: path.result,
    headers: { "authorization": apiKey }
  })
  parsed = jsonParse({ text: response.body })
  return parsed.value.name
}
```

## CLI

```
safescript run <file.ss> [fn] [--args '{"key":"val"}']
safescript signature <file.ss> [fn]
safescript transpile-ts <file.ss> [fn]
safescript transpile-py <file.ss> [fn]
safescript test <file.ss>
safescript skill <file.ss>
```

### Running programs

```
safescript run script.ss                # auto-detects main() or first function
safescript run script.ss myFn --args '{"name":"world"}'
```

### Running tests

`safescript test <file.ss>` runs all zero-input functions in a safescript file
and reports pass/fail. Use `assert` and `override` to mock side effects:

```
import { createDocument } from "../scripts/create-document.ss"

mockHttpRequest = (host: string, method: string, path: string, ...): { ... } => {
  return { status: 201, body: "..." }
}

testCreateDocument = (): { ok: boolean } => {
  result = override(createDocument, { httpRequest: mockHttpRequest })("Test", "Content", id)
  assert({ condition: result.status == 201 })
  return { ok: true }
}
```

No TypeScript wrapper needed — just the `.ss` file.

### Generating docs

`safescript skill <file.ss>` extracts `doc()` annotations from a safescript file
and generates markdown suitable for a SKILL.md or README.

```
doc({ text: "My module description..." })

myFn = (x: string): string => {
  doc({ target: myFn, text: "Takes a string, returns a string." })
  return x
}
```

Produces module-level prose followed by per-function `## myFn` sections.

## Differences from JavaScript & Quick Reference

Safescript is **NOT** JavaScript or TypeScript. It has a custom compiler,
parser, and runtime. If you try to write standard JS/TS, the parser will fail.
Always follow these syntax translations:

### 1. Variables & Declarations

- **In JavaScript:** `const x = 1;` or `let y = "hello";`
- **In Safescript:** No `const`, `let`, or `var` keywords. Semicolons are
  completely optional. Assign variables directly:
  ```typescript
  x = 1;
  y = "hello";
  ```

### 2. HTTP Requests & Fetching

- **In JavaScript:**
  `const res = await fetch("https://api.com/users"); const data = await res.json();`
- **In Safescript:** No `fetch` or `.json()`. Use the built-in `httpRequest` op
  (with static `host` literal) and `jsonParse` op:
  ```typescript
  res = httpRequest({ host: "api.com", method: "GET", path: "/users" });
  // Response has ONLY response.status (number) and response.body (string)

  parsed = jsonParse({ text: res.body }).value;
  // parsed is now the decoded object/array
  ```

### 3. Iteration & Loops

- **In JavaScript:** `for (let i = 0; i < arr.length; i++) { ... }` or
  `while (cond) { ... }`
- **In Safescript:** Loops are completely forbidden. You MUST use functional
  mappings: `map`, `filter`, or `reduce` to iterate:
  ```typescript
  double = (n: number): number => {
    return n * 2;
  };
  doubled = map(double, [1, 2, 3]);
  ```

### 4. String Operations & Concatenation

- **In JavaScript:** `const full = \`hello ${name}\`;`or`"order-" + id`
- **In Safescript:** No template literals. String concatenation with `+` only
  works if both operands are strings. To concatenate safely, use the
  `stringConcat` op:
  ```typescript
  path = stringConcat({ parts: ["/users/", userId, "/profile"] }).result;
  ```

### 5. String Prototype Methods

- **In JavaScript:** `name.indexOf("sub") !== -1`, `name.toLowerCase()`, or
  `name.replace("a", "b")`
- **In Safescript:** String prototype methods do not exist. Use built-in pure
  ops:
  - **Substring check:**
    `stringIncludes({ haystack: name, needle: "sub" }).result` (returns boolean)
  - **Lowercase:** `stringLower({ text: name }).result` (or unary:
    `stringLower(name).result`)
  - **Replace:**
    `stringReplace({ haystack: name, needle: "a", replacement: "b", all: true }).result`
  - **Regex match:**
    `stringRegex({ text: name, pattern: "order-(\\d+)" }).groups`
  - **Split:** `stringSplit({ text: name, delimiter: "," }).parts` (returns
    array)

### 6. Array Length

- **In Safescript:** Array length is natively supported via standard dot
  notation: `organizations.length`. (Getting a string's length is done via
  `stringSplit({ text: str, delimiter: "" }).parts.length`).

### 7. Returns & Blocks

- **In JavaScript:** `if (x) { return 1; }`
- **In Safescript:** Semicolons are optional, but **early returns are strictly
  forbidden**. The `return` statement can ONLY appear as the very last line of
  the function body. Variables assigned inside blocks (like `if`/`else`) persist
  outside of them:
  ```typescript
  res = "no";
  if (x == 2) {
    res = "yes";
  }
  return res;
  ```

---

### What's NOT Allowed

- No recursion (detected and rejected at parse time)
- No direct calls to user-defined functions — only via `map`/`filter`/`reduce`
- No classes, closures, or lambdas
- No `try`/`catch`/`throw`
- No `null` or `undefined`
- No destructuring or spread
- No logical operators (`&&`, `||`) — use ternary or `if`/`else`
- No bitwise operators
- No `function` keyword
- No `else if` (nest `if` inside `else` block instead)

---
> Source: [uriva/safescript](https://github.com/uriva/safescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
