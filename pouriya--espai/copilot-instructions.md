## espai

> Coding conventions for this project. Follow these when writing or modifying code.

# Agent Guidelines

Coding conventions for this project. Follow these when writing or modifying code.

## Context: kept plain

The whole native host is one [`main.rs`](src/main.rs) with two
dependencies (`serde_json`, `tiny_http`), and the extension is six files the browser loads
as they are — no bundler, no npm, no build step beyond copying them next to the manifest
rendered from the template in [`extension/`](extension/). Before
adding a dependency or splitting a module, be sure the thing you are adding pays for the
loss of "read one file, understand the system". `main.rs` stays one file; if it passes
roughly 1200 lines, that is the moment to reconsider, not before. Count the body above
`#[cfg(test)]` — the test module costs a reader nothing. **That body is at ~1194 lines
today, so the next feature of any size is the one that triggers this.** Splitting is a
decision to raise with whoever owns the project, not one to take while doing something
else.

The style rules below favour locally-readable code over idiomatic density. They are
choices, not oversights — do not "modernize" them into iterator chains, combinator
pipelines, or extracted abstractions.

## Generic primitives, never site adapters

espai must not grow a `twitter.js` or an `instagram.js`. Those sites regenerate their
markup continuously and every selector committed here would be broken within weeks, with
nothing to warn us but a user reporting that likes silently stopped working. The three
capabilities that make an app site tractable — a snapshot with refs, trusted input, and the
network capture — are all site-agnostic, and together they are enough. Anything
site-shaped belongs in the agent's prompt, where it costs nothing to be wrong.

## Never write to stdout

**`println!` is a fatal bug.** stdout is the native messaging channel, and one stray byte
desynchronises the length-prefixed frame stream: Chrome sees a garbage length, drops the
port, and kills this process. There is no recovery and the symptom is a silent
disconnection, not an error.

Every diagnostic goes to stderr via `eprintln!`, prefixed `espai:`. Chrome inherits our
stderr from the browser process, so it lands wherever Chrome was started from — the
terminal if launched from one, nowhere if launched from the desktop. Treat it as best
effort. Nothing may depend on a human seeing it.

## How Chrome starts and kills this process

Chrome spawns one host process **per `connectNative()` call** — not per tab, not per
browser. It is lazy: nothing runs until the service worker connects. `argv[1]` is the
calling extension's origin. The working directory is unspecified, so never use a relative
path.

Chrome, Chromium, Edge and Brave each read the host manifest from **their own profile
root**, and a manifest in the wrong one is silently ignored — the symptom is an extension
that loads fine, reports no error a user would notice, and never starts the host.
`make chrome` and `make chromium` write a ready `com.espai.host.json` beside each built
extension; `make install-chrome` and `make install-chromium` copy it into the root that
browser actually reads.

Those install targets **fail on a profile root that does not exist** rather than creating
one, and that is not a convenience worth adding later. Creating `~/.config/google-chrome` on
a machine that only runs Chromium leaves a manifest nothing will ever read and a directory
that looks installed — the exact failure the check exists to catch. Edge and Brave have no
make target for the same reason: a root this repo has to guess is a root it can guess wrong.
[`install.sh`](install.sh) writes all four, but it never guesses either — it copies into
every root that already exists and creates none.

Every manifest points at `bin/espai`, which `release` copies out of `target/`. A path into
`target/` would break on the next `cargo clean` or debug build, and break silently, since a
missing host binary looks exactly like a missing manifest.

The two halves version together: `version` in [`Cargo.toml`](Cargo.toml) and in
[`manifest.chromium.json.in`](extension/manifest.chromium.json.in) name the same release
and are bumped in the same commit. They ship as one thing and speak a private frame
protocol to each other, so a version that identifies the extension but not the host binary
behind it describes half a system. `make check-version` enforces the pairing and runs as part
of `make all`. Note that Chrome refuses to load an extension whose version goes backwards, so
the manifest only ever moves forward.

Shutdown arrives as **EOF on stdin**, and `UnexpectedEof` on the length read is therefore
the ordinary exit path, not an error to log loudly. The host deregisters and exits; a host
that ignored EOF would be killed anyway.

## Why there is a heartbeat

The MV3 service worker that owns our port is killed after ~30s idle, and killing it closes
the port and kills this process. Native port traffic resets Chrome's idle timer, so the
20s ping in `main` is the only thing keeping espai reachable between agent calls. It looks
like a no-op — replies to `id: 0` are discarded — and deleting it appears to change
nothing, until the eye is dead every time an agent actually reaches for it.

