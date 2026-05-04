## clojure-plywright

> Provides a Playwright `APIRequestContext` bound to `client`.

# Agent Development Guide — clojure-playwright

This document is a reference for AI agents (and developers) working on the
`clojure-playwright` library itself.

---

## 1. What this library is

`clojure-playwright` is a thin Clojure wrapper around the **Playwright Java SDK**.
It exists because:

- Raw Playwright interop is verbose — browser/context/tracing lifecycle is
  repeated in every test.
- We want every test to automatically produce a Playwright Trace Viewer recording
  without any per-test boilerplate.

The library is intentionally small.  It is **not** a framework — it is a few
macros and helpers that handle the lifecycle so test authors can focus on
assertions.

---

## 2. Repository Layout

```
clojure-playwright/
├── deps.edn                    # library + aliases (:test, :nrepl, :clean, …)
├── tests.edn                   # Kaocha config for :test alias
├── src/
│   └── testing/
│       ├── pw.clj              # Primary public API (with-ui, with-api, step, …)
│       ├── pw_report.clj       # Aggregate HTML report generator
│       ├── pw_http_server.clj  # Pure-Java HTTP server for trace viewer
│       └── repl.clj            # REPL helpers (page atom, restart, stop)
└── test/
    └── samples/
        └── pw_demo_test.clj    # Sample / smoke tests for the library itself
```

---

## 3. Public API (`testing.pw`)

All public symbols live in `src/testing/pw.clj`.

### `with-ui`

Main test macro.  Manages browser + tracing lifecycle.  Binds the Playwright
`Page` both lexically (to the name you choose) and dynamically to
`*current-page*`.

```clojure
(with-ui [page]
  (step "Navigate" #(.navigate page "https://example.com"))
  (step "Assert"   #(is (= "Example Domain" (.textContent (.locator page "h1"))))))

;; with options
(with-ui [page {:headless false :output-dir "target/my-traces"}]
  ...)
```

Options: `:headless` (default `true`), `:output-dir` (default `"target/pw-traces"`),
`:slow-mo` (default `0`, or `80` when headed).

`with-pw` is a **deprecated alias** — keep it for back-compat but do not use
it in new code.

### `with-api`

Provides a Playwright `APIRequestContext` bound to `client`.

- **Attached** (inside `with-ui`): derives client from `BrowserContext.request()` —
  HTTP calls appear in the same trace as browser actions.
- **Standalone**: starts a fresh headless browser + tracing context, produces its
  own `trace.zip`, then cleans everything up.

```clojure
;; Standalone
(with-api [client]
  (let [resp (.get client "https://api.example.com/status" nil)]
    (is (= 200 (.status resp)))))

;; Inside with-ui (attached, same trace)
(with-ui [page]
  (step "REST call"
    #(with-api [client]
       (is (= 200 (.status (.get client url nil)))))))
```

Important implementation note: `BrowserContext.request()` returns an
`APIRequestContext` **directly** — it is NOT a factory and `.newContext` must
not be called on it.  Only `Playwright.request()` is the factory.

### `step`

Wraps a thunk in a named Playwright trace group.  Reads `*current-page*`
automatically.

```clojure
(step "Fill login form"
  (fn []
    (.fill *current-page* "#email" "user@example.com")
    (.click *current-page* "button[type=submit]")))
```

### `pw-screenshot`

Captures a labelled screenshot and logs it.  Reads `*current-page*`.

```clojure
(pw-screenshot "after-submit")
```

### `start-browser` / `stop-browser`

Low-level helpers used internally by `with-ui` and `with-api`.  Exposed
publicly for advanced use.  `start-browser` returns a map of
`{:playwright :browser :context :page :trace-path}`.

### Dynamic vars

| Var | Bound by | Purpose |
|---|---|---|
| `*current-page*` | `with-ui`, `with-api` (standalone) | Current Playwright `Page` |
| `*current-api*` | `with-api` | Current `APIRequestContext` |

---

## 4. Sample Tests (`test/samples/pw_demo_test.clj`)

The sample tests double as smoke tests for the library.

| Test | What it verifies |
|---|---|
| `example-dot-com-test` | Basic navigation, title/heading assertions, explicit screenshot |
| `playwright-website-test` | Multi-step test with options map (`output-dir`) |
| `failing-step-demo` | **Intentionally fails** — verifies trace captures failure state |
| `with-api-standalone-test` | `with-api` with no active browser; GET + POST |
| `with-api-attached-test` | `with-api` nested inside `with-ui`; same trace |

Run:
```bash
clojure -M:test
```

Expected result: 6 tests, 1 failure (`failing-step-demo` — intentional).

---

## 5. Interactive REPL

```bash
clojure -M:nrepl
# nREPL on port 7888; headed Chromium opens immediately
```

```clojure
(require '[testing.repl :refer [page restart stop]])

(.navigate @page "https://example.com")
(.textContent (.locator @page "h1"))   ;; => "Example Domain"

(restart)  ; fresh browser
(stop)     ; close everything
```

---

## 6. Aliases

| Alias | Command | What it does |
|---|---|---|
| `:test` | `clojure -M:test` | Run tests via Kaocha |
| `:aggregate` | `clojure -M:aggregate` | Copy trace viewer assets + generate `index.html` from existing zips |
| `:traces` | `clojure -M:traces` | Serve the report over HTTP at `http://localhost:8080` |
| `:nrepl` | `clojure -M:nrepl` | Start headed browser + nREPL (port 7888) |
| `:clean` | `clojure -M:clean` | Delete `target/` |

---

## 7. Design Constraints

- **No JUnit dependency** — this is Playwright on the JVM only.
- **No new framework** — keep the API surface minimal.  If something can be done
  with raw Playwright interop in 1-2 lines, do not abstract it.
- **Every test gets a trace** — `with-ui` and `with-api` (standalone) always
  produce a `target/pw-traces/<name>-<ts>.zip`.
- **LSP false positives are expected** — clj-kondo / clojure-lsp cannot resolve
  macro-introduced bindings (`page`, `client`, `*current-page*` inside body
  forms).  These are cosmetic and do not affect compilation or test execution.
- **`with-pw` is a deprecated alias** — keep it for back-compat; it just
  delegates to `with-ui`.

---

## 8. Making Changes

1. Edit `src/testing/pw.clj`.
2. Run `clojure -M:test` — expect 6 tests, 1 intentional failure.
3. If you add a new public symbol, add a sample test in `pw_demo_test.clj`
   and document it in this file and in `README.md`.

---
> Source: [alpha-prosoft/clojure-plywright](https://github.com/alpha-prosoft/clojure-plywright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
