## pops

> A.  Defines the layout rules for files in `paul/`.

# Outline Format Rules

1.  Purpose
    A.  Defines the layout rules for files in `paul/`.
    B.  Each line shows only its local marker; uniqueness
        comes from the marker path.

2.  File type
    A.  Plain text (`.txt`), no Markdown.
    B.  Use quotes only for literal tokens (e.g.,
        `@@thought:`).

3.  Style (brevity)
    A.  Keep bullet text concise and brief.
    B.  Prefer short, declarative sentences.

4.  Outline levels and markers
    A.  L1: numeric; pad 1–9 with `1.  `; 10+ with `10. `.
    B.  L2: uppercase (`A.  `).
    C.  L3: numeric; pad 1–9 with `1.  `; 10+ with `10. `.
    D.  L4: lowercase (`a.  `).
    E.  Deeper: document before using.

5.  Indentation and alignment
    A.  4 spaces per level.
    B.  Child marker starts under the first character of the
        parent's text (works for `9.  ` and `10. ` alike).
    C.  No drift: deeper markers start exactly at the parent
        text-start column (never past the wrap column).

6.  Text wrapping (reflow)
    A.  Wrap at ~60 characters.
    B.  Continuation lines start under the first character
        of the bullet text.
    C.  Reflow requirement: after any edit, reflow the whole
        file to the ~60 target (don't leave early manual
        wraps).

7.  Spacing
    A.  Blank line between Level 1 sections.

8.  Implied IDs (how uniqueness works)
    A.  Unique ID = marker path (example: `3C2.`).

9.  Minimal example (layout)
    A.  Example only (markers shown literally):
        1.  `1.  Title`
        2.  `    A.  Child`
        3.  `        1.  Grandchild`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mPulseMedia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
