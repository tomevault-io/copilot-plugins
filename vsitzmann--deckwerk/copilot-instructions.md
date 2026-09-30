## deckwerk

> **Every slide is a 1920×1080 web page, and you are its front-end engineer.**

# Working on this deck

**Every slide is a 1920×1080 web page, and you are its front-end engineer.**
Author slides the way you would build a polished landing-page hero: semantic
HTML, flexbox and grid, whitespace doing the work. A real browser lays your
markup out and the editor bakes the result into the presentation. There is no
special slide language to learn — CSS is the slide language.

This folder is that deck:

	deck.json   compiled output — never edit or imitate it
	theme.css   the design system: typography, colour, the role-* classes
	edit/       your HTML files, watched by the editor
	assets/     media, referenced as assets/…

Do not read `deck.json` and do not compute pixel geometry — writing CSS and
letting the browser measure is the entire point of this workflow.

## Start here: comments are your task list

Humans leave instructions *inside the deck* as comments, attached to slides
and to individual objects. Begin every work session with:

    slide-agent comments .              # every comment, with its slide number
    slide-agent comments . --unresolved # just the open ones

Act on them, then mark each one done —
`slide-agent comments . --resolve <commentId>` — and reply when useful:
`slide-agent comments . --add "done, see slide 12" --slide <slideId>`.
Never delete a human's comment.

## If you were given a URL instead of this folder

A `http://…:58xx/?deck=…` URL is a **live collaboration session** — prefer it
over this file-based loop when you have a browser. Everything is discoverable
from the server:

- `GET /api/brief` — the full onboarding for the live workflow.
- `GET /api/comments?deck=<id>` — the same comment list, over plain HTTP.
- On the client page, `window.agent` (seeComments, commit, uploadAsset, …)
  and `window.store` drive the deck directly; edits sync live to everyone.

Do not run this folder's `apply` loop *and* drive the live session at the
same time — pick one. (An offline `apply` while a server hosts the deck is
picked up by its watcher, but it is a whole-file write: last resort, not the
default.)

## Design bar

These slides go on a projector next to professionally made ones. You are an
expert slide designer; hold the standard you would hold for a client's
marketing page:

- One idea per slide: a strong title, a few supporting elements, and room to
  breathe. Generous margins (~120px sides), aligned edges, consistent spacing
  — build with `display:flex`/`grid` and `gap`, not pixel nudging.
- Read `theme.css` before writing anything and compose with its `role-*`
  classes so your slides look native to this deck, not pasted in.
- Media large and deliberate: a result video is the hero of its slide, not a
  thumbnail in a corner; captions under figures, credits small.
- **Machine markup is not your example.** Exported/imported slides are baked
  `position:absolute` output. Never imitate that style for new content —
  write the nested, semantic markup you would write for the web, and check
  your work by rendering a PNG.
- **Critique before you call any slide done — not optional.** Render a PNG
  and review it as a stranger's work: write down at least three concrete
  deficiencies (composition, dead space, alignment, hierarchy, crowding),
  fix the ones that matter, render again. You have just built the slide,
  which is exactly when your judgment is most generous. "Renders without
  errors" is the floor; a slide is finished when you would sign it as a
  designer. Match the deck's *conventions* (fonts, colors, backgrounds),
  but do not treat its existing slides as the quality bar — many were made
  in a hurry.
- **theme.css is the stylesheet — put your classes there.** It is yours to
  edit and the editor hot-reloads it. Layout from any CSS bakes correctly, and
  a container's paint (background, border, radius) survives however it was
  styled — but text colour and fonts from a `<style>` block inside an edit
  file will NOT follow into the deck. Reusable styles belong in theme.css;
  inline styles are for one-offs.

## The loop

    slide-agent context                                      # outline + slideCount
    slide-agent inspect . --html --slide <id> > edit/work.html
    # edit edit/work.html and save it

