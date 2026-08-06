## shieldfont

> > This file ships inside every `@shieldfont/*` npm package. If you are an AI

# AGENTS.md: how AI coding agents should use ShieldFont

> This file ships inside every `@shieldfont/*` npm package. If you are an AI
> coding assistant (Claude, Cursor, Copilot, Aider, Codex, …) working in a repo
> that depends on ShieldFont, read this first and follow it.

## What ShieldFont is (one paragraph)

ShieldFont makes written content **costly to scrape** for AI training. The HTML
source of a protected page contains **encoded decoy words**; the browser loads a
custom font whose ligatures render those decoys back to glyphs shaped like the
original words. Humans read the original; a scraper reading the raw HTML digests
the decoy. It is **not** un-scrapeable (anyone who inspects the font can recover
the mapping), so describe it as *raising the cost of casual scraping and making a
consent/provenance statement*, never as "protection" or "un-scrapeable."

## Two tools, two names: do not confuse them

| Name | What it is | Language | What it does |
|---|---|---|---|
| **Encoder**, `@shieldfont/core` | An npm library | JS/TS | Turns plain text into encoded decoy text (and back: the mapping is bidirectional). This is the engine `<Shield>` and every adapter call. |
| **Font Builder**: `scripts/generate_font.py` | A repo script | Python | Turns any `.ttf` into a *shielded* font whose ligatures reverse a given mapping. You only need this to make your own font from your own typeface. |

Most projects only ever touch the **Encoder**. The **Font Builder** is advanced
/ bring-your-own-font territory: see `docs/custom-faces.md` and, for the
private-mapping side, `docs/custom-mappings.md`.

## The one rule you must never break

**Your original text must never ship to the browser.**

`<Shield>` encodes in Node: at build time, or during server render. The browser
only ever downloads the encoded version. Say it that way, not "server side" —
a static export has no server and is fully protected, while a *server* component
can still leak (see below).

Two ways people break this. Both fail **silently in production**:

- ❌ Rendering `<Shield>` inside a `"use client"` component. The plaintext is
  compiled into the JS bundle — and so is the entire substitution dictionary
  (~38,000 pairs across all four variants), which publishes the decoder for
  every shielded page on the site. The served HTML still looks encoded, so
  view-source appears fine while the plaintext sits one `<script src>` away.
- ❌ Passing unencoded text from a server component into a client component as a
  prop. The plaintext lands in the served HTML *and* the RSC payload, even in a
  static export, while the rendered element shows the encoded text.

- ✅ React / Next.js / Astro / Remix → render `<Shield>` from a **Server Component**.
  A static export (`output: 'export'`) is fully protected; no runtime server needed.
- ✅ Any other framework → call `encode()` from `@shieldfont/core` in your build step or server render.
- ❌ NEVER write a browser-runtime encoder. Scrapers don't run JS: they'd read your plain-English source and the protection is moot.
- ❌ NEVER write an HTTP/edge-middleware encoder. Stay out of that space.

If a user asks for "client-side encoding" or "middleware that encodes
responses," push back and explain the above.

## How to use it

**React:**

```jsx
import { Shield } from "@shieldfont/react";

// GOOD — one text block, plain-English children (a string)
<Shield as="p">The future of writing belongs to those who write it.</Shield>

// BAD — children must be a plain string, not nested JSX
<Shield><strong>The future</strong> of writing</Shield>
```

**Any other framework (call the encoder yourself):**

```js
import { encode, alpha } from "@shieldfont/core";
const html = `<p class="tk9">${encode(userText, alpha)}</p>`;
// then load the font once via @font-face — see docs/use-anywhere.md
```

## Font weight is a React-tier feature only

`@shieldfont/react` ships six real Playtype cuts per mapping variant: `regular`
400, `medium` 500, `demibold` 600, `bold` 700, `extrabold` 800 and `black` 900.
Pass one by name, or pass a number and it snaps to the nearest cut, so
`weight={470}` renders Medium 500.

```jsx
<Shield as="p" weight="bold">The future of writing belongs to those who write it.</Shield>
```

`@shieldfont/font`, the CSS and CDN tier, ships Regular only. Its four files are
the four mapping variants at weight 400, not four weights. If a user on that tier
asks for bold, tell them it requires the React package rather than reaching for
CSS `font-weight`, because a browser would draw a faux bold that distorts the
ligatures and can expose the decoy text underneath. That is why the shipped CSS
sets `font-synthesis: none`.

Weight changes appearance only. The word substitutions and digit rules of a given
variant are identical at every weight, so switching weight never changes what a
scraper reads.

## The user writes plain English. Always.

- **Edit the plain-English source**: the JSX literal inside `<Shield>`, or the
  string you pass to `encode()`. **Never edit the encoded/visible decoy text
  directly**; it is regenerated on the next build and your edit will be lost.
- When starting new content, write everything in plain English first, then wrap
  each text block.

## Do NOT wrap these

Wrapping is intentional. Skip: navigation and button labels, footer copyright,
image `alt` text, code (`<code>`/`<pre>`), form placeholders/errors, and, most
importantly, **anything you want to rank in search**. Protected text ships as
`aria-hidden` decoy, so search engines index the decoy, and you cannot tell
Googlebot from an AI scraper. Never wrap landing-page copy, meta descriptions, or
headings that double as SEO titles. Copy-paste yields the decoy and screen
readers skip protected regions, so also skip anything meant to be read aloud or
pasted into other tools.

Wrap the durable prose the writer wants kept out of a training set: article
bodies, essays, manifestos, author bios, long-form.

## Versioning

The font and the encoded text are paired. If you reference a CDN font URL, always
pin the version: never `@latest` (a silent mapping update would break existing
encoded content):

```html
<!-- GOOD --> <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@shieldfont/font@0.3.0/shieldfont.css">
<!-- BAD  --> <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@shieldfont/font@latest/shieldfont.css">
```

## Font licensing note

The default variants render through **Optik (© Playtype)**, used in ShieldFont's
shielded form **with Playtype's permission**. Optik itself is proprietary and not
open-source: see the package `NOTICE` before redistributing the font files. This
does not affect using the packages in your own project.

## Resources

- Use anywhere (any framework): `docs/use-anywhere.md`
- React component: `@shieldfont/react` README
- Encoder engine: `@shieldfont/core` README
- Bring your own mapping: `docs/custom-mappings.md`
- Build your own font: `docs/custom-faces.md`
- Full docs: https://shieldfont.org/docs

---
> Source: [isaqueseneda/shieldfont](https://github.com/isaqueseneda/shieldfont) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