`chrome.alarms` in [`background.js`](extension/background.js) is the other half: at 30s, the
shortest period Chrome permits, it resurrects a worker that died regardless. The ping
prevents the death; the alarm recovers from it. Both are load-bearing.

## Input goes through the debugger, not the DOM

`element.click()` and a hand-built `dispatchEvent` produce events whose `isTrusted` is
false, and React — along with most of what the large app sites are built on — ignores
untrusted events on its own controls. The failure is silent: the call returns successfully,
the page does nothing, and the agent has no way to tell the difference from a bad selector.
So every click, hover and keystroke goes through `chrome.debugger` and
`Input.dispatchMouseEvent` / `Input.dispatchKeyEvent`, which enter the browser's real input
pipeline. **Do not "simplify" [`cdp.js`](extension/cdp.js) back into content-script events.**

The consequences were weighed and accepted: an attached debugger shows a banner the user
cannot dismiss, and only one client may attach to a tab, so DevTools and espai cannot both
own one. Attachment is per tab and kept for the tab's life rather than taken per action —
the banner appears either way, and detaching between actions would tear down the network
capture in the gaps.

Coordinates are the subtle part. `Input.*` dispatches at top-level viewport coordinates, so
[`actions.js`](extension/actions.js) scrolls the element into view, reads its rectangle
**after** the scroll, and walks up the `frameElement` chain to convert. A cross-origin
iframe cannot see its own position in the parent, so it reports `offsetUnknown` and the
command fails with an explanation. That is deliberate: a guessed coordinate clicks
whatever happens to be at that spot, which is worse than an error.

## Chromium only. Firefox is not supported

**Do not add Firefox support back.** It was built and then removed deliberately, and a
patch that reintroduces it will be rejected on sight.

Firefox implements none of `chrome.debugger`. Everything in the section above — trusted
input, screenshots, `eval`, the network capture — is unavailable there, which leaves reads
and navigation: not espai, just a worse way to fetch a page. It also has no MV3 service
worker, so the background half needs a second shape, and no `key` field, so the id and the
host manifest need a third. Three divergences to maintain, for a build that cannot click.

The cost was not only the build. Every `if (!chrome.debugger)` guard in the extension
existed for Firefox alone, and each one turned a browser that cannot do the thing into a
browser that says nothing until an agent notices its clicks are silently doing nothing.
Those guards are gone with it — in Chromium the `debugger` permission is declared and
always present, so a guard against its absence is dead code that hides a real failure.

If someone wants a Firefox eye, it is a different project with a different mechanism, not
a flag here.

## Network capture is pushed, never buffered here

The service worker observes the network but must not remember it. It is killed after ~30s
idle and everything it held dies with it, so anything an agent asked for later would be
gone. Every captured event is posted to the host the moment it happens and lives in the
ring buffer in `main.rs`, which lasts as long as the browser does.

This is why frames from the browser come in two shapes. A reply carries the `id` of the
request it answers; an **event** carries none and is appended to the buffer instead. Both
arrive on the same pipe and the stdin loop tells them apart by the presence of `event`.

Response bodies are the exception and are fetched on demand, because holding every body
would exhaust the worker on any image-heavy feed. Chrome discards them once a page
navigates away, so a `null` body is expected rather than a bug.

## Refs, and why a stale one is 410

A ref is an agent's handle on one element, handed out by the snapshot. It exists because
app sites generate their class names, so an agent cannot write a selector that will still
match — and should not be encouraged to try.

`snapshot.js` keeps ref → `WeakRef(element)` and element → ref. The second map makes a ref
stable: the same element keeps the same ref across snapshots, so a ref an agent already
holds does not silently start meaning something else. The first being weak is what lets a
detached element be recognised as gone rather than resurrected.

A ref that no longer resolves is **410, not 502**. The distinction is the whole point: 502
means "that failed, maybe retry", while 410 means "the page moved on, snapshot again", and
only one of those is useful advice. The `stale` flag is carried from `content.js` through
the worker to `ask_browser` for exactly this.

Never key a deduplication on a ref. Virtualized lists recycle one DOM node across many
rows, so the node is not the row's identity — this is why `collect` hashes field values
instead.

## The registry, and why liveness is probed

