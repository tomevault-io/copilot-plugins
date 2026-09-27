## robot

> A simple calculator web app in Clojure. The server ([http-kit](https://github.com/http-kit/http-kit))

# AGENTS.md — robot

A simple calculator web app in Clojure. The server ([http-kit](https://github.com/http-kit/http-kit))
owns all state — the current expression and its result live in a server-side
atom. The frontend is a single HTML page driven by
[Datastar](https://data-star.dev) (v1.0.3): every key press is a `@post` to the
backend, and the backend answers with a Datastar SSE event that morphs the
display fragment. There is no client-side calculator logic and no JavaScript
written by us.

Built with the Clojure CLI (`deps.edn`), not Leiningen. Main namespace:
`robot.core`.

## How it works

```
browser                                   server (http-kit)
───────                                   ─────────────────
GET /            ───────────────────────▶ web/page-html @state
                 ◀─── text/html ────────  full page incl. <div id="display">

click "7"        ───────────────────────▶ (swap! state calc/press "7")
  data-on:click="@post('/press/7')"
                 ◀─── text/event-stream ─ event: datastar-patch-elements
                                          data: elements <div id="display">…</div>
Datastar morphs #display in place
```

- `robot.calc` — pure functions: `press` (state transition per key) and
  `evaluate` (hand-written recursive-descent parser for `+ - * / ( )` and
  decimals). **Never** evaluate user input with `read-string`/`eval`.
- `robot.web` — rendering (`page-html`, `display-html`), the SSE encoder
  (`patch-elements-sse`), and the ring handler (`make-handler`).
- `robot.core` — the `state` atom, `start!`/`stop!`, and `-main`.

Key names in URLs are ASCII words (`plus`, `times`, `equals`, `clear`,
`back`, …) mapped to characters by `calc/keys->chars`; see the `keypad`
table in `robot.web` for the full list.

## Layout

```
robot/
├── deps.edn
├── src/robot/
│   ├── core.clj        # (ns robot.core)  server lifecycle + -main
│   ├── calc.clj        # (ns robot.calc)  pure calculator logic
│   └── web.clj         # (ns robot.web)   HTML, SSE, routing
├── test/robot/
│   ├── calc_test.clj
│   └── web_test.clj
└── .gitignore
```

Namespace ↔ path mapping is standard: `robot.calc-test` →
`test/robot/calc_test.clj` (dashes become underscores in filenames).

## Scaffold from scratch

If the files above are missing, create them exactly as follows.

### `deps.edn`

```clojure
{:paths ["src"]
 :deps  {org.clojure/clojure {:mvn/version "1.12.3"}
         http-kit/http-kit     {:mvn/version "2.8.1"}}
 :aliases
 {:run  {:main-opts ["-m" "robot.core"]}
  :test {:extra-paths ["test"]
         :extra-deps  {io.github.cognitect-labs/test-runner
                       {:git/tag "v0.5.1" :git/sha "dfb30dd"}}
         :main-opts   ["-m" "cognitect.test-runner"]
         :exec-fn     cognitect.test-runner.api/test}
  :nrepl {:extra-deps {nrepl/nrepl {:mvn/version "1.3.1"}}
          :main-opts  ["-m" "nrepl.cmdline" "--interactive"]}}}
```

### `src/robot/calc.clj`

```clojure
(ns robot.calc
  "Pure calculator: state transitions and expression evaluation.
   No I/O, no atoms — everything here is a plain function."
  (:require [clojure.string :as str]))

(def initial
  "Fresh calculator state."
  {:expr "" :result nil})

;; ---------------------------------------------------------------------------
;; Evaluation: a small recursive-descent parser for + - * / ( ) and decimals.
;; Deliberately not `read-string`/`eval` — never evaluate user input as code.

(defn- tokenize [s]
  (let [toks (re-seq #"\d+(?:\.\d+)?|[-+*/()]" s)]
    (when (not= (apply str toks) (str/replace s #"\s" ""))
      (throw (ex-info "invalid characters" {:expr s})))
    toks))

(declare parse-expr)

(defn- parse-atom [[t & more]]
  (cond
    (nil? t)  (throw (ex-info "unexpected end of input" {}))
    (= t "(") (let [[v rest] (parse-expr more)]
                (when (not= (first rest) ")")
                  (throw (ex-info "expected )" {})))
                [v (next rest)])
    (= t "-") (let [[v rest] (parse-atom more)] [(- v) rest])
    (re-matches #"\d+(?:\.\d+)?" t) [(parse-double t) more]
    :else (throw (ex-info (str "unexpected " t) {:token t}))))

(defn- parse-binary
  "Left-assoc binary operators at one precedence level."
  [ops next-level toks]
  (loop [[v rest] (next-level toks)]
    (if-let [f (ops (first rest))]
      (let [[v2 rest2] (next-level (next rest))]
        (recur [(f v v2) rest2]))
      [v rest])))

(defn- parse-term [toks] (parse-binary {"*" * "/" /} parse-atom toks))
(defn- parse-expr [toks] (parse-binary {"+" + "-" -} parse-term toks))

(defn evaluate
  "Evaluate an arithmetic expression string to a double.
   Throws ex-info on malformed input or non-finite results."
  [s]
  (let [[v rest] (parse-expr (tokenize s))]
    (when (seq rest)
      (throw (ex-info "trailing input" {:rest rest})))
    (when-not (Double/isFinite v)
      (throw (ex-info "non-finite result" {:value v})))
    v))

(defn format-number
  "Render a double without a trailing .0 when it is integral."
  [^double v]
  (if (== v (Math/rint v))
    (str (long v))
    (str v)))

;; ---------------------------------------------------------------------------
;; Key presses

(def keys->chars
  "URL-safe key names (used in /press/<key>) to the character they append."
  {"0" "0" "1" "1" "2" "2" "3" "3" "4" "4"
   "5" "5" "6" "6" "7" "7" "8" "8" "9" "9"
   "dot" "." "plus" "+" "minus" "-" "times" "*" "divide" "/"
   "lparen" "(" "rparen" ")"})

(defn press
  "Apply one key press to a calculator state. Unknown keys are ignored."
  [{:keys [expr] :as state} key]
  (case key
    "clear"  initial
    "back"   (assoc state :expr (subs expr 0 (max 0 (dec (count expr))))
                          :result nil)
    "equals" (try
               (assoc state :result (format-number (evaluate expr)))
               (catch Exception _
                 (assoc state :result "Error")))
    (if-let [c (keys->chars key)]
      (assoc state :expr (str expr c) :result nil)
      state)))
```

### `src/robot/web.clj`

```clojure
(ns robot.web
  "HTTP layer: page rendering, Datastar SSE responses, routing."
  (:require [clojure.string :as str]
            [robot.calc :as calc]))

(def datastar-src
  "https://cdn.jsdelivr.net/gh/starfederation/datastar@1.0.3/bundles/datastar.js")

;; ---------------------------------------------------------------------------
;; Rendering

(defn- escape-html [s]
  (str/escape (str s) {\& "&amp;" \< "&lt;" \> "&gt;" \" "&quot;"}))

(defn display-html
  "The fragment Datastar morphs after every key press. Must keep id=\"display\"."
  [{:keys [expr result]}]
  (str "<div id=\"display\">"
       "<div class=\"expr\">" (escape-html (if (str/blank? expr) "0" expr)) "</div>"
       "<div class=\"result\">" (escape-html (or result "")) "</div>"
       "</div>"))

(defn- button [label key]
  (str "<button data-on:click=\"@post('/press/" key "')\">" label "</button>"))

(def ^:private keypad
  [["C" "clear"] ["(" "lparen"] [")" "rparen"] ["⌫" "back"]
   ["7" "7"] ["8" "8"] ["9" "9"] ["÷" "divide"]
   ["4" "4"] ["5" "5"] ["6" "6"] ["×" "times"]
   ["1" "1"] ["2" "2"] ["3" "3"] ["−" "minus"]
   ["0" "0"] ["." "dot"] ["=" "equals"] ["+" "plus"]])

(defn page-html [state]
  (str "<!doctype html><html><head><meta charset=\"utf-8\">"
       "<title>robot calc</title>"
       "<script type=\"module\" src=\"" datastar-src "\"></script>"
       "<style>"
       "body{font-family:system-ui,sans-serif;display:grid;place-items:center;min-height:100vh;margin:0;background:#111;color:#eee}"
       ".calc{width:280px;background:#222;border-radius:12px;padding:16px}"
       "#display{background:#000;border-radius:8px;padding:12px;margin-bottom:12px;text-align:right;min-height:3.5em}"
       ".expr{font-size:1.25rem;word-break:break-all}.result{font-size:1rem;color:#8f8;min-height:1.2em}"
       ".keys{display:grid;grid-template-columns:repeat(4,1fr);gap:8px}"
       "button{font-size:1.25rem;padding:12px 0;border:0;border-radius:8px;background:#444;color:#eee;cursor:pointer}"
       "button:hover{background:#555}"
       "</style></head><body><div class=\"calc\">"
       (display-html state)
       "<div class=\"keys\">" (apply str (map #(apply button %) keypad)) "</div>"
       "</div></body></html>"))

;; ---------------------------------------------------------------------------
;; Datastar SSE

(defn patch-elements-sse
  "Body of a one-shot SSE response carrying a `datastar-patch-elements` event.
   Every line of the HTML must be prefixed with `data: elements `."
  [html]
  (str "event: datastar-patch-elements\n"
       (str/join (map #(str "data: elements " % "\n") (str/split-lines html)))
       "\n"))

(defn- sse-response [body]
  {:status  200
   :headers {"Content-Type"  "text/event-stream"
             "Cache-Control" "no-cache"}
   :body    body})

(defn- html-response [body]
  {:status 200 :headers {"Content-Type" "text/html; charset=utf-8"} :body body})

;; ---------------------------------------------------------------------------
;; Routing

(defn make-handler
  "Ring handler closed over a state atom (so tests can supply their own)."
  [state]
  (fn [{:keys [request-method uri]}]
    (cond
      (and (= :get request-method) (= uri "/"))
      (html-response (page-html @state))

      (and (= :post request-method) (str/starts-with? uri "/press/"))
      (let [key (subs uri (count "/press/"))]
        (sse-response (patch-elements-sse (display-html (swap! state calc/press key)))))

      :else
      {:status 404 :headers {"Content-Type" "text/plain"} :body "not found"})))
```

### `src/robot/core.clj`

```clojure
(ns robot.core
  "Entry point: starts the http-kit server."
  (:require [org.httpkit.server :as hk]
            [robot.calc :as calc]
            [robot.web :as web])
  (:gen-class))

(defonce state (atom calc/initial))

(defonce ^:private server (atom nil))

(defn start!
  "Start the server on `port`. Returns the stop fn."
  [port]
  (when @server (@server))
  (reset! server (hk/run-server (web/make-handler state) {:port port}))
  (println (str "robot calc listening on http://localhost:" port))
  @server)

(defn stop! []
  (when-let [s @server] (s) (reset! server nil)))

(defn -main [& [port]]
  (start! (if port (parse-long port) 8080)))
```

### `test/robot/calc_test.clj`

```clojure
(ns robot.calc-test
  (:require [clojure.test :refer [deftest is testing]]
            [robot.calc :as calc]))

(deftest evaluate-test
  (testing "precedence and associativity"
    (is (== 7 (calc/evaluate "1+2*3")))
    (is (== 9 (calc/evaluate "(1+2)*3")))
    (is (== 1 (calc/evaluate "8/4/2")))
    (is (== -1 (calc/evaluate "-3+2"))))
  (testing "decimals"
    (is (== 0.75 (calc/evaluate "0.5+0.25"))))
  (testing "errors"
    (is (thrown? Exception (calc/evaluate "1+")))
    (is (thrown? Exception (calc/evaluate "1/0")))
    (is (thrown? Exception (calc/evaluate "(1+2")))
    (is (thrown? Exception (calc/evaluate "abc")))))

(deftest format-number-test
  (is (= "3" (calc/format-number 3.0)))
  (is (= "0.5" (calc/format-number 0.5))))

(deftest press-test
  (let [tap (fn [keys] (reduce calc/press calc/initial keys))]
    (is (= {:expr "1+2" :result nil} (tap ["1" "plus" "2"])))
    (is (= {:expr "1+2" :result "3"} (tap ["1" "plus" "2" "equals"])))
    (is (= {:expr "1+" :result nil}  (tap ["1" "plus" "2" "back"])))
    (is (= calc/initial (tap ["1" "plus" "clear"])))
    (is (= {:expr "1+" :result "Error"} (tap ["1" "plus" "equals"])))
    (is (= calc/initial (tap ["bogus"])) "unknown keys are ignored")
    (is (= "" (:expr (calc/press calc/initial "back"))) "backspace on empty is safe")))
```

### `test/robot/web_test.clj`

```clojure
(ns robot.web-test
  (:require [clojure.string :as str]
            [clojure.test :refer [deftest is testing]]
            [robot.calc :as calc]
            [robot.web :as web]))

(defn- handler [] (web/make-handler (atom calc/initial)))

(deftest index-test
  (let [{:keys [status headers body]} ((handler) {:request-method :get :uri "/"})]
    (is (= 200 status))
    (is (str/starts-with? (headers "Content-Type") "text/html"))
    (is (str/includes? body "datastar"))
    (is (str/includes? body "id=\"display\""))
    (is (str/includes? body "@post('/press/7')"))))

(deftest press-test
  (let [h (handler)
        press #(h {:request-method :post :uri (str "/press/" %)})]
    (testing "state accumulates across requests"
      (press "1") (press "plus") (press "2")
      (let [{:keys [status headers body]} (press "equals")]
        (is (= 200 status))
        (is (= "text/event-stream" (headers "Content-Type")))
        (is (str/starts-with? body "event: datastar-patch-elements\n"))
        (is (str/includes? body "data: elements <div id=\"display\">"))
        (is (str/includes? body "1+2"))
        (is (str/includes? body "<div class=\"result\">3</div>"))))))

(deftest not-found-test
  (is (= 404 (:status ((handler) {:request-method :get :uri "/nope"})))))

(deftest sse-format-test
  (is (= "event: datastar-patch-elements\ndata: elements <a>\ndata: elements <b>\n\n"
         (web/patch-elements-sse "<a>\n<b>"))))
```

### `.gitignore`

```
.cpcache/
.nrepl-port
target/
*.class
.lsp/
.clj-kondo/.cache/
```

## Commands

Run all from the project root.

| Task            | Command                                          |
|-----------------|--------------------------------------------------|
| Run server      | `clojure -M:run` (port 8080) or `clojure -M:run 3000` |
| Run tests       | `clojure -M:test`                                |
| REPL            | `clojure -M:nrepl` (or plain `clj`)              |
| Check it loads  | `clojure -M -e "(require 'robot.core)"`          |
| Smoke test      | `curl -X POST localhost:8080/press/7` → SSE body |

From a REPL: `(require 'robot.core) (robot.core/start! 8080)` /
`(robot.core/stop!)`. Reloading `robot.web` is enough to see handler
changes — `make-handler` is re-invoked only on `start!`, so call `stop!`
then `start!` after editing routes.

Toolchain on this machine: Clojure CLI 1.12.3, OpenJDK 25.

## Datastar cheat sheet (v1.0.3)

Only what this project uses — full reference at https://data-star.dev.

- Script: `<script type="module" src="https://cdn.jsdelivr.net/gh/starfederation/datastar@1.0.3/bundles/datastar.js">`
- Attribute: `data-on:click="@post('/url')"` (colon form, `@get`/`@post`/…).
- Response: `Content-Type: text/event-stream`, body is one or more events:
  ```
  event: datastar-patch-elements
  data: elements <div id="display">…</div>

  ```
  Default mode is `outer` — the element is matched by `id` and morphed. Every
  line of multi-line HTML needs its own `data: elements ` prefix, and the
  event ends with a blank line. `web/patch-elements-sse` does this.
- Signals (`data-signals`, `data-bind`, `data-text`, `datastar-patch-signals`)
  are not used yet; if you add them, note that `@get` sends signals as a
  `?datastar=<json>` query param and `@post` sends them as a JSON body.

## Conventions

- Keep `robot.calc` pure and free of I/O; all state mutation happens through
  the single `swap!` in `robot.web`'s `/press/` route.
- Any fragment the server patches must carry a stable `id` and be produced
  by one render fn used for both the initial page and the SSE patch
  (`display-html` is the model).
- Every new `src/robot/x.clj` gets a matching `test/robot/x_test.clj`.
  `web_test` calls the handler directly with ring request maps — no need to
  bind a port in tests.
- Add dependencies to `:deps` in `deps.edn`; test-only deps go under the
  `:test` alias's `:extra-deps`. No routing lib, no templating lib, no JSON
  lib unless a feature genuinely needs one.
- Prefer `clojure.test` — no extra test framework.
- Don't add a Leiningen `project.clj`, a `build.clj`, or a `pom.xml` unless
  asked; `deps.edn` is the single source of truth.
- Verify changes with `clojure -M:test` before reporting done.

---
> Source: [jjsullivan5196/robot](https://github.com/jjsullivan5196/robot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
