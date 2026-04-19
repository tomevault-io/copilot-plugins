## req

> This documents the language. It is a DSL that reads like a sentence and runs in a shell.

# .cursor/rules/grammar.mdc

# req grammar - v0.1 specification

This documents the language. It is a DSL that reads like a sentence and runs in a shell.

## Command shape

    req <verb> <url> [clauses...]

Clauses are key=value pairs where keys are semantic words, not config words.

Order of clauses is free.

Unknown clause keys are errors.

## Verbs

- read - GET, print to stdout
- save - GET, write to file via to=
- send - default GET, POST if with= present
- upload - POST when attach= or with= present, else error
- watch - GET with SSE or polling
- inspect - HEAD only
- authenticate - login and store session state
- session show, session clear, session use - session management

## Clauses

| Clause     | Meaning                                     | Repeatable | Example                                                                 |
|-----------|---------------------------------------------|------------|-------------------------------------------------------------------------|
| using=    | HTTP method override                         | no         | using=PUT                                                               |
| include=  | Add headers, params, cookies                 | yes        | include='header: Authorization: Bearer token; param: q=search query'     |
| with=     | Request body                                 | no         | with=@user.json or with='{"name":"Adam"}'                               |
| expect=   | Assertions on response                       | no         | expect=status:200, header:Content-Type=application/json, contains:"ok"  |
| as=       | Output format for stdout                     | no         | as=json                                                                 |
| to=       | Destination path                             | no         | to=out.json                                                             |
| retry=    | Retry attempts for transient errors          | no         | retry=3                                                                 |
| under=    | Timeout or size limit                        | no         | under=30s or under=10MB                                                 |
| via=      | Proxy URL                                    | no         | via=http://proxy:8080                                                   |
| attach=   | Multipart parts for upload or send           | yes        | attach='part: name=avatar, file=@me.png; part: name=meta, value=xyz'    |
| follow=   | Redirect policy for write verbs              | no         | follow=smart                                                            |
| insecure= | Disable TLS verification for this request    | no         | insecure=true                                                           |

### include= value grammar

- One include clause may contain multiple items separated by semicolons that are not inside quotes.
- Allowed item tags:
  - header: Name: Value
  - param: key=value
  - cookie: key=value
  - basic: username:password

Merging
- Headers: keep all values for multi valued headers, else last wins.
- Params: repeated keys become repeated pairs in insertion order.
- Cookies: last value wins per cookie name.
- Basic Auth: sets Authorization header, overrides any existing Authorization header.

Quoting
- If an item payload contains a semicolon, quote the value.
- Backslash escapes allowed inside quoted values for the quote char and backslash.

Errors
- Unknown tag before the first colon.
- Header without Name colon Value.
- Param or cookie missing equals.
- Basic item missing colon (must be username:password format).
- Unquoted semicolon inside an item payload.

Examples

    include='header: Accept: application/json, application/problem+json; q=0.9'
    include='param: q=search query; param: tag=ai; param: tag=ml'
    include='cookie: session=abc; cookie: prefs="a=1; b=2"'
    include='basic: user:pass; header: Accept: application/json'

### attach= value grammar

- One attach may contain multiple items separated by semicolons that are not inside quotes.
- Allowed item tags:
  - part: name=..., file=@path  or  part: name=..., value=...
  - Optional filename=...
  - Optional type=media/type
  - boundary: TOKEN optional

Validation
- name is required.
- exactly one of file or value is required.
- for file parts, @path must exist at execution time.

Header behavior
- Any attach= forces multipart Content-Type with a generated boundary.
- If user included a manual Content-Type, override it and print a one line note.

Examples

    attach='part: name=avatar, file=@./me.png, filename=me.png, type=image/png'
    attach='part: name=meta, value={"name":"adam"}'
    attach='part: name=file, file=@./a.png; part: name=meta, value=xyz'

### with= body modes

- Inline text or JSON.
- @file to read from file.
- @- to read from stdin.

Content type
- If Content-Type is not set and inline begins with "{" or "[", infer application/json and note on stderr.
- An explicit Content-Type header always overrides inference.

### expect= assertions

Single clause with comma separated checks. All must pass.

Supported checks
- status:200
- header:Content-Type=application/json
- contains:"text"
- jsonpath:"$.items[0].id"
- matches:"^OK\\b"

Exit codes
- 0 success and expectations passed.
- 3 request ok but an expectation failed.

Failure messages must be concise and specific.

### Redirects

- read and save follow up to 5 redirects by default.
- write verbs do not follow by default.
- follow=smart for writes follows only 307 and 308, up to 5 hops.
- On 301, 302, 303 for writes, do not follow and print an advisory.

### Compression