Open that file in a browser: it *is* the slide, full size, with this deck's
theme and assets. Edit it like a web page — flexbox, grid, semantic HTML — and
the browser computes the geometry. While the editor is open, saving the file
updates exactly those slides about a second later, as one undoable change.
With the editor closed there is no watcher, so apply the same file explicitly:

    slide-agent apply . --html edit/work.html
    slide-agent validate

**To append new slides you do not need to export anything first.** Write a new
file in `edit/` containing only new `<section class="slide">`s (no
`data-slide-id`): it appends at the end — on save with the editor open, or
with `slide-agent apply . --html edit/new.html` (`--after <slideId>` to
place it elsewhere). Export a range only when you want to *change* it.

## Rules that keep the loop safe

- **`context` first, in full.** It prints `slideCount` up front; do not
  `head`-truncate the outline and mistake the visible part for the deck.
- **Sections are slides.** Adding, removing or reordering `<section>`s in the
  file adds, removes or reorders exactly those slides. Removing a section from
  a file that exported it **deletes** that slide.
- **After every successful sync, your file is rewritten in place**: each new
  `<section>` gets its assigned `data-slide-id` stamped in. That is what
  makes saving or applying the same file again *replace* those slides instead
  of duplicating them. Re-read the file after a sync rather than keeping an
  old copy.
- **If `apply` times out, do not apply again.** The editor may still land the
  change; run `slide-agent context` and look at the outline first.
- **Flags are strict.** `--slide <id>` (repeatable, or comma-separated),
  `--selected`, `--all`. Misspelt flags and unknown slide ids are errors,
  never silent fallbacks.
- **Keep the file in `edit/`** — its `<base>` is what makes `assets/…`
  and the theme resolve — and write it atomically if you can.

## Media, maths, style

- **Video:** a bare tag is the whole story:
  `<video src="assets/clip.mp4"></video>` (autoplay, loop, muted are the
  defaults). Import foreign codecs first: `slide-agent asset import . file.mp4`
  — **its JSON output tells you the final `src`** (files are content-hashed
  on the way in), so read it rather than guessing or listing `assets/`.
- **Images:** `<img src="assets/figure.png">`, laid out with normal CSS.
  For a circular headshot, `border-radius: 50%` on the image (or its
  container) survives the compile and clips correctly.
- **Maths:** `$…$` and `$$…$$` in any text, rendered by built-in KaTeX.
  Never build equations out of positioned text.
- **Text:** pick a theme role — `role-title`, `role-heading`, `role-body`,
  `role-caption` — instead of inline font sizes.
- Wrapper `<div>`s are layout: they dissolve on compile and their children
  become the slide objects. Do not hand-copy `class="element …"` wrappers
  from exports around your own markup; plain semantic HTML is the input.

Exported files are large — the full KaTeX font set rides inline so they render
anywhere — so never `cat` one; read from `<body>` on
(`sed -n '/<body/,$p'`). Your own markup is all that lives there.

Verify with `slide-agent validate . --slide <your-ids>`: it checks the deck's
structure and reports elements extending past the canvas (`overflows`) for
the slides you name — deliberate bleeds are fine, but a text box past the
bottom edge is a mistake the projector will show. Unscoped, it reports the
whole deck, pre-existing bleeds included.
When looks matter, render a PNG: `slide-agent render . --slide <id> --output /tmp/shots`.
Add `--contact-sheet` for one tiled overview — the cheap way to survey a whole
deck before deciding what to change. To *show* the deck to a human, run
`slide-agent preview . --open` in the background: it serves the deck through
the real player on localhost and prints the URL.

`slide-agent capabilities` documents builds, Morph, crops, video trim
and KaTeX, each with working markup. `slide-agent docs` has the full guide.
The older JSON transaction API is still there for tooling with no browser, but
it is not how slides are authored any more.

If `slide-agent` is not on your PATH, it is at:

    /Users/vincentsitzmann/Coding/slide-editor/bin/slide-agent

This file was generated when the deck was opened. It is yours now — add notes
about the talk to it if you like; the editor never rewrites it.

---
> Source: [vsitzmann/deckwerk](https://github.com/vsitzmann/deckwerk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
