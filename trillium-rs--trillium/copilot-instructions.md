## trillium

> Here are some conventions in this document.

# About this document

Here are some conventions in this document.

* `use` declarations will only be listed once on the first usage of a
  given type in order to keep code samples concise
* In-line code looks like this: `|conn: Conn| async move { conn }` and
  will generally not involve fully qualified paths
* Footnotes are represented like this[^1]
* Informational asides look like this:
  > ℹ️ Fun fact: Facts are fun
* Advanced asides look like this
  > 🧑‍🎓 The handler trait provides several other lifecycle hooks for
  > library authors
* Comparisons with Tide
  > 🌊 Tide endpoints look like `|_req: Request<_>| async {
  > Response::new(200) }` whereas Trillium handlers look like `|conn:
  > Conn| async move { conn.with_status(200) }`
* Comparisons with Plug:
  > 🔌 Halting a plug looks like `conn |> halt` (elixir), and the
  > equivalent in trillium is returning `conn.halt()`

[^1]: Footnotes can always be skipped

## Who is this document for?

This document expects some familiarity with async rust. We intend to
offer a beginner level document at some point, but for now we
recommend looking at [the rust book](https://doc.rust-lang.org/book/)
and [the async book](https://rust-lang.github.io/async-book/).

We also assume familiarity with web development in general, including
concepts and patterns in http servers and frameworks.

In particular, we offer comparisons to rust's
[tide](https://github.com/http-rs/tide) and elixir's
[phoenix](https://www.phoenixframework.org/) /
[plug](https://hexdocs.pm/plug/readme.html), as they serve as the
primary inspirations for trillium.

---
> Source: [trillium-rs/trillium](https://github.com/trillium-rs/trillium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