`~/espai.toml` is how an agent finds us at all: ports come from the OS at startup,
so there is nothing to hardcode. One entry per process, rewritten at startup and at clean
exit.

It lives at the top of `$HOME`, not under `.config`, and that is deliberate. Its reader is
usually an agent that was handed a one-line instruction and nothing else, so a path it can
guess — and that shows up in a bare `ls ~` — beats the tidier location. Do not move it back
without a way for that reader to still find it.

**It is TOML for the comments, not for the syntax.** `REGISTRY_HEADER` is re-emitted on
every write, with or without an instance under it, so the file explains what it is, says to
`curl /` for the API, and tells a reader what an empty list means — at the one moment there
is no server left to ask. `instances = []` in JSON conveyed none of that. The header is
prepended as text rather than kept as document decor precisely so an old file picks up the
current wording instead of preserving its own.

Entries are deserialised **one table at a time**, not as a `Vec<Instance>` over the
document. The file is shared, so a single unreadable entry must cost that entry and no
other; taking the array in one go would let one bad field deregister every live browser.

Three more details are load-bearing. The file is a **read-modify-write under a lock file**,
because two Chrome profiles restoring windows really do race; a lock older than 5s belonged
to a killed process and is broken rather than waited on. The write goes through a temp file
and `rename`, so a reader never sees half a document. And a stale entry is detected by
issuing a real `GET /health` and looking for `"espai":true` — **a bare TCP connect is not
enough**, because the OS recycles ports and an unrelated process would pass it.

`/health` is answered by the host alone and never forwarded to the browser. That is
deliberate: it reports "this process still holds this port", which stays true and useful
even when the browser side is wedged. Routing it through the browser would make every
instance collect every other during a hang.

`kill -9` leaves an entry behind, which no amount of shutdown code can prevent. The GC is
the answer to that, not a redundant safety net.

## The 404 is the documentation

Every unknown path returns 404 with `API_GUIDE` — the same markdown `GET /` serves. An
agent that guesses wrong is handed the entire API in the error body and gets it right on
the next call, without a human ever writing integration notes.

This only works if the guide stays complete and honest: **when you add or change an
endpoint, change `API_GUIDE` in the same commit.** A guide that describes an API that no
longer exists is worse than none, because the agent trusts it. Keep it markdown, not JSON —
it is read straight out of `curl` output, and an escaped document full of `\n` is hostile
to that.

## Web pages are shut out. Local processes are not

There are two attackers and only one of them is answered.

**A web page in the user's own browser** was the live risk, and `serve` now refuses it up
front. Any page can POST to a loopback port, and a CORS *simple request* — a POST whose
content type is `text/plain` — goes out without the preflight the browser would have
blocked. The reply is unreadable to the page, which does not matter: a blind click or
keystroke lands anyway, in whatever session the user is logged into, and finding the port
is a sweep of 64k `no-cors` fetches. Three checks, in that order:

- any request carrying an **`Origin`** header is 403. A browser always sends one on a
  cross-origin POST; curl never sends one at all. This is the whole defence — the rest is
  belt to its brace.
- any request whose **`Host`** names something other than `127.0.0.1` or `localhost` is
  403, which is the same check seen from the other side: a domain rebound to this address
  reaches the socket still calling itself by name. The **port is deliberately not
  compared**, because the registry liveness probe speaks HTTP/1.0 and sends a bare
  `Host: 127.0.0.1`. Comparing it would make every instance garbage-collect every other.
- a **POST** that is not `application/json` is 415, including one with no body. A form, an
  `<img>` and a `sendBeacon` cannot produce that content type, so this closes the one
  request shape a page may send unpreflighted even if the `Origin` arm were ever bypassed.

These checks run **before the body is read**, so a refused request has no effect at all.
Keep them there.

**Another process at the same uid** is not answered and cannot be, by anything in this
file. It reads the port out of `~/espai.toml` like any agent does. A token stored
where the agent can read it is readable by that process too; this is an OS boundary, not an
HTTP one. **So do not describe this API as authenticated.** Nothing here knows *who* is
calling — only that it is not a web page.

Do not quietly add auth, and do not quietly pretend it exists.

## Titles come from the DOM on purpose

`chrome.tabs.query` already carries a title, and `/tabs` uses it. `/tabs/{id}/title`
deliberately does not: it round-trips through the content script and reads `document.title`
in the page. That path — HTTP, stdio, service worker, content script, DOM — is the thing
espai exists to provide, and it is the only route that proves all four hops still work.
Do not "optimise" it into a `chrome.tabs` lookup.

