## mdrise

> >-


# Markdown → Styled HTML

Convert a Markdown source file into a single self-contained HTML file that renders the *same content* as a finished web page — every word preserved, presented with deliberate typography, layout, and visual identity.

## The two invariants

These rules together define the skill. Everything else serves them.

1. **Text fidelity** — every word, sentence, list item, code snippet, link, table cell, and blockquote in the source Markdown appears unchanged in the HTML. No paraphrasing, no shortening, no "improving" the prose, no translation, no fixing what looks like a typo. The author's voice, including its imperfections, is sacred.

2. **Visual elevation** — the HTML is not a vanilla `<h1>` + `<p>` render. It uses considered typography, a deliberate color system, layout structure (sections, cards, grids where appropriate), and tasteful visual enhancements chosen to match the content's tone.

If these conflict, fidelity wins. Style supports the content; it never overrides it.

A useful self-check at the end: if someone copy-pasted the rendered text out of the HTML and diffed it against the source MD's text, the diff should be empty (modulo whitespace).

## Workflow

### 1. Read the whole source first

Read the entire MD file before writing any HTML. Picking a style without the full picture leads to a tone mismatch halfway down the page.

### 2. Diagnose the content's tone

Silently answer (don't write this in the output):
- **What is this?** Technical README, personal essay, product launch, course notes, recipe, novel chapter, internal memo, …
- **Who's the likely reader?** Developer, layperson, executive, friend, classmate, …
- **What emotional register?** Precise/technical, warm/personal, urgent/promotional, contemplative/literary, playful, somber, …

### 3. Commit to a single aesthetic direction

**Core principle — identify the medium the content imitates, and let that drive the aesthetic.**

- A **literary essay** imitates a **book page** → paper texture, serif body, ornamental dividers, narrow column.
- A **technical doc / status report** imitates a **documentation site** (Stripe Docs, Vercel Docs, Anthropic Docs) → screen-native, crisp hierarchy, clean canvas — *not* paper, *not* serif Latin body.
- A **product / startup launch** imitates a **modern product landing page** (Linear, Resend, Plain) → confident, designed, screen-native, bold typographic choices.
- A **recipe** imitates a **magazine spread**. A **course note** imitates a **notebook**. A **poem** imitates a **printed page**. And so on.

Picking the right medium first prevents the most common failure mode: **smuggling book-page cues into non-literary content.** Cream paper backgrounds, aged-paper noise textures, serif Latin body, ❖/※/asterism dividers, drop caps, narrow paperback margins — these are *reserved* for literary content. A technical status report on cream paper with a serif Latin body and ❖ ornaments is a category error: it doesn't feel "carefully crafted," it feels confused. A SaaS product launch with editorial paper textures is the same mistake.

Don't blend three styles into mush. Pick one direction and execute with precision:

| Content type | Direction (imitate) | Avoid |
|---|---|---|
| Open-source dev tool README | Documentation site (Stripe / Vercel / Anthropic docs): clean white or true-dark canvas, crisp sans body for Latin, mono for code, structured hierarchy, plenty of negative space | Cream paper, serif Latin body, aged-paper noise, ❖ dividers, drop caps — anything whispering "book" |
| Product or startup launch README | Modern product landing page (Linear / Resend / Plain): bold designed surface, screen-native, often dark or stark-minimal, distinctive display type, asymmetric layouts | Editorial / book-page hybrids, paperback paper textures, "warm paper + dark code" combos, anything that reads as printed |
| Technical project status / audit report | Engineering documentation surface: strong hierarchy, mono numerals, tabular data crisp. A warm off-white background is fine, but the surface stays digital — no paper texture, no aged-paper noise. Latin body should be sans, mono for data. Chinese body can be Noto Serif SC (that's a normal documentation choice for CJK, not a "book" signal on its own) | Aged-paper noise, ❖ ornaments, drop caps, Latin serif body, calligraphic display fonts on every heading — the *accumulation* of these reads as "book", which this isn't |
| **Personal essay / blog post** | **Book-page editorial**: serif body, narrow column (580–680px), generous line-height (~1.9), considered margins, drop caps or ornamental section-opener, classical dividers (❖, ※, asterism, or hairline rules). Should feel like a printed page from a literary magazine or quality paperback — page-like, not webpage-like. A warm cream / rice-paper canvas with subtle radial gradient is often the right move | Webpage chrome, hero CTAs, feature grids, modern landing tropes |
| Recipe / how-to guide | Friendly magazine spread: warm palette, numbered steps as cards, ingredient call-out boxes | Severe / brutalist coldness |
| Course / study notes | Notebook: clear hierarchy, highlight boxes for key concepts, margin annotation feel | — |
| **Poetry / literary writing** | **Page-like canvas**: warm paper background, large negative space, refined serif (Noto Serif SC / Source Han Serif for Chinese), single restrained accent. Vertical rhythm matters more than visual flair. A handwritten or brush-style accent font (Ma Shan Zheng, ZCOOL XiaoWei) on the *title only* is fine — never on body | Anything digital-screen-native, card grids, landing-page tropes |
| Conference talk / pitch | Slide-like sections: bold display type, full-bleed color blocks, big numbers | Paper/book aesthetics |

**The book-page reservation is the most important boundary in this table.** When the content type doesn't appear above, ask: "what real-world artifact does this most resemble?" and pick the matching aesthetic — but the answer is almost never "a book" unless the content is genuinely literary.

### 4. Pick concrete style tokens before coding

Decide these explicitly first — don't drift into defaults mid-write:

- **Font pair** — one display font for headings + one body font. AVOID generic AI defaults (Inter, Roboto, Arial, system-ui, default sans). Reach for distinctive Google Fonts. Match the family to the medium you're imitating:
  - **For technical docs / status reports / product launches** (screen-native content): Latin body should be a distinctive **sans** — Bricolage Grotesque, Manrope, Outfit, Sora, Geist, Space Grotesque, IBM Plex Sans, Inter Tight (NOT plain Inter). Use a **mono** for code / numerals / data: JetBrains Mono, Geist Mono, Space Mono, IBM Plex Mono. Display can be a distinctive serif (Fraunces, Instrument Serif) used as an accent — but body stays sans.
  - **For literary essays / poetry** (book-page content): Latin body should be a quality **serif** — Newsreader, Crimson Pro, Libre Caslon, EB Garamond, Spectral, Lora, IBM Plex Serif. Display can be more expressive (Fraunces, DM Serif Display, Playfair Display).
  - **For Chinese content** in any medium: Noto Serif SC and Source Han Serif are appropriate for body text across both technical and literary use (CJK serif is not a "book-only" signal the way Latin serif is). LXGW WenKai (霞鹜文楷) is warmer and works well for literary content. Reserve **brush-style fonts** (Ma Shan Zheng, ZCOOL XiaoWei) for literary titles or signature lines only — never on technical content, never on body.
- **Color palette** — 1 background, 1 body text, 1 accent, 1 muted/secondary. Skip the purple-gradient-on-white reflex. Consider warm off-whites, dark earth tones, single bold accent against neutral, paper-and-ink monochrome with one color of restraint.
- **Layout width** — narrow column (640–720px) for prose, wider with sidebar for technical docs, full-bleed sections for landing-page energy.

### 5. Emit the HTML

A single self-contained file with:
- `<!doctype html>`, `<meta viewport>`, `<meta charset="utf-8">`, `<title>` derived from the first H1 in the MD
- **Idempotency metadata** in `<head>` so future invocations can detect whether re-rendering is needed:
  - `<meta name="mdrise-source-hash" content="sha256:<hex>">` — SHA-256 of the source `.md` file bytes (UTF-8)
  - `<meta name="mdrise-generated-at" content="<ISO-8601 UTC timestamp>">`
  - `<meta name="mdrise-source-path" content="<basename of source .md>">`
- Fonts loaded from Google Fonts via `<link>` in `<head>` (the **only** allowed external resource, besides Google Fonts)
- All other CSS inlined in a single `<style>` block
- All Markdown faithfully rendered: headings, paragraphs, ordered/unordered lists, code blocks (with tasteful syntax-aware coloring if you can manage it without bloat), tables, blockquotes, images, links, inline emphasis
- Responsive: must look correct at ~375px (mobile) and ~1280px (desktop). Test by mentally resizing
- Inline `<script>` is allowed for: the floating table-of-contents (collapse/expand + scroll-spy), copy-to-clipboard on code blocks, language toggles for bilingual documents. Avoid JS for cosmetic effects (carousels, parallax) on static text

### 6. Save

Save the file alongside the source MD with the same basename and `.html` extension, unless the user specifies otherwise. For `README.md`, output `README.html` in the same directory.

## What "enhancement" allows (and forbids)

This skill operates in **enhancement mode**, not strict mode. Visual additions are welcome — but only those that *amplify* the content, never replace or paraphrase it.

**Allowed enhancements:**
- Heading icons (e.g., a 📦 next to "Installation", a custom SVG next to "Features") matched to content
- Anchor links and a floating table-of-contents for long documents
- Rendering a list as a card grid when items have parallel structure (e.g., "Features", "Why us", "FAQ")
- Callout boxes for blockquotes, "Note:", "Warning:", "Tip:" patterns
- Decorative dividers, drop caps, badge-style rendering for inline `code`
- Subtle background textures, gradients, accent flourishes — used with restraint
- Generated decorative SVG (geometric shapes, abstract hero graphics) that don't claim to be content
- Tasteful hover states and micro-interactions if they reward exploration

**Forbidden:**
- Rewriting any prose, even slightly. Not even "fixing" what looks like a typo
- Adding new sentences, sections, captions, or alt text that wasn't in the source. Alt text may *quote* surrounding source text but never invent
- Reordering content
- Removing content because it "doesn't fit the design" — adapt the design instead
- Generating placeholder text (lorem ipsum, "Your product here")
- Inserting CTAs, social-share buttons, newsletter signups, or footers the source doesn't have

## Floating side table-of-contents (default — not optional)

A floating, collapsible side ToC is a **default feature**, not a "long documents only" enhancement. Markdown's heading structure is one of its most valuable affordances — the rendered page must let readers scan structure and jump anywhere instantly, from any scroll position. The user has explicitly asked for this.

**Required behavior:**

1. **Auto-generate** from `<h2>` and `<h3>` headings in the rendered HTML (give each a stable `id`).
2. **Fixed-position panel** on the left or right side of the viewport — pick whichever side fits the layout. The panel can be visually quiet (a hairline rail with small-caps section names) or richer (a full pane with numbered items) — that's a style choice that must match the page's aesthetic direction.
3. **Persistent toggle icon** — a small button (`☰`, `⋮`, a chevron, or a custom glyph chosen to fit the aesthetic) stays in a *fixed position on screen at every scroll position*, even when the ToC is collapsed. Clicking it expands/collapses the panel. The toggle must never disappear as the user scrolls — that's the whole point.
4. **Active-section highlighting** — as the user scrolls, the entry corresponding to the current section is visually marked (e.g., filled background, bold, accent-color rule). Use an inlined `IntersectionObserver` for scroll-spy.
5. **Mobile (<768px)**: default the panel to collapsed; the toggle stays accessible as a floating button (top-right or bottom-right). Tapping it slides the panel in as an overlay.
6. **Default expanded state**:
   - Technical / reference / docs content → start **expanded** (readers expect navigation).
   - Literary / essay / poetry → start **collapsed** (don't disrupt the reading surface), toggle still visible.

**Skip the ToC only when:**
- The document has fewer than 2 H2 headings (no real structure to navigate).
- The content is a single-block piece (single-paragraph note, very short announcement).

**Styling:** the ToC and its toggle are part of the design, not bolted on. Match its colors, typography, and motion to the page identity. A page-like book essay might use italic serif section names in the ToC; a developer doc might use mono numerals; a landing page might use small-caps with the accent color marking the active section. Bad styling here ruins the rest of the page.

**Implementation sketch** (inline this kind of JS — keep it minimal):

```html
<aside id="toc" class="toc">
  <button class="toc-toggle" aria-expanded="false" aria-controls="toc-list">☰</button>
  <nav id="toc-list" class="toc-list" hidden>
    <a href="#section-1">Section 1</a>
    <a href="#section-2">Section 2</a>
    ...
  </nav>
</aside>
<script>
  const btn = document.querySelector('.toc-toggle');
  const list = document.getElementById('toc-list');
  btn.addEventListener('click', () => {
    const open = list.hasAttribute('hidden') ? false : true;
    if (open) { list.setAttribute('hidden',''); btn.setAttribute('aria-expanded','false'); }
    else { list.removeAttribute('hidden'); btn.setAttribute('aria-expanded','true'); }
  });
  const links = [...document.querySelectorAll('.toc-list a')];
  const map = new Map(links.map(a => [a.getAttribute('href').slice(1), a]));
  const io = new IntersectionObserver((entries) => {
    entries.forEach(e => {
      if (e.isIntersecting) {
        links.forEach(a => a.classList.remove('active'));
        map.get(e.target.id)?.classList.add('active');
      }
    });
  }, { rootMargin: '-20% 0px -70% 0px' });
  document.querySelectorAll('h2[id], h3[id]').forEach(h => io.observe(h));
</script>
```

The exact markup, classes, and CSS are up to you — but those four behaviors (auto-gen, fixed toggle, scroll-spy, collapse/expand) are non-negotiable.

## Multi-language and multi-script support

The skill must work for content in any language and script — not just Chinese and English. Most failures here aren't aesthetic; they're literal: text rendering as `□□□` tofu boxes because the loaded font doesn't cover the characters.

**Step 1 — Identify scripts in the source.** Scan the visible characters. The source may contain one script (e.g., a French essay in Latin) or several (e.g., a Hindi article quoting English code samples and Devanagari verse). Catalog all of them before picking fonts.

**Step 2 — Pick fonts that actually cover those scripts.** Don't fall back to system defaults. Reach for Google Fonts that have proper coverage for the scripts involved. If unsure, the **Noto family** is the safe universal default — `Noto Serif <Script>` / `Noto Sans <Script>` exists for almost every script, designed by Google specifically to give consistent coverage worldwide.

| Script | Body candidates | Display / accent candidates |
|---|---|---|
| Latin (English, French, German, Spanish, Italian, Portuguese, Polish, Vietnamese, Turkish, ...) | Newsreader, Spectral, Lora, IBM Plex Serif/Sans, Manrope, Outfit, Geist, Bricolage Grotesque | Fraunces, Instrument Serif, DM Serif Display, Playfair Display |
| Cyrillic (Russian, Ukrainian, Bulgarian, Serbian, ...) | Spectral, IBM Plex Serif/Sans, Manrope, Fraunces, EB Garamond (most modern Google Fonts now ship Cyrillic) | Same |
| Greek (modern Greek) | Spectral, IBM Plex Serif/Sans, Manrope, Fraunces | Same |
| Chinese (Simplified, Traditional) | Noto Serif SC / TC, Noto Sans SC / TC, Source Han Serif/Sans, LXGW WenKai | ZCOOL XiaoWei, Ma Shan Zheng (literary titles only) |
| Japanese | Noto Serif JP, Noto Sans JP, Shippori Mincho, Zen Old Mincho, Klee One | Yuji Syuku, RocknRoll One, Reggae One |
| Korean | Noto Serif KR, Noto Sans KR, Nanum Myeongjo, IBM Plex Sans KR, Gowun Batang | Black Han Sans, Jua, Single Day |
| Arabic (**RTL** — Arabic, Persian, Urdu) | Noto Naskh Arabic, Noto Sans Arabic, Amiri, Cairo, IBM Plex Sans Arabic | Reem Kufi, Aref Ruqaa, Lalezar |
| Hebrew (**RTL**) | Noto Serif Hebrew, Noto Sans Hebrew, Frank Ruhl Libre, David Libre, IBM Plex Sans Hebrew | Suez One, Heebo, Bellefair |
| Devanagari (Hindi, Marathi, Sanskrit) | Noto Serif Devanagari, Noto Sans Devanagari, Hind, Tiro Devanagari | Yatra One, Mukta Vaani, Baloo 2 |
| Bengali (Bangla, Assamese) | Noto Serif Bengali, Noto Sans Bengali, Hind Siliguri, Tiro Bangla | Mina, Galada, Baloo Da 2 |
| Tamil | Noto Serif Tamil, Noto Sans Tamil, Catamaran, Tiro Tamil | Pavanam, Hind Madurai, Anek Tamil |
| Telugu, Kannada, Malayalam, Gujarati, Punjabi (Gurmukhi), Oriya | Noto Serif / Sans for each script (Noto Serif Telugu, Noto Sans Kannada, etc.) | Noto Display variants, or script-specific picks like Anek Telugu, Baloo Tammudu |
| Thai | Noto Serif Thai, Noto Sans Thai, IBM Plex Sans Thai, Sarabun, Prompt | Mitr, Charm, Pridi |
| Lao | Noto Serif Lao, Noto Sans Lao, Phetsarath | — |
| Khmer | Noto Serif Khmer, Noto Sans Khmer, Battambang, Hanuman | Bayon, Moul |
| Myanmar (Burmese) | Noto Serif Myanmar, Noto Sans Myanmar, Padauk | — |
| Georgian | Noto Serif Georgian, Noto Sans Georgian, BPG Nino Mtavruli | — |
| Armenian | Noto Serif Armenian, Noto Sans Armenian, Mariupol | — |
| Ethiopian (Amharic, Tigrinya) | Noto Serif Ethiopic, Noto Sans Ethiopic, Abyssinica SIL | — |

**Step 3 — Set `<html lang="...">` correctly** using the source content's primary language tag: `en`, `zh-Hans` / `zh-Hant`, `ja`, `ko`, `ar`, `fa`, `he`, `hi`, `bn`, `ta`, `th`, `ru`, `vi`, `de`, etc. Get this right — it affects line breaking, hyphenation, accessibility tools, and how browsers fall back on glyphs.

**Step 4 — RTL scripts (Arabic, Hebrew, Persian, Urdu).** Add `dir="rtl"` to `<html>`. The entire visual layout mirrors: text flows right-to-left, the floating ToC sits on the **left** edge of the viewport (the "right" in reading order), padding and margin directions flip, list bullets and numbered markers appear on the right. Use logical CSS properties (`padding-inline-start`, `margin-inline-end`, `border-inline-start`) instead of `left`/`right` so the layout adapts automatically. **Keep code blocks, file paths, URLs, and embedded English brand names LTR** with an explicit `dir="ltr"` on those elements — RTL applied to them is wrong and looks broken.

**Step 5 — Mixed-script content.** Most real-world MD has a primary script plus inline accents (Chinese with English code; Hindi with Latin URLs; Arabic with English brand names). Build font stacks so each character finds a renderer:

```css
body {
  font-family: "Noto Serif Arabic", "Noto Serif", serif;  /* Arabic body with Latin fallback */
}
code, .ltr {
  font-family: "JetBrains Mono", monospace;
  direction: ltr;
  unicode-bidi: embed;
}
```

For pages with two heavy scripts (e.g., a Japanese article quoting Devanagari poetry), include both Noto fonts and let the cascade resolve per glyph.

**Step 6 — Verify glyph coverage before declaring done.** Mentally scan the rendered HTML for any character that might fall back to a system default — uncommon CJK characters, emoji, math symbols, ligatures, less-common diacritics. If the loaded fonts don't cover something, add the right font to the stack.

When in doubt about an unfamiliar language, **default to Noto**. The Noto family is Google's deliberate effort to cover every script with consistent metrics. It will never be the most distinctive choice, but it will never break.

## Anti-slop principles

The single biggest failure mode for AI-generated HTML is generic-ness. Watch for and resist:

- **Book-page cues on non-literary content.** Cream paper backgrounds, aged-paper noise textures, serif *Latin* body, ❖/※/asterism dividers, drop caps, paperback-margin column widths — these belong on literary essays and poetry. On technical reports or product launches they're a category error and make the page feel confused, no matter how nice in isolation. Audit your output: if it looks like a book and the content isn't one, restart.
- **Generic fonts.** Inter, Roboto, Arial, default `system-ui`, default sans — these scream "AI default." Pick something with character. Always.
- **Purple-gradient-on-white.** And mint-to-lavender hero gradients. And blue-to-purple anything. These are exhausted. Reach for unexpected color combinations grounded in the content's mood.
- **Centered-everything layouts** with identical rounded-corner cards stacked in threes. Asymmetry, deliberate alignment, and considered hierarchy beat uniform "card-card-card."
- **Filler decoration.** If a flourish doesn't earn its place by serving the content, remove it.
- **Mismatched effort.** A minimal essay needs restraint and precise typography, not maximalist animation. A bold landing page can take ambitious motion. Match implementation intensity to the aesthetic vision.

## Detection workflow — don't regenerate redundantly

This skill is designed to be **safely idempotent**: invoking it repeatedly on an unchanged source must not produce redundant rerenders. Before doing any conversion work, run the detection workflow below. This is what separates "smart maintenance" from "blanket trigger."

### Step 0 — Determine target file

- **README auto mode**: the user asked for a visual / web / styled rendering without naming a specific file, AND a `README.md` exists in the working directory. The target is `README.md` → `README.html`.
- **Explicit mode**: the user named a specific `.md` path and asked to render it. The target is that file → sibling `.html` with the same basename.
- If neither condition is met, do not proceed — ask the user which file they mean.

### Step 1 — Sync check against existing output

For the target's expected `.html` output path, ask three questions in order:

1. **Does the `.html` exist at all?**
   - No → **proceed silently to render.** Brand-new generation, no question.
2. **If yes, does it contain a `<meta name="mdrise-source-hash" content="…">` tag?**
   - No → **proceed to render** (the HTML was made by something else, can't tell if it's in sync — safer to regenerate).
3. **If yes, does the embedded hash match a fresh hash of the current source `.md`?**
   - **Hashes differ** → the source has been modified since the HTML was rendered. **Proceed silently to render.** (This is the typical "AI just edited README.md" case — exactly what should happen.)
   - **Hashes match** → the existing `.html` is already in sync with the current source. Behavior depends on mode:
     - **Auto / README mode** → **ASK the user** before regenerating. Phrase it like: *"`README.html` is already in sync with the current `README.md` (last rendered <timestamp from meta>). Regenerate anyway? — `yes` / `no` / `open the existing one`."* Only render on explicit yes.
     - **Explicit mode** (user asked specifically) → **proceed to render**, but mention in the response: *"Note: the existing `<path>.html` was already in sync; regenerating per your request."* The user asked, so honor it — just be transparent.

### Step 2 — Render (only if step 1 said proceed)

Follow the standard workflow (Read → Diagnose tone → Pick aesthetic → Emit HTML → Save). When writing the HTML, **embed the source-hash and a timestamp** in the `<head>`:

```html
<meta name="mdrise-source-hash" content="sha256:<hex-of-canonical-source-bytes>">
<meta name="mdrise-generated-at" content="2026-05-12T14:30:00Z">
<meta name="mdrise-source-path" content="README.md">
```

**Important — what to hash:** hash the **canonical source content**, which is the MD file with the auto-managed `<!-- mdrise:banner --> ... <!-- /mdrise:banner -->` block stripped out (see the "Bidirectional banner" section below). The banner is auto-injected by this skill on every successful render, so including it in the hash would cause spurious mismatches the next time we look at the file (it would always look "modified" because we just edited it). Strip the block (and any trailing blank line that immediately followed it) before hashing.

Use the bytes of the **stripped** `.md` content (UTF-8 encoded) for the hash. SHA-256 is fine. Use whatever hashing tool is available (`Get-FileHash` in PowerShell, `sha256sum` in bash, `hashlib.sha256(stripped_bytes).hexdigest()` in Python).

### Step 3 — Inject / refresh the banner in the source MD

See the "Bidirectional banner" section below for the exact format. After Step 2 writes the HTML, write a fresh banner into the source MD pointing at the new HTML. This makes the `.md ↔ .html` mirror visible from both sides.

### Why hash, not mtime

File modification times are unreliable: `git clone` resets every file's mtime to the clone moment, so a freshly-cloned repo would look "fully in sync" by mtime even though the HTML inside the repo was rendered at commit time. Hashing the source content sidesteps this. Mtime can be a *fallback* signal when no hash is present (e.g., legacy HTMLs without the meta tag), but the hash is authoritative.

### Edge cases

- **HTML exists but is malformed or unreadable** → treat as "no HTML" and render fresh.
- **Source `.md` is empty** → still go through the detection (and emit a styled but minimal page); don't produce nothing.
- **The user explicitly says "force / rerender / 重新生成"** → skip the sync check, always regenerate.
- **The user explicitly says "no, don't"** when asked → respond briefly that you skipped, do nothing further. Don't suggest other actions.

This detection workflow makes the skill safe to invoke after every conversation that mentions the README. The skill won't churn out a new HTML every time — it only fires when something actually changed.

## Bidirectional banner — point the MD back at the HTML

A `mdrise` mirror is a two-way relationship: the HTML carries the source-hash so it can detect drift, and the source MD carries a banner so readers landing on the MD see that there's a rendered version. This is what the user actually invoked the skill for — to maintain *both sides* of the mirror.

**On every successful render, the skill must inject (or refresh) a banner at the top of the source MD.** The banner has a fixed format wrapped in HTML-comment markers so it's recognizable and removable:

```
<!-- mdrise:banner -->
<div align="center">

### 👉 [看渲染版 / View styled HTML →](./<basename>.html)

</div>

<!-- /mdrise:banner -->

```

Where `<basename>.html` is the relative path from the source MD to the rendered HTML — typically `./README.html` when rendering `README.md`. If the user specified a custom output path, use the path relative to the source MD location.

**Injection logic:**

1. **Look for an existing banner block** — search the source MD for the pattern `<!-- mdrise:banner -->` … `<!-- /mdrise:banner -->`.
2. **If present** → REPLACE the entire block (including the markers) with the freshly-built banner. This keeps the link path correct if the output location changed, and guarantees the banner content always reflects the current skill version.
3. **If absent** → INSERT the banner as the very first content in the file, before any other markdown. Two exceptions:
   - If the MD starts with a YAML frontmatter block (`---` … `---`), insert the banner AFTER the closing `---`.
   - If the MD starts with a leading shebang-like HTML comment (`<!-- ... -->` for tooling), insert after that.
4. **Write the modified MD back to disk.** Yes, this means the skill *modifies the source MD*. That's the intent — the banner is part of the maintained mirror state, not user content.

**Critical invariants:**

- **Banner content is auto-managed.** If a reader edits the text inside `<!-- mdrise:banner -->` … `<!-- /mdrise:banner -->`, those edits will be overwritten on the next render. The whole block is owned by the skill. Document this in your response to the user the first time you inject a banner.
- **The banner is never rendered into the HTML.** When converting MD → HTML, strip the banner block first; HTML viewers are already on the rendered version and don't need a "view rendered version" link.
- **The banner is excluded from the source hash** (see Detection workflow Step 2). Otherwise every render would change the file, change the hash, and the next sync check would always disagree.
- **Custom hosting URL**: if the user has a deployed URL for the HTML (e.g., GitHub Pages at `https://<user>.github.io/<repo>/`) and tells you about it, prefer that absolute URL over the relative `./<basename>.html`. The relative path is the safe default when no deployment URL is known. The user can manually replace the URL between the comment markers — but it'll be overwritten on the next render; for stable customization, edit the SKILL configuration (a future feature) or just always run renders with the URL explicit in the request.

**Why this matters:** without the banner, the only way someone landing on the MD (e.g., on a GitHub repo page where README.md is what GitHub shows) discovers the rendered HTML is by hunting for the file. With the banner, the rendered version is one click away. This closes the loop — `mdrise` doesn't just generate HTML, it maintains the discoverability of HTML from the MD side.

## Handling edge cases

- **Embedded raw HTML in the MD** — preserve as-is; the author chose it deliberately. Style around it.
- **Local image references** (`![alt](./img.png)`) — leave the `src` pointing at the relative path. Note to the user that the HTML needs the image file alongside it to render.
- **Mermaid / math / custom code fences** — render as styled code blocks if you can't render the diagram itself; never silently drop them. Tell the user which fences fell back.
- **Front-matter (YAML at top of MD)** — do not render as visible content; use the title/description fields to populate `<title>` and `<meta name="description">` if present.
- **Extremely long documents** (>5000 words) — add an anchor-link table of contents at the top or as a floating sidebar. Don't compress the content.
- **Empty or near-empty MD** — produce a minimal styled HTML anyway; don't add filler.

## Output verification (before declaring done)

1. **Text-fidelity diff** — mentally scan the rendered HTML against the source MD. Any sentence in one that's not in the other? Fix it.
2. **First-screen test** — does the top of the page tell the reader what this is and set the tone? If not, the hierarchy or hero is off.
3. **Mobile sanity check** — does it hold up at ~375px width? Single column, readable type, no horizontal scroll. ToC toggle still accessible.
4. **ToC check** — is the floating ToC present (if doc has ≥2 H2)? Does the toggle icon stay visible while scrolling? Does the active section highlight as the user scrolls past it?
5. **Self-containment check** — only external resource is Google Fonts. No `<script src="...">`, no `<link>` to local CSS files.
6. **Tell the user** — output path, the aesthetic direction chosen (one sentence), and any edge cases that needed a fallback.

---
> Source: [YukiOvOb/mdrise](https://github.com/YukiOvOb/mdrise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