- If user did not set Accept-Encoding, send "Accept-Encoding: gzip, br".
- Auto decompress gzip or br before as= and expect=.
- Print a one line note when decompression occurs.

### TLS

- insecure=true disables certificate verification for this request only.
- Print one line warning on stderr.

### Sessions

authenticate
- Follows redirects.
- Captures Set-Cookie.
- If response is JSON with a top level access_token, store it as a Bearer token.
- Store per host under user state dir with strict perms.

Auto use
- Any request to a host with a stored session auto applies cookies and Authorization unless caller includes those explicitly.
- Print "Using session for <host>" when applied.

session verbs
- session show <host> prints redacted info. as=json prints machine friendly form.
- session clear <host> deletes state.
- session use <host> prints env stub for shell scoping.

### Method defaults

- read GET
- save GET
- send GET by default, POST if with= present
- upload POST when attach= present, else POST if with= present, else error
- watch GET
- inspect HEAD
- authenticate POST if with= present, else require using=

### Multiplicity and ordering

- Singletons: using, with, expect, as, to, retry, under, via, insecure, follow.
- Repeatable: include, attach.
- Clause order is free.
- Explicit include of Authorization or Cookie overrides session.

### Token and quoting model

- Parser consumes argv tokens as provided by the shell.
- Do not re split on spaces.
- Values containing semicolons must be quoted.
- Backslash escapes allowed inside quotes for the quote char and backslash.
- Environment variables are expanded by the shell before argv.

### Errors that must be loud and specific

- Unknown clause key.
- Duplicate singletons.
- include item with unknown tag or malformed payload.
- header item missing Name colon Value.
- param or cookie missing equals.
- basic item missing colon (must be username:password format).
- unquoted semicolon in an item payload.
- attach part missing name, or missing both file and value, or providing both.
- URL parse failure.
- file path not found for with or attach.
- timeout or size limit exceeded.
- TLS error when insecure=false.

### Error message examples

Each error class must produce a specific, actionable message. Examples:

Unknown clause key

    $ req read https://example.com invalid=clause
    Error: parse error at position 2 (token: "invalid"): unknown clause

Duplicate singleton

    $ req read https://example.com with=test with=test2
    Error: parse error at position 3 (token: "with"): duplicate singleton clause 'with' (did you mean "remove duplicate 'with=' clause"?)

Unquoted semicolon in include

    $ req read https://example.com include='param: q=test;value'
    Error: parse error at position 2 (token: "include"): unquoted semicolon in include item

Malformed header (missing Name: Value)

    $ req read https://example.com include='header: InvalidHeader'
    Error: parse error at position 2 (token: "include"): header item missing Name: Value format

Missing equals in param or cookie

    $ req read https://example.com include='param: q'
    Error: parse error at position 2 (token: "include"): param missing equals

Basic item missing colon

    $ req read https://example.com include='basic: userpass'
    Error: parse error at position 2 (token: "include"): basic item must be in format username:password: basic: userpass

Attach part missing name

    $ req upload https://example.com attach='part: file=@test.png'
    Error: parse error at position 2 (token: "attach"): attach part missing name

Attach part with both file and value

    $ req upload https://example.com attach='part: name=test, file=@test.png, value=text'
    Error: parse error at position 2 (token: "attach"): attach part cannot have both file and value

File not found for with

    $ req send https://example.com with=@nonexistent.json
    Error: file not found: nonexistent.json

File not found for attach

    $ req upload https://example.com attach='part: name=file, file=@nonexistent.png'
    Error: file not found: nonexistent.png

### Output contracts

stdout
- Response body, formatted per as=.
- For save, write to file and keep stdout empty unless a specific mode says otherwise.

stderr
- Compact meta block: status, url, duration, bytes, content type.
- Notices: session use, decompression, redirect trace, TLS warning, multipart override note.
- Secrets redacted in meta lines.

### End to end examples

Read with params and header

    req read https://api.example.com/search \
      include='param: q=search query; header: X-Trace: 1' \
      as=json

Write JSON with auth and assertion

    req send https://api.example.com/users \
      using=POST \
      include='header: Authorization: Bearer $TOKEN' \
      with='{"name":"Adam"}' \
      expect=status:201, header:Content-Type=application/json \
      as=json

Multipart upload

    req upload https://api.example.com/upload \
      attach='part: name=file, file=@./avatar.png, type=image/png; part: name=meta, value={"name":"adam"}' \
      as=json

Authenticate then use session

    req authenticate https://api.example.com/login \
      using=POST \
      with='{"user":"adam","pass":"xyz"}'

    req read https://api.example.com/me as=json

Write with safe redirects

    req send https://api.example.com/endpoint \
      using=POST \
      with='{"a":1}' \
      follow=smart \
      expect=status:200

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adammpkins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