## Use `for` loops, not iterators

Prefer explicit `for` loops over `map`, `filter`, `fold`, `collect`, and similar chaining,
so control flow stays obvious.

## Unwrap `Option` and `Result` explicitly

Avoid `map`, `and`, `and_then`, `or`, `or_else`. Use `match` or `if let`. The terminal
helpers `unwrap_or`, `unwrap_or_else`, `unwrap_or_default`, and `map_err` are fine — the
rule is about not *chaining* logic through combinators.

## Inline code unless reuse is certain

Do not extract a function unless you are certain it is used more than once. Default to
inlining at the call site.

Most of the functions beside `main` clear that bar on callers alone — `respond`,
`copy_query`, `ask_browser`, `read_events` and `default_timeout` each have three or more.
The five that do not are the exceptions worth knowing, because each is extracted for a
reason that is not reuse:

- `serve` is the body of a spawned thread, which has to be *something*.
- `parse_range` and `merge_payload` are the two pieces of pure logic with unit tests, and
  a test is a second caller in every sense that matters here.
- `query_params` and `base64_decode` are written out rather than pulled in from a crate,
  and keeping each whole in one place is what makes that trade reviewable.

"It reads better over there" is not on that list. If a new extraction does not match one
of these shapes, inline it.

## Tests run without a browser

**`make all` before every commit.** It is `check-style`, `clippy`, `test`, `e2e`, then the
two extension builds, cheapest first. `clippy` is part of the gate rather than a target you
remember to run: the style rules above are about not chaining logic through combinators,
and clippy is the only thing that will notice when a change drifts back toward them.

`make e2e` runs the real binary with [`tests/host.py`](tests/host.py) impersonating Chrome
on the other end of the pipe, and [`tests/registry.py`](tests/registry.py) racing two
instances and killing one. Between them they cover everything that does not need a live
page: routing, bodies, per-request timeouts, that a hung call blocks nothing, the event
buffer, the long-poll, the liveness GC, and every request a web page could aim at the port.

That last group is the one to be careful with. Each check there stands for a request a page
in the user's own browser can really make, and a passing test is the only thing saying that
request still cannot click. When you add an endpoint, it inherits the guard automatically —
but if you ever touch the guard itself, the HTTP/1.0 health probe case is the one that
breaks quietly, and it takes the registry down with it rather than the API.

Both redirect `HOME` to a scratch directory. That is not tidiness — without it a test run
rewrites `~/espai.toml` out from under a browser that is using espai right now, and
leaves its own dead entries behind. Keep it that way when adding a script here.

Anything involving trusted input, the snapshot walker or the network capture cannot be
tested this way and has to be checked against a real browser.

## Failures are reported, not swallowed

A browser-side failure comes back as `{ok: false, error}` and becomes a 502 with the
browser's own words — "Receiving end does not exist" tells an agent exactly what is wrong
with a `chrome://` tab. A timeout is a 504. Registry failures are the exception: they are
logged and execution continues, because a host that cannot register is merely invisible,
which beats no browser eye at all.

## Releases

Tagging `v<version>` runs [`release.yml`](.github/workflows/release.yml), which verifies the
tag against `make version`, then calls `make dist TARGET=<triple>` on four runners and
uploads what it finds. The workflow does no packaging of its own — that belongs in the
Makefile, so a release can be reproduced by hand and so the two cannot drift.

**The public key is committed; `key.pem` is not, and must never be.** Chrome derives an
extension's id from the public half alone, which is the only reason a CI machine can build an
extension with espai's id at all. The private key signs `.crx` files: publishing it would let
anyone build an extension claiming this id, and the host manifest names exactly that id in
`allowed_origins`, with `chrome.debugger` behind it. Nothing in the build reads `key.pem`
anymore, and nothing should start.

**The host manifest cannot be a build artifact.** It names the binary by absolute path, and no
build knows where a user will unpack a tarball. `dist` therefore ships
`com.espai.host.json.in` with an `__ESPAI_BIN__` placeholder, carrying the one thing only the
build knows — the extension id — so [`install.sh`](install.sh) substitutes a path and never
holds a copy of the id.

`dist` is deliberately separate from `release` rather than built on it: `release` is the local
dev flow and stays host-only, so `make all` never cross-compiles.

---
> Source: [pouriya/espai](https://github.com/pouriya/espai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
