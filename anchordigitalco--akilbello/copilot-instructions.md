## akilbello

> **How to use this document.** Sections 0 through 8 are the master brief. Paste it once at the start of the project and write it to `CLAUDE.md` at the repo root so it persists across sessions. Then run Batches 1 through 6 one at a time, in order, pasting each batch as its own instruction. Do not paste all six batches at once. Akil's own build rule caps a build list at roughly 15 items because longer lists produce silent errors, and each batch here is already at or near that ceiling.

# AkilBello.com v4.0: Claude Code Build Prompt

**How to use this document.** Sections 0 through 8 are the master brief. Paste it once at the start of the project and write it to `CLAUDE.md` at the repo root so it persists across sessions. Then run Batches 1 through 6 one at a time, in order, pasting each batch as its own instruction. Do not paste all six batches at once. Akil's own build rule caps a build list at roughly 15 items because longer lists produce silent errors, and each batch here is already at or near that ceiling.

**Before anything else, read Section 8.** The client folder is mounted in this project. It is about 5 GB and most of it is not website material: old builds, a 2.5 GB video library, research corpora, analytics exports, and one file of personal subscriber data that must never be opened. Section 8 says exactly which files to read, which to mine for assets, and which to leave alone. Do not explore the folder on your own.

Everything in Sections 1 through 7 is locked. Everything in the batches is locked unless a line says otherwise. Where something is genuinely undecided it is marked **RAISE, DO NOT GUESS**, which means stop and ask Akil rather than picking something reasonable.

---

## 0. Project Brief

Build v4.0 of akilbello.com, the personal and professional site of Akil Bello: standardized testing expert, writer, and speaker. Director of College Advising and FAFSA Completion Implementation at SUNY. Coined the term "highly rejective." Bylined in Forbes, The Chronicle of Higher Education, Inside Higher Ed, and The Washington Post. Quoted in the NYT, The Atlantic, Vanity Fair, and the WSJ. On-screen contributor in Netflix's *Operation Varsity Blues*.

v4.0 is a clean slate, not a refactor of v3. The current site (v3.6x live, v3.69 in the client folder) is a static HTML build migrated off WordPress with no architecture planning behind it. v4.0 plans architecture first. Content ports over where useful, structure does not carry forward by default.

The site's job, in priority order:

1. Route three different audiences to the right material fast (researchers and policymakers, media and institutions, families and counselors).
2. Land paid speaking bookings. Speaking is the primary sales feature of the site.
3. Establish that the arguments are backed by actual research and data, not opinion.

---

## 1. Non-Negotiable Rules

These apply to every file, every page, every batch. Violating one is a build failure, not a style disagreement.

### Process

- **Never build with unresolved questions open.** Resolve first, then build. If something is ambiguous, stop and ask.
- **No build without an explicit go-ahead.** Do not run a build or package a release because a batch looks finished.
- **Cap any build or task list at roughly 15 items.** Longer lists produce silent errors.
- **Mechanical technical fixes can proceed without asking.** Content and voice decisions always require Akil's explicit call.
- **Never overwrite Akil's own edits.** If he has edited a file, that version is the new base.
- Increment the version number on every build.

### Writing and copy

- **No em dashes** except inside directly quoted material. Use commas, colons, or parentheticals.
- **First person throughout**, except formal bios, which are third person.
- **No eyebrow text above any headline, anywhere on the site.** No small label line, no kicker, no category tag sitting above an H1 or H2. This rule has already been used to correct three drafts. Do not reintroduce it.
- **"Writing," never "Blog." "In Public," never "Media."**
- **"Highly rejective," never "selective"** when describing elite colleges. But never make "highly rejective" the lead identifier for Akil himself.
- **"More than thirty years"** spelled out in words, always with the "more than" modifier. Numerals are fine inside stat cards ("30+").
- **Never call Twitter "X."** Always "Twitter," always the bird icon, always `twitter.com` links.
- **No emoji** unless it is clearly ironic.
- Never list FairTest as a current employer. SUNY is current. Many third-party bios are outdated on this.
- Never position Akil as a tutor or an admissions consultant.
- Never list book blurbs or supporters as confirmed. Nobody has been formally asked.
- Family details are private. Do not surface them.

### Design and behavior

- **No orange anywhere.** Mustard/gold is the pop accent. Orange was explicitly rejected for reading as urgent or caution.
- **`object-fit: cover` on every image.** No white bars, ever.
- **CTAs are contextual to a section's real content, one per hub at most.** No generic floating "book me" or "hire me" buttons. A speaking CTA only appears attached to real keynote or briefing content.
- Google Analytics goes in at build time.

### URLs and deployment

- Static site, deployed to Netlify. No framework, no build step required to serve.
- **Every page is generated as `page-name/index.html` on disk** so it serves at `/page-name/` with no `.html` file present to redirect from.
- **Do not rely on Netlify's "Pretty URLs" toggle.** It is confirmed unreliable and often leaves the `.html` version live alongside the extensionless one. v3 has an active `/about` vs `/about.html` duplication because of this.
- All internal links use absolute paths with a leading slash and a trailing slash: `/research/`, `/writing/advice/`.

---

## 2. Tech Stack and Repo Structure

Hand-written static HTML, CSS, and vanilla JS. No React, no Next.js, no build tooling, no npm dependencies. This matches how the site is actually maintained and deployed.

- One shared stylesheet at `/assets/css/site.css` holding all tokens and shared components. Page-specific CSS goes in a `<style>` block in the page head only when it is genuinely single-use.
- One shared `/assets/js/site.js` for nav, spotlight strip, and the shared reveal interaction. Page-specific JS inline at the bottom of the page.
- No external JS libraries. The image lightbox, the accordion, and the filters are all vanilla.
- Fonts from Google Fonts with `preconnect`. Nothing else external except the GA snippet.

```
/
├── index.html                  Landing
├── research/index.html
│   └── library/index.html      papers list, static
├── insights/index.html
├── advice/index.html
├── writing/index.html          filterable archive
│   ├── research/index.html     pre-built filtered view
│   ├── insights/index.html
│   ├── advice/index.html
│   └── [article-slug]/index.html
├── projects/index.html
├── speaking/index.html
├── about/index.html
├── contact/index.html
├── book/index.html
├── resources/index.html
├── in-public/index.html        the searchable index, linked from Insights
├── crisis-construction/        ported from v3, not rebuilt in this scope
├── tutoring/index.html         unlisted, noindex, off sitemap
├── 404.html
├── assets/
│   ├── css/site.css
│   ├── js/site.js
│   └── img/
├── _redirects
├── robots.txt
└── sitemap.xml
```

`_redirects` carries forward from v3.69, extended with any v3 to v4 path changes. The existing file already handles WordPress-era legacy paths (`/blog-2/`, `/category/*`, `/author/*`, `/tag/*`). Do not drop those.

---

## 3. Design System

### 3.1 Direction in one paragraph

Editorial, high-contrast, photo-forward, generous with space. Off-white paper base with full-bleed solid cobalt blocks that mark the moments that matter. Bold sans display type set large and tight. Real photography of Akil, never stock. The interface borrows one honest visual metaphor from the subject matter, the Scantron answer bubble, and uses it as the site's single signature interaction rather than decorating everything with test imagery.

The reference images Akil collected point at three things, and only these three: give every element enough breathing room, set display type big and confident, and let one saturated color do the heavy lifting against a neutral base.

### 3.2 Color tokens

```css
:root {
  /* Primary */
  --cobalt:       #0047AB;   /* the site's color, used for full solid blocks */
  --cobalt-deep:  #00327A;   /* gradient partner, hover states on cobalt */

  /* Accent */
  --mustard:      #D9A441;   /* the pop accent, replaces orange entirely */
  --mustard-deep: #B8842A;

  /* Neutrals */
  --ink:          #10131A;   /* structural accent only, never a background fill */
  --paper:        #F7F6F2;   /* default page background */
  --white:        #FFFFFF;   /* card surfaces on paper */

  /* Derived */
  --text-dim:     rgba(247,246,242,0.72);  /* body text on cobalt */
  --rule:         rgba(16,19,26,0.12);     /* hairlines on paper */
}
```

Rules on color:

- **Cobalt solid blocks are a site-wide pattern, not a per-page decoration.** They mark key conversion and identity moments. Confirmed page-level placements, and there are exactly three: the Landing hero, the About page's Three Bios section, and the Contact page's photo block. The Global Spotlight Strip reuses the same treatment as global chrome. Do not add a fourth page-level block without asking.
- **Azure `#007FFF` is rejected for large fills.** It reads neon. Use it only as a small accent, if at all.
- **Black is a structural accent only.** Rules, borders, type. Never a background fill. The site is not dark-themed.
- No orange. See Section 1.

### 3.3 Typography

**Status: not locked.** The architecture doc says Fraunces and all serif display are out, direction is bold sans caps display, and the specific faces were handed to a designer rather than spec'd. Four candidates are named: Space Grotesk, Archivo, General Sans, Neue Montreal.

Build with this token trio so the swap is a one-line change:

```css
:root {
  --font-display: 'Space Grotesk', system-ui, sans-serif;
  --font-body:    'Inter', system-ui, sans-serif;
  --font-mono:    'IBM Plex Mono', ui-monospace, monospace;
}
```

Space Grotesk, Inter, and IBM Plex Mono are the working default because the existing v4 card mockup already uses them and they render the intended feel correctly. Inter specifically is what Akil's own "really good design" reference image is pointing at, used as a bold display face rather than only as body text.

**Deliverable in Batch 1:** build `/type-specimen/index.html`, an unlinked page (noindex) that renders the Landing hero headline, one card, one hub headline, and a paragraph of body copy four times over, once in each candidate display face, side by side at real size. This is what Akil needs to lock the choice. Do not lock it yourself.

Type scale:

| Role | Size | Weight | Tracking | Leading |
|---|---|---|---|---|
| Display / hero H1 | `clamp(2.4rem, 5.2vw, 4.2rem)` | 700 | `-0.015em` | 1.02 |
| Page H1 | `clamp(1.9rem, 4.2vw, 3.1rem)` | 700 | `-0.01em` | 1.08 |
| Section H2 | `clamp(1.5rem, 2.6vw, 2.1rem)` | 700 | `-0.01em` | 1.12 |
| Card H3 | `1.18rem` | 700 | `-0.005em` | 1.22 |
| Body | `1rem` / `1.0625rem` | 400 | 0 | 1.6 |
| Small / meta | `0.86rem` | 400 | 0 | 1.55 |
| Label / mono | `0.68rem` | 500 | `0.14em` | 1 |

Mono is used only for card letters (A/B/C), category labels, dates, and data figures. It is the "test document" voice of the typography. Never for body copy.

### 3.4 Layout and space

- Max content width `1180px`, gutters `40px` desktop and `22px` mobile.
- Baseline spacing unit `8px`. Section vertical padding `clamp(64px, 9vw, 128px)`.
- Breakpoints: `1024px`, `820px`, `560px`.
- Give everything more room than feels necessary. Every reference Akil saved is about breathing room. When in doubt, add space rather than another element.
- Border radius: `2px` on cards and buttons. This site is square and editorial, not rounded and friendly. The one exception is the Scantron bubble, which is a true circle.

### 3.5 Motion

- Transitions `.3s` to `.4s`, easing `cubic-bezier(.2,.7,.2,1)`.
- Content animates in on page load with a short stagger (`.05s`, `.16s`, `.27s` for a three-item group). Translate up 18px and fade, nothing more elaborate.
- **`@media (prefers-reduced-motion: reduce)` must disable every animation and transition on the site.** Not most of them. Every one. This is non-negotiable for an education access site.

### 3.6 Accessibility

- Every interactive card is a real `<button>` or `<a>`, never a div with a click handler.
- Every hover reveal must have an identical `:focus-visible` state. Keyboard users see the same thing mouse users see.
- Focus outline: `2px solid var(--mustard)`, `outline-offset: 3px`.
- Contrast: body text on cobalt uses `--text-dim` at minimum, headline text on cobalt is pure white. Verify 4.5:1 on all body text and 3:1 on all large text before calling any batch done.
- Every image has real alt text describing what is in it. Decorative background layers get `aria-hidden="true"`.

---

## 4. The Signature Interaction

The site has **one** interaction pattern, deliberately reused across three pages rather than reinvented per page. Build it once in `site.css` and `site.js` as a reusable component, then apply it.

**Behavior.** Minimal at rest: a headline and a Scantron bubble. On hover or focus, the element expands upward and reveals supporting detail, and the bubble fills in like a pencil mark. On page load the group animates in with a stagger.

**The Scantron bubble.** A thin outline circle rendered as inline SVG, subtle, positioned behind or within the element, never dominant. The letter (A, B, C) stays visible next to or inside it. At rest the ring is a hairline stroke at 55% opacity. On hover or focus the ring goes mustard and an inner filled circle scales from 0 to 1 with a slight overshoot, `cubic-bezier(.34,1.4,.4,1)`. It should read as the physical act of filling in an answer, because that is the argument the whole site is making.

**Where it is used, and only here:**

1. **Landing**: the three answer cards. This is the canonical implementation.
2. **Speaking**: the featured-talk cluster. One talk headline at rest; hover or click reveals related engagements on the same topic across other venues.
3. **About**: the three bios. A photo at rest; hover or click reveals the full bio text.

A working reference implementation of the card behavior already exists at `v4-homepage-cards-mockup.html` in the v4.0 folder. Read it before building. Its CSS for `.card`, `.bubble`, `.detail`, and the `card-in` keyframe is correct and should be lifted more or less directly, with the copy replaced by the locked copy in Batch 2. Note that its headline copy and its `.tag` eyebrow are placeholders that violate the no-eyebrow rule. Drop the `.tag` element entirely.

**On touch devices**, hover-only reveals do not exist. Cards render in their expanded state at `820px` and below, or expand on first tap with the navigation happening on second tap. Pick one and apply it consistently.

---

## 5. Global Chrome

### 5.1 Navigation

```
Start Here ▾   Research   Insights   Advice   Speaking   Contact   About   Resources
   └ Writing
   └ Projects
```

- Resources is top-level only and is not duplicated inside the Start Here dropdown.
- Crisis Construction is reachable through Research. It gets no nav slot.
- Book is reachable through About. It gets no nav slot.
- There is no Media nav item. The press kit is deferred and the speaker kit lives on About.
- Nav sits on the paper background on interior pages and directly over the cobalt hero on Landing.

### 5.2 Global Spotlight Strip: LAUNCH-CRITICAL

This is the mechanism that solves cross-audience findability for rare, high-weight items that do not belong to any one hub and need to be reachable from any entry point. Historically this is about two items a year, things like Crisis Construction or an open letter.

- **It is part of the global chrome**, the persistent frame around every page. It is not hub content and it does not live on Landing or on any single hub home. It appears site-wide.
- **Capacity is two concurrent slots, maximum.** Reserved for genuinely high-weight items. This is not a rotating content feature.
- **It uses the cobalt-block pattern** for "this matters" signaling, consistent with the Landing hero, About's Three Bios, and Contact's photo block.
- **Two concurrent items render compact**, side by side or stacked. They do not each get a full-weight block.
- **When nothing qualifies, the strip is absent entirely.** No empty state, no reserved real estate, no placeholder.
- **Control is manual.** Akil sets and clears it. No auto-rotation.
- **Expiration is per item, set at creation.** Default assumption is about three months followed by manual review, not an auto-expiring timer. An item with its own natural anchor, like an external launch date, uses a link-target switch at that date instead.

**Implementation.** Drive it from a single JS config object at the top of `site.js`:

```js
const SPOTLIGHT = [
  // { label: '', headline: '', href: '', added: 'YYYY-MM-DD', review: 'YYYY-MM-DD' }
];
```

An empty array renders nothing and injects no DOM. One entry renders a single full-width compact strip. Two entries render the paired layout. Three or more should log a console warning and render only the first two. Akil edits this array by hand, so keep it at the very top of the file, clearly commented, with the shape documented inline.

**The mechanism itself must be live at launch**, even if the array ships empty.

### 5.3 Footer

- Column-grouped, stacking vertically on mobile.
- Primary links: Start Here, Research, Insights, Advice, Writing, Speaking, About, Contact.
- Secondary links: Projects, Book, Resources, Privacy, Copyright.
- The footer intentionally duplicates nav-adjacent items. Its job is to be the complete sitemap, so the redundancy rule that governs main nav does not apply here.
- **The newsletter signup sits above the footer link columns, never below.** Conversion attention is highest before a reader scans a dense link list.
- Social links: Substack `https://akilbello.substack.com`, LinkedIn `https://www.linkedin.com/in/akil-bello/`, Instagram `https://www.instagram.com/akilbello/`, Twitter with a bird icon and a `twitter.com` link.
- **Exception: the Landing page ends with the newsletter signup and has no footer link block beneath it.** The header nav plus the three cards already do that page's navigational work.

### 5.4 Newsletter block

The newsletter is named **"Signal, not Noise."** That is a declarative stance, not "Signal vs. Noise." Do not render it as "vs."

Standard block: a one-line value statement, an email field, a submit button, and a short reassurance line. Substack is the destination. Keep it to one row on desktop.

---

## 6. Assets

Source assets live in the client folder alongside these specs. Copy what is used into `/assets/img/` with descriptive kebab-case filenames. Do not hotlink from the source folders. **Section 8 is the full map of that folder, including what is off limits. Read it before you touch anything.**

**The primary asset source is `akilbello-com-v3.69/images/`, not the raw `Media/` folder.** Those files are already web-sized, already sensibly named, and already in use in production. Reach into `Media/` only for something v3.69 does not have.

| Need | Source, in priority order |
|---|---|
| Outlet logos for the "As seen in" strip | `v3.69/images/logo-*.png`. Already cropped and named: `logo-netflix`, `logo-forbes-hd`, `logo-nyt-hd`, `logo-wsj`, `logo-vf-hd`, `logo-wapo`, `logo-chronicle-v2`, `logo-msnbc`, `logo-cbs`, `logo-ihe`, `logo-atlantic`. Use these, not the duplicate-heavy originals in `Media/media logos/`. |
| Speaking photography | `v3.69/images/speaking-*.jpg`. Includes `speaking-gates-admission-reimagined.png` and `speaking-mercy.jpg`, which are the two venues named in the Speaking featured-talk cluster. Also amazon, gmac, harvard-sdp, sidwell, aapf-rally, masters, npea. |
| Headshots and about photography | `v3.69/images/`: `headshot-teaching.jpg`, `YP3_6793_headshot_formal.jpg`, `hack-the-gates.jpg`, `about-keynote-meritocracy-bloomington.jpg`, `about-unfiltered-rules-in-place-of-resources.jpg`. Higher-resolution originals are in `Media/Akil photos/Akil Bello potential Headshots/` if a bigger crop is needed. |
| Photo gallery ("The Real Work") | `v3.69/images/gallery/`, already event-tagged for the existing carousel. |
| A new hero photograph | `Media/Akil photos/Akil Presenting and Action Shots/`. Roughly 85 files. These are full-resolution originals, 300KB to 14MB. **Resize and convert to WebP before committing anything from here.** |
| Logo, wordmark, monogram | `logo/`: use the PNGs (`AB logo.png`, `AB logo Alt.png`, `Circle AB logo.png`, `logo.png`). Ignore the `.psd` files. |
| Favicon | Carry forward from v3.69: `favicon.png`, `favicon.ico`, `favicon-192.png`. The correct mark is the AB monogram, navy background, white serif letters, blue arc. **Do not create a `favicon.svg` and do not add any `<link rel="icon" type="image/svg+xml">` tag.** A wrong SVG favicon was already deleted once. |
| Paper PDFs for the Research list | `v3.69/research/`. Exactly five papers, each with a matching cover JPG already generated. See Section 8. |
| Video | See Section 8. **Do not commit any video file to the repo.** |

**The hero photograph must be real photography of Akil**, speaking, teaching, or walking. Never stock. Treat it transparently so the cards carry the visual weight and the photo supports rather than competes. The wireframe deck shows a knockout of Akil pointing, placed right, bleeding off the top and bottom of the cobalt block. That composition works, build toward it.

**Hero background behavior:** the image stays visually anchored and responsive while a transparent layer scrolls over it. Reference `saragoldrickrab.com` for the effect. It is not a static background and it is not a naive `background-attachment: fixed`, which breaks on iOS. Use a `position: sticky` or transform-based approach and verify on mobile Safari.

An optional sound-on toggle for a speaking clip is a stretch idea only. If it ships, it must be user-initiated. **Never autoplay audio.**

---

## 7. Voice, For Any Copy You Draft

Most copy in this build is locked and quoted verbatim in the batches below. Where you need to write connective tissue, alt text, or microcopy, match the register.

Akil writes in seven distinct registers, not one voice at different volumes. The relevant mapping:

| Page | Register |
|---|---|
| Landing, Speaking | Spoken and live. Direct address, short punches, closes on something unexpected. |
| Research | Formal. Citation-driven, evidence walked through before the conclusion. Wit rationed to one line at the close. |
| Insights, Advice subheads and pull quotes | Twitter and blog short-form. Declarative, confrontational, wordplay as default, rhetorical binaries. |
| About narrative, Book | First-person indignant. Personal stakes named directly, controlled indignation, not venting. |
| Testimonials, FAQ | Peer and vernacular. Dropped guard reads as authentic here. |

Recurring devices worth reaching for: reveal-the-euphemism ("We say good and we mean famous, we say elite and mean rich, we say merit and we mean metrics"), rhetorical binaries (Good vs. Known, Merit vs. Metrics, Rejective vs. Selective), and the frame-reset declarative, a single flat sentence that resets the conversation.

**If you are unsure whether a line sounds like him, do not write it.** Flag the gap and let Akil fill it. Content and voice decisions are his call, always.

---

## 8. The Client Folder: What to Use, What to Ignore

The `akilbello.com/` folder is mounted in this project. It is roughly 5 GB and most of it is not website material. It is a working archive: old builds, research corpora for Akil's writing, raw video, analytics exports, and personal files.

**Do not glob, grep, or index the folder as a whole. Do not open anything not listed as USE below.** Reading broadly here will fill your context with 100 MB research PDFs and 2.5 GB video files and teach you nothing about the site.

### 8.1 Read these first, in this order

| File | What it gives you |
|---|---|
| `akilbello-com v4.0/v4-architecture-and-visual-system_aug_1.md` | Nav, sitemap, footer, Spotlight Strip, color, typography direction, terminology, build rules. |
| `akilbello-com v4.0/v4-content-and-page-specs_aug_9.md` | Every page's locked copy and structure, plus the full News vs. Noise pairings with source citations and the FairTest year-by-year table. |
| `akilbello-com v4.0/v4-voice-reference.md` | Seven registers, phrase bank, register-to-section mapping. |
| `akilbello-com v4.0/v4-homepage-cards-mockup.html` | Working reference implementation of the card and Scantron bubble interaction. Lift its CSS. |
| `Background and Research/akil-bello-preferences-supplement.md` | Biography, career arc, book context, and the biographical landmines in Section 1. |

That is the complete required reading. Five files. Everything else in the folder is either a supporting asset or noise.

### 8.2 USE, for specific jobs

**`akilbello-com v4.0/`** is the primary source folder.

- `akilbello site v4.pptx` (43 MB): Akil's own wireframes for Landing, the three hubs, Writing, and About. **Layout intent only.** Extract the slides to images rather than parsing the XML. Several slides show copy that has since been superseded, including the old Landing headline and two sections that were explicitly cut. When the deck and the specs disagree, **the specs win.**
- `design ideas/` (3 PNGs): Instagram screenshots of design tips. They are pointing at three ideas and nothing more: breathing room, big confident display type, one saturated color against a neutral base. Do not try to reproduce the furniture or bicycle sites in them.
- `Akil Bello - Media Presentations Articles cv.xlsx`: the source data for the "In Public" index. Articles, media citations, podcasts, TV. Parse it when you build `/in-public/`.
- `Akil Bello - Media Presentations Articles cv - Media - Video Appearences.tsv`: the video subset of the same, small and easy to read.

**`akilbello-com-v3.69/`** is the live site. It is a **parts donor, not a design reference.** Take content and assets. Take nothing about how it looks.

- `images/`: the primary asset source. See the Section 6 table.
- `research/`: **exactly the five papers the Research page needs**, each already paired with a cover JPG: `dirty-dozen`, `fairtest-merit-awards-myths-and-barriers`, `pandemic-test-optional-bello-baker`, `paradoxical-purposes`, `shsat-testimony-2019`. This matches the spec's "three to five papers at launch" and its note that the real count of authored or co-authored papers, including the NYC Council testimony, is five. Do not go looking for more.
- `posts/`: 44 article directories, the source for the Writing archive. Every article the specs name by title is here, including all three Advice featured picks (`the-myths-of-gpa-in-college-admissions-explained`, `its-so-hard-to-get-into-college`, `what-is-a-good-college`) and the Insights ones (`the-ny-times-doesnt-cover-college`, `large-language-models-misnamed-ai-are-not-intelligent`, `cutting-room-the-misguided-war-on-test-optional`, `cutting-room-floor-the-hidden-factors-influencing-college-admissions-decisions`, `first-black-graduate`). **Skip `posts/retired/`.** Those are deliberately retired and do not migrate.
- `crisis-construction/`: port as-is with paths updated. Do not rebuild it.
- `tutoring.html`: port to `/tutoring/index.html` with `noindex`.
- `_redirects`: carry forward and extend. It already handles the WordPress-era legacy paths.
- `favicon.png`, `favicon.ico`, `favicon-192.png`: carry forward.
- `.claude/settings.json`: the Claude Code config from the v3 work. Read it once to see the existing conventions, then decide whether to carry it over.

**`Background and Research/`** is mostly Akil's research library for his own writing, not site content. Three files are useful:

- `akil-bello-preferences-supplement.md`: required reading, listed above.
- `akilbello-site-context.md`: **process rules only.** Its design system section describes v3 (Fraunces, Outfit, Syne, `#1B9FE0`, dark navy homepage) and is **entirely superseded by Section 3 of this document.** Do not implement anything from it. The parts that still apply are the voice rules, the social links, and the build process rules, which are already restated in Section 1.
- `akil_bello_brand_context 2.md` (44 KB): the newer of the two brand context files. Optional. Read it only if you need more depth on positioning than Section 7 gives you. **Ignore `akil_bello_brand_context.md`**, the older one.

**`logo/`**: use the PNGs. Ignore the `.psd` files, you cannot open them.

**`Media/`**: reach here only for what v3.69's `images/` folder lacks.

- `Akil photos/Akil Bello potential Headshots/` and `Akil photos/Akil Presenting and Action Shots/`: full-resolution originals. Fine to use, but every file needs resizing and WebP conversion first.
- `Akil photos/Other Pictures of Akil/`: **contains family photos.** One is named `akil alex adam.jpg`. Family is private per Section 1. Do not use anything from this folder without asking.
- `Akil photos/Akils Selfie/` and `Akil photos/photos with friends/`: **do not publish without Akil's explicit approval.** Personal, not professional.
- `media logos/`: the raw originals. Heavy on duplicates, several files per outlet. v3.69's `logo-*.png` set is the cleaned-up version of exactly this. Prefer that.

**`Akil Bello Speaker Kit/`**: the speaker kit files for the About page download links. Five files, and they are not interchangeable. `Akil_Bello_Speaker_Kit.pdf` is 4.8 MB. The two "Speaker Book" PDFs are 17 MB each. **RAISE, DO NOT GUESS:** ask Akil which file is the canonical public download before linking one, and do not serve a 17 MB PDF without compressing it.

### 8.3 DO NOT USE

Not "use sparingly." Do not open these at all.

| Path | Why |
|---|---|
| `akilbello-subscribers-all.csv` | **Personal data.** A real subscriber email list. Never read it, never copy it, never reference it, and make sure it never lands anywhere near the repo or a commit. |
| `Web analytics/` | Contains `Akil Bello _ Facebook.pdf`, `Your Twitter Data.pdf`, and platform exports. Personal account data. Also unnecessary: the analytics work is already done. The three Advice featured articles were selected from this data using 2023 to 2026 Jetpack pageviews. **Do not re-derive that decision.** |
| `UCSD and uc berkeley/` | About 130 files, the research corpus behind Crisis Construction. Several PDFs exceed 100 MB, one is 135 MB. Crisis Construction ports from v3 rather than being rebuilt, so you never need this. |
| `Old sites/` | Twelve zipped builds from v26 through v3.68, plus old Crisis Construction HTML. Roughly 1 GB. **Never unzip any of it.** v3.69 is the only version that matters. |
| `akilbello-com-v3.69.zip` (199 MB) | A zip of a folder you already have unpacked. |
| `bellowings.WordPress.2026-05-06.xml` and the media XML | The WordPress export. Every post that matters is already migrated into `v3.69/posts/`. Touch this only if a specific post body turns out to be missing, and then read only that post's node. |
| `The Twitter Files.docx` (603 KB) | A voice source that has already been distilled into `v4-voice-reference.md`. Use the distillation. |
| `Background and Research/Surviving Standardization - Book Proposal - Jan 2026.md` (120 KB) | Agent-packet material. The Book page copy is already locked and the spec is explicit that proposal content does not go on the page. |
| `Background and Research/` cover letters and resumes | Job application material. Not site content. |
| `Background and Research/` article PDFs and lecture transcripts | Akil's published work and talk recordings. They are research inputs for his writing, not website copy. The one legitimate use is confirming a title, outlet, or date for the In Public index or a featured card. Open a single file for that, not the set. |
| `Media/videos/` and `akilbello-com v4.0/4.0 Reel.mp4` | Seven files up to 2.5 GB, plus a 777 MB reel. **No video file gets committed to a static repo.** The Speaking page's video section needs a hosting decision (YouTube or Vimeo embed) before it can be built. Flag it, do not solve it by committing a file. |
| `Media/wp-uploads/` and `v3.69/images/wp-uploads/` | The legacy WordPress media library. Only pull a single file if a ported post references an image that is genuinely missing. |
| `akilbello-com v4.0/Untitled-3.psd`, `Media/Akil photos/Untitled-*.png` | Working files with no context. |
| Any `.DS_Store`, any `~$` file | macOS and Office junk. `~$uc_open_letter_signers_extracted_only.xlsx` is a lock file, not a spreadsheet. |

### 8.4 Superseded files inside the v4.0 folder

The v4.0 folder contains multiple dated versions of the same two documents. **Read only the newest of each.** The others are earlier drafts and will contradict the current specs if you read them.

| Read this | Ignore these |
|---|---|
| `v4-architecture-and-visual-system_aug_1.md` | `v4-architecture-and-visual-system.md` (missing the Spotlight Strip and the Research vs. Projects rule) |
| `v4-content-and-page-specs_aug_9.md` | `v4-content-and-page-specs_aug_1.md`, `v4-content-and-page-specs.md` (the second covers only four pages) |

### 8.5 Precedence, when sources disagree

1. This document.
2. `v4-content-and-page-specs_aug_9.md` and `v4-architecture-and-visual-system_aug_1.md`.
3. `akilbello site v4.pptx`, for layout intent only.
4. The v3.69 site, for content and assets only, never for design.

Anything in `akilbello-site-context.md` about fonts, colors, or page backgrounds describes v3 and is dead. If you catch yourself about to use Fraunces, Outfit, Syne, `#1B9FE0`, or a dark navy homepage, you are reading the wrong document.

---

# BATCH 1: Foundation and Global Chrome

Do not build any page content in this batch.

1. Initialize the repo with the directory structure in Section 2. Write this entire brief to `CLAUDE.md` at the root.
2. Build `/assets/css/site.css` with the full token set from Section 3.2, the type scale from 3.3, the layout scale from 3.4, and the motion rules from 3.5.
3. Wire Google Fonts with `preconnect`, loading Space Grotesk, Inter, and IBM Plex Mono at the weights the scale requires.
4. Build the reusable reveal-card component from Section 4: markup pattern, CSS, and the Scantron bubble SVG. Read `v4-homepage-cards-mockup.html` first and lift its working CSS.
5. Build the global header and nav from 5.1, including the Start Here dropdown with keyboard support and the mobile menu.
6. Build the Global Spotlight Strip from 5.2, including the `SPOTLIGHT` config array. Ship it with an empty array so it renders nothing.
7. Build the footer from 5.3 and the newsletter block from 5.4, including the Landing-page variant that omits the footer link columns.
8. Build the cobalt-block section component as a reusable class. It is used in four places: three page-level blocks plus the Spotlight Strip.
9. Copy the logo, favicon, and outlet-logo assets into `/assets/img/` per Section 6, pulling from `v3.69/images/` first and `Media/` only for gaps. Wire the favicon with `.png` and `.ico` only. Resize and convert to WebP anything sourced from `Media/`. Commit no file over 500 KB and no video at all.
10. Build `/type-specimen/index.html` per Section 3.3, `noindex`, unlinked from anywhere.
11. Set up `robots.txt`, a stub `sitemap.xml`, and carry `_redirects` forward from v3.69.
12. Add the Google Analytics snippet.
13. Build a `/_template/index.html` skeleton: header, spotlight mount, `<main>`, newsletter, footer. Every page in later batches starts from this file.

**Stop and report before Batch 2.** List anything in the token set or the component API you had to decide yourself.

---

# BATCH 2: Landing Page

Status: **FULLY LOCKED.** Copy is final. Do not improve it.

**File:** `/index.html`

**Structure:** cobalt hero block with headline and three cards → testimonial → newsletter signup. Page ends there. No footer link block.

**Headline (H1, no eyebrow above it):**

> What kind of problem are you trying to solve?

This supersedes "Where should we start?" and "What can I help you with today?", both of which appear in older mockups. The final version positions Akil as the authority answering the visitor's question rather than a peer co-investigating alongside them.

**The three answer cards.** Each uses the signature reveal component. Headline visible at rest, supporting line revealed on hover or focus.

| | Headline | Supporting line | Destination |
|---|---|---|---|
| **A** | Show me the evidence. | Research on the numbers behind admissions, access, affordability, testing, and rankings. | `/research/` |
| **B** | Explain the game. | How institutions use metrics, incentives, and selective storytelling to make decisions. | `/insights/` |
| **C** | Help me make a call. | Practical guidance for choosing schools, handling tests, and cutting through unhelpful noise. | `/advice/` |

Cards are **evenly aligned, not staggered.** The moving-company reference Akil looked at informs interactive behavior only, never positioning.

**Testimonial:**

> "Challenged us to think critically about equity, opportunity, and the systems that shape postsecondary access."
>
> — Kourtney Cockrell, JPMorgan Chase & Co.

Landing's testimonial is the ideas-and-rigor one. Personality is secondary here. This quote already satisfies that.

**RAISE, DO NOT GUESS:** her full title is "Vice President and Regional Director, Global Philanthropy and Corporate Responsibility, JPMorgan Chase & Co." The spec says it is long enough to need trimming for display and the trimmed version is not yet decided. Render the short form above and ask Akil for the trim.

**Explicitly cut, do not add back:**

- No skip-link. Start Here in the nav already does that job.
- No secondary link row. "Popular Writing," "Upcoming Talks," and a per-category "Current Issue" link were all considered and cut. Each hub's own featured section does that job natively. They appear in the older PowerPoint wireframe. Ignore them.
- No "What people are reading right now" article row. Also in the old wireframe, also cut.

**Build checklist:**

1. Cobalt hero block, full bleed, with the real Akil photograph treated per Section 6.
2. Giant ghosted "ABC" letterforms behind the cards, low opacity, decorative, `aria-hidden`.
3. Three reveal cards with Scantron bubbles, stagger-animated in.
4. Testimonial block on paper.
5. Newsletter signup as the final element.
6. No footer link block. Verify the Landing variant of the footer component is what renders.
7. Verify keyboard parity on all three cards.
8. Verify reduced-motion kills the stagger and the bubble fill.

---

# BATCH 3: The Three Hubs

All three are **FULLY LOCKED** on copy. Build all three in this batch, then stop.

## 3A. Research: `/research/`

**Headline:** The information behind Insights and Advice.
**Subhead:** My original papers and data, plus outside research worth knowing.

Deliberately not "elsewhere on this site." Research draws on outside scholarship too, not only material supporting the other two hubs.

**Structure:** intro text → featured work → papers list (static) → dataset search tool.

- **Original papers:** a static list, not a search tool. Three to five papers at launch. The real count of co-authored or authored serious academic papers is five, including the NYC Council testimony. That is a real number. Do not pad it.
- **Dataset search tool:** at launch this is a **static section with entries, not a working search engine.** The interactive filter-and-sort version is deferred until the GitHub data layer exists. Build the section, list the datasets, and link each one out to its Projects page where one exists.
- **Crisis Construction:** anchored here with real weight, not one card among many. The fuller branded explanation lives on Projects. Link to `/crisis-construction/`, which ports from v3 rather than being rebuilt in this scope.
- **First Black Graduate:** **not launch-ready as an interactive tool.** At launch it is a simple static page or stays a regular post. Do not build a filter interface for it.
- **Test-history database:** PDF-only reference for now. Same deferred treatment.
- **No bibliography section.** "Further Reading" links out to the existing Resources library's "Books, podcasts, and public scholarship" category rather than duplicating it.
- **Do not feature** "Large Language Models (misnamed AI) are Not Intelligent" here. It is writing and commentary, not research. It goes to Insights.
- **No speaking CTA on this page by default.** If one appears at all it is small, text-based, and tied to real keynote or briefing content.

## 3B. Insights: `/insights/`

**Headline:** What the story leaves out
**Subhead:** Reporters and institutions reach for the simplest version of a story. I add back what got cut, the data, the context, the parts that complicate the narrative.

**Structure:** intro text → featured article → Signal vs. Noise preview → link out to the full "In Public" index.

- **Featured article:** "The Reckless Rankings Game," The Chronicle of Higher Education, October 14, 2022 issue. Strong artwork, use it.
- **Signal vs. Noise:** Insights' version of the paired claim-and-response device. **Structurally distinct from Advice's "News vs. Noise." Do not merge them.** Here it is text-only, no screenshots and no embeds. The signal (Akil's work) leads, and the noise is a light contextual tag underneath it. That is the opposite weighting from Advice, where the noise source is a full visual artifact. Insights and Research anchor in the work itself; Advice confronts the noise directly.
  - In the real system this is **generated, not hand-curated**, from the In Public index: any entry carrying a `responds_to_narrative` (the claim being addressed, one line) and a `responds_to_source` (citation and link) surfaces here automatically. At launch, build it as a static section following the same data shape so the generation layer can be wired in later without a redesign.
- **"In Public" index:** lives on its own page at `/in-public/`, not embedded here. Insights-home shows a compact teaser only, with a "See all coverage →" link. Modeled on Muck Rack: excerpt, outlet or venue, date, link out, filterable by type. **Never reproduce an article in full**, which also sidesteps any rights question.
- **No media authority strip.** Dropped as redundant. Research establishes credibility site-wide and In Public surfaces outlet names naturally.
- **Rapid Media Response CTA:** kept, and given real weight now that search moved to its own page. This is the one CTA on this hub.

Also confirmed here, not on Advice: "The NY Times Doesn't Cover College," and the pairing of David Leonhardt's "The Misguided War on the SAT" against Akil's Inside Higher Ed response, which runs as a single feature rather than a News vs. Noise pairing.

## 3C. Advice: `/advice/`

**Headline:** Let's make the call.
**Subhead:** Choosing a school, prepping for a test, reading an acceptance or a rejection, whatever the decision is, here's what actually matters and what you can ignore.

Answers Landing Card C directly. A visitor who clicks "Help me make a call." arrives here and gets answered in the same voice. The headline leads with the advice, not with the debunking. The debunking is the mechanism, not the page's stated job.

**Featured writing: cards, not a plain list.** Confirmed visual preference. Three articles, chosen from 2023 to 2026 Jetpack pageview data:

1. **The Myths of GPA in College Admissions Explained**
2. **It's So Hard (to get into college)!**
3. **What Is a Good College?**

Do not substitute. "The P in PSAT Doesn't Stand for Practice" was dropped for seasonality despite strong numbers, and "The NY Times Doesn't Cover College" was reassigned to Insights.

**"News vs. Noise" section: LOCKED.** Structural model is `aronfrishberg.com`. Look at it before building. Clean, simple, single-issue editorial device.

- **Hero:** the most recent item, or a small featured set at the top.
- **On-page section definition, written as visible copy not just internal spec:** every popular narrative strips out nuance and sometimes misleads, and that is the Noise. The News side tells the true, complicated version: strip the sales pitch, surface the confounding factors, show the data. It is a clarifying spin, not a positive one.
- **"Also in This Issue" layer:** full-width, two-column. Each row pairs a News item and a Noise item side by side, each with its own image header. **The Noise side uses a screenshot of the actual article being critiqued.**
- **Self-contained content rule, non-negotiable:** each pairing is fully readable on the page. Two to three paragraphs plus supporting data per side. No click-through required to get the whole argument. A link to a fuller post is optional, never load-bearing.
- **"Dig deeper with me →":** an optional link out to a related Insight or Research piece, used only where a genuine connection exists. Do not force one onto every pairing.
- **Past Issues layer:** same two-column structure, compressed, using an **inline accordion that expands in place.** No page load, no link-out. Four to five archived issues fully readable without navigation.

**Five pairings are content-ready.** Whether all five ship or launch trims to three is a build-time scope call, not a content-quality one. The subjects: AI in admissions and the overselling of it; testing reinstatement; Southern migration; HBCU affordability. Full source citations and supporting data for each are in `v4-content-and-page-specs_aug_9.md` under Advice-Home. Pull them from there verbatim rather than researching fresh, and cite every source.

The testing-reinstatement pairing uses the FairTest test-optional count by year, 2018 through 2026, rising from 1,000+ to 2,085+. That table with its nine archive-link sources is in the spec doc. **Treat it as directionally correct scaffolding, not final copy.** The underlying dataset gets a fresh analysis pass in a separate project before launch.

**Tutoring:** no card, no section. **One subtle inline text link only**, to inquire. The page stays unlisted, noindex, and off the sitemap.

**Resource directory:** do not build a second one. Advice-home teases and links into `/resources/`.

**Build checklist for Batch 3:**

1. Research page, all five sections, dataset section static.
2. Insights page with the Signal vs. Noise static section built to the eventual data shape.
3. `/in-public/` index page, Muck Rack model, filterable by type, excerpt only.
4. Advice page hero and featured cards.
5. News vs. Noise section with the two-column pairing layout.
6. Past Issues accordion, vanilla JS, expanding in place.
7. The single inline tutoring link.
8. Verify Advice's News vs. Noise and Insights' Signal vs. Noise are visually and structurally distinct.
9. Verify the accordion is keyboard operable and correctly labeled with `aria-expanded`.

**Stop and report.** Flag any pairing where the supporting data in the spec doc was insufficient to write a self-contained two-to-three-paragraph News side.

---

# BATCH 4: Speaking, About, Contact

## 4A. Speaking: `/speaking/`

**This is the primary sales feature of the site.** Not a comprehensive talk archive. A credibility page built to land corporate and large-endowment bookings.

Tone target for any copy you draft: Sara Goldrick-Rab's public authority, Malcolm Gladwell's crossover credibility, Scott Galloway's directness without the anger, a less academic Tressie McMillan Cottom. That is guidance, not text to reuse.

**Structure, in order:**

1. Intro and positioning hero
2. **On the calendar**: a compact strip directly under the hero showing live upcoming engagements. Not buried at the bottom.
3. Featured talk, using the signature reveal component
4. Video and clips
5. Talk range
6. Testimonials
7. Book a Talk CTA

**Locked copy:**

> **Ideas worth bringing into the room.**
>
> From keynote stages to policy briefings, I help institutions see where policy and procedure quietly protect the status quo, and make better decisions because of it.

Talk-range section intro:

> From college course guest lectures to Fortune 500 boardrooms, the questions don't change. The stakes just get bigger.

CTA block, retained verbatim from v3.66:

> **Ready to book?**
> If your audience is ready to question something they thought was settled, let's talk.
> Rates are reasonable and based on your endowment.

Calendar strip label: **On the calendar**

**Featured-talk mechanism.** Reuses the homepage cards' reveal language rather than inventing a second interaction. One featured talk headline; hover or click reveals a cross-venue cluster of related engagements on the same topic. The worked example:

> **Lecture on Tap: Rejective vs. Selective** → reveals → *also discussed at:* Gates Foundation ("Admission Reimagined"), Mercy University ("What Is a Good College?")

This does double duty: a compelling single feature, plus proof of range and repeat institutional demand, without needing a comprehensive list.

**Talk range** is curated, not comprehensive. It should **visibly skew toward Amazon, Gates, and Mercy-tier engagements over ACAC-circuit volume.** That skew is the point. It signals scale and openness to corporate bookings.

**Testimonials:** the full set lives canonically here. Speaking is the sales page.

**Speaker kit:** lives on About, not here. Speaking links to it rather than duplicating the assets. One canonical home, multiple entry points.

**Press kit:** deferred, not part of launch. Do not build a stub.

**No standalone "In Public" page linked from here.** v3.66's static version spent real estate on the appearance of comprehensiveness without the substance. Its real function is served by `/in-public/`, built in Batch 3. Any remaining lightweight credibility-quote content folds into the Testimonials section here.

The site-wide CTA rule is not broken on this page. It is fulfilled at maximum intensity, because the entire hub's content is the sales case.

## 4B. About: `/about/`

**Design principle:** echoes Landing's register without duplicating it. Minimal, photo-forward. **No hero image.** The three bio photos carry the visual weight the way Landing's three cards carry Landing's.

**Headline, no subhead** (mirrors Landing's headline-only approach):

> Who is Akil? Depends on Who's Asking.

**Structure:**

1. Headline only.
2. **Three Bios**: Formal, Real, Unfiltered. Photo at rest, hover or click reveals the full bio text. Signature reveal component again, third and final use. **This section is a cobalt solid block.** Bio text for all three variants exists on v3.68 and ports over. This is not a writing task.
3. **Credibility strip**: the speaker kit link (canonical home) plus one testimonial:

   > "Huge thanks to Akil Bello for an inspiring and hilarious talk. We are united in the fight to ensure everyone can access an excellent, transformative college education."
   >
   > — Susan Parish, President, Mercy University

   Chosen for combining personality ("hilarious") with real title weight.

4. **Dynamic photo gallery, "The Real Work."** Already exists on v3.66 as an event-tagged carousel. Migrate and refine, do not rebuild. **Grid as the base layout, not a strip or infinite scroll.** Hover-peekaboo, click to expand into a larger view. Grid was chosen specifically because it lets visitors see and compare several images before choosing, which a scrolling strip works against. Whether the grid paginates or infinite-scrolls is a build-time call, make it and say which you picked.
5. Newsletter signup, then footer.

**Speaker kit contents:** bio variants and headshots, which About already holds naturally, plus a topics one-sheet, AV and tech requirements, and an intro script. The kit itself is complete and available offline. Build the section and the download links; ask Akil for the files.

**Two site-wide testimonial rules, apply everywhere:**

- **No verbatim-duplicate testimonials across pages.** The full set lives on Speaking; other pages sample a single different quote. A randomized rotating pool was considered and rejected: it adds real build complexity against the site's static philosophy and does not actually guarantee non-duplication. Hand-picked fixed assignment per page instead.
- **Quote tone by placement.** Landing's testimonial is ideas, rigor, and academic focus, with personality secondary. About's is personality and engagement forward, with credibility still present through the speaker's title.

**Design references logged for this page, use them, do not re-litigate them:** the Ashade photography WordPress theme for gallery layout; "Shuffling Effect in Pure CSS" and "Expand Image" from freefrontend.com as candidate mechanisms for the hover-peekaboo and click-to-expand behavior; `motivoweb.com/ruya` for text load-in animation and the general page feel.

## 4C. Contact: `/contact/`

**Structure:** header and subhead → single modest photo → four category cards → one form with a routing dropdown → closing line.

**Locked copy:**

> **Let's talk about what we can solve together.**
>
> Whether you're planning an event, writing a story, or trying to make sense of the admissions landscape — reach out. Response times vary, but every inquiry is read.

**The photo is conceptual, not another portrait of Akil.** The site already has plenty of those on About, in the gallery, and on Speaking. Instead: **a tin-can string telephone next to a smartphone showing "Call Failed."** It plays directly against the page's own closing line. Broken corporate phone technology against the oldest and most direct communication tool there is. **It sits in a cobalt solid block**, the third and last of the three page-level uses of that pattern. A school-telephone-in-background variant is worth exploring at design time but is not required.

**Four category cards.** Heading and description only. **No eyebrow labels.** The original v3.66 draft had small subtitles under each heading ("Keynotes, Workshops & Panels" and so on) and they were dropped for violating the no-eyebrow rule.

- **Speaking** — "I speak on testing, admissions, and educational access from K-12 through graduate school: for university audiences, policy organizations, corporate teams, and as a guest lecturer in college courses. Include your event date, location, and audience in your message."
- **Media** — "Available to journalists covering standardized testing, graduate school admissions, college access, and selective elementary and high school admission. I'm a background source as often as a named one, and I always try to tell you who else to speak to."
- **Research & Consulting** — "Universities, foundations, nonprofits, and policy organizations. If you're trying to understand what the data actually shows — or why your current approach isn't working — let's talk."
- **Pitch me an idea?** — "If it doesn't fit the categories above, use the form. I read everything."

Note: the consulting work is real and practiced, not aspirational. **Contact is intentionally its only visible mention on the site.** That is by design, not oversight. Do not surface it elsewhere.

**Form.** One form with light routing, not separate forms per category. Fields: First name, Last name, Email, Organization (optional), Type of inquiry (dropdown: Speaking, Media, Research or Consulting, Other), Message. Netlify Forms with honeypot spam protection.

**Closing line, kept as-is:**

> This form goes directly to me. No autoresponders. No CRM. Just email.

**Cut from the v3.66 draft, do not restore:** an "About these inquiries" section beginning "I'm one person with a day job" and its follow-up paragraph. It read as defensive and repeated credentials already established on About.

**Tutoring is not a category on this form** and never appears here. The tutoring page has its own lead capture.

**Build checklist for Batch 4:**

1. Speaking hero and positioning copy.
2. On the calendar strip.
3. Featured-talk reveal cluster.
4. Video and clips section.
5. Talk range, curated and skewed as specified.
6. Testimonials, canonical set.
7. Book a Talk CTA.
8. About headline and Three Bios cobalt block with reveal.
9. Credibility strip with speaker kit links.
10. The Real Work photo grid with click to expand.
11. Contact page, four category cards, cobalt photo block.
12. Contact form wired to Netlify Forms with honeypot.
13. Verify no testimonial appears verbatim on two pages.
14. Verify the reveal component is genuinely shared across Landing, Speaking, and About and not forked three ways.

**Stop and report.**

---

# BATCH 5: Book, Projects, Writing, 404, Resources

## 5A. Book: `/book/`

Reachable from About. No nav slot.

A shared link could land an agent on this page cold, so it carries its own credibility proof rather than assuming they have seen the rest of the site.

**Structure:** title and a single status line → hook paragraph → credibility strip → "Why I'm the one to write it" → newsletter CTA.

**Locked copy:**

> **Surviving Standardization**
> *Working title. Work in progress. Seeking representation.*
>
> An insider's guide to admissions testing's century of over-promising and under-delivering.
>
> For nearly a century, admission testing has presented the approximations of psychology as the precision of physics. I'm writing the book that exposes what tests actually measure, how the industry operates, and what a hundred years of deceptive precision has cost American students. The goal is to replace mythology and marketing with clarity that actually helps anyone who wants to understand why the system works the way it does.

> **Why I'm the one to write it**
>
> More than thirty years of testing expertise, gained working with the students who struggle most, not the ones who need the least. I've taken five versions of the SAT, three versions of the SHSAT, the GMAT, GRE, and two versions of the LSAT. I founded a test prep company built for underserved communities, served as Director of Equity and Access at The Princeton Review, Senior Director of Advocacy at FairTest, and now serve as Director of College Advising and FAFSA Completion Implementation at SUNY. I've worked inside every part of this system. I know where the arguments are data, and where they're marketing.

> Follow the newsletter →

**Credibility strip:** reuses the existing "As seen in" outlet list. Forbes, The Chronicle of Higher Education, Inside Higher Ed, The Washington Post, The New York Times, The Wall Street Journal, Vanity Fair, MSNBC, Netflix. Same logo assets already used elsewhere. No new asset production.

**The three-part status line is one line, not three eyebrows.** The v3.66 draft had "Work in progress" and "Working title" floating as separate labels above the title, plus "Why I'm the one to write it" as a subtitle above a restated header. Three competing altitude levels, all violating the no-eyebrow rule. It is collapsed into one status line and clean section headers. Do not undo that.

**Do not include** the Spencer Fellowship result, any agent-query history, or any blurbs or supporters. That is agent-packet material, not public-page material.

## 5B. Projects: `/projects/`

Four projects. Not three. The count is intentional.

- **Highly Rejective**: two-column: an embed or screenshot of the project page on one side, text and link on the other. Full treatment, launch-ready.
- **Crisis Construction**: same two-column treatment. Full treatment, launch-ready.
- **First Black Graduate**: one column of short text plus one column holding a data point. Links to the fuller writeup on Research.

  > 101 years. That's the average gap between when a college opened its doors and when it graduated its first Black student. Not a founding-era failure that got fixed early, decades of business as usual. I built this dataset to make that gap visible, school by school, because "when did they start including us" turns out to be a very answerable question. Institutions just don't like being asked.

  **Launch stat: 101 years.**

- **Testing History**: same treatment. Links to the dataset on Research.

  **Launch stat: two numbers shown together, SAT 22 and SHSAT 15.** Deliberately a pairing rather than a single figure: the most recognized test alongside surprising volume from a lesser-known one.

**At launch, each project shows one static, hand-picked data point.** Do not build a dropdown or selector. A live selector would imply dataset-engine infrastructure that does not exist yet. The post-launch version ships alongside the Research dataset tool, after GitHub setup.

Projects is where you get the headline. Research is where you dig. Keep that split.

**RAISE, DO NOT GUESS:** the spec flags one figure for a deliberate decision that has not been made. It is far more visceral and personal than the other aggregate stats, and it is not clear whether it belongs in the public headline rotation or inside the fuller writeup instead. It is in the First Black Graduate table in the spec doc. Ask Akil before using it anywhere public.

## 5C. Writing: `/writing/`

**Writing is one standalone, filterable archive.** It is not siloed inside each hub. Individual articles outperform the homepage for organic search, so each hub routes into a filtered view of this single archive rather than duplicating content.

Build:

- `/writing/`: the full archive, filterable.
- `/writing/research/`, `/writing/insights/`, `/writing/advice/`: **pre-built static filtered pages**, not client-side state.

**The decision here is explicit: pre-built filtered destination pages, not a persistent cross-site "lens" that follows a visitor via cookies or localStorage.** Fully static, matching current single-session search-driven traffic. Revisit only when the newsletter's subscriber base and repeat-visitor volume justify persistent state, plausibly three to five years out.

**Two independent taxonomy axes, not a hierarchy.** Every article has both:

- **Category:** Research, Insights, or Advice. Governs nav and routing. Multi-category is the **expected default, not the exception.** Most content qualifies for two of the three. The angle of a piece determines category, not the topic. Grade inflation as a topic can be Advice, Insights, or Research depending on framing.
- **Topics:** an open-ended tag layer, maximum three per item. Working set: Access, Policy, Guidance, Admissions, Testing, Equity, Grade Inflation, Rankings, Technology, Learning, Affordability, Quality. Topics are organizational, not restrictive. If content does not fit, add a Topic rather than force-fitting it. This same system feeds the Resources page's browse categories: one system, two use cases.
  - **"Merit" is retired as a Topic label.** Vague, loaded, with a history whose original meaning has been lost. Existing Merit-tagged items get retagged to something more specific. This retirement is about the tag only, not the word in prose.

**Editorial rule to enforce in the data:** an article can be tagged into multiple hubs, but must not be double-featured as a headline pick in more than one.

**Do not build tag pages, filter UI, or routing for:** Pillars (the intellectual framework), Framework terms, or any of Akil's coinages (Deceptive Precision, Accumulated Opportunity, Crisis Construction, Reputational Research, KRE, Hobo Jungles). These are prose-only brand vocabulary with no structural role. Pillars govern argument and voice, not navigation.

Article template: `/writing/[slug]/index.html`. Every article carries a "Read next" related-reading block and the newsletter CTA at the bottom. Post images get a click-to-enlarge lightbox in vanilla JS, no library.

## 5D. 404: `/404.html`

Status: **FULLY LOCKED.**

**Structure:** standard site header → headline and subhead → multiple-choice destination picker → standard site footer. This page gets normal chrome, not a stripped-down error treatment.

**Copy:**

> **404**
> You've reached a page that doesn't exist.
> This happens more than you'd think. Especially on education sites.
>
> Which of the following is where you were trying to get to instead?

- **A** — Home *(I just wanted to start over)* → `/`
- **B** — Writing *(I was looking for something to read)* → `/writing/`
- **C** — Research *(I wanted the receipts)* → `/research/`
- **D** — Speaking *(I wanted to book you)* → `/speaking/`

**Reuses the site's multiple-choice element, not the full Landing combination.** The v3.66 draft had a redundant eight-item mid-page nav list below four answer choices that were explanations rather than destinations, an indirect two-step. That is collapsed: the answer choices are the destinations. **Do not add a mid-page nav list back.** The header and footer already do that job.

## 5E. Resources: `/resources/`

**Status: not reconciled.** The baseline doc has a draft that has not been checked against the decisions made across the v4 planning process. It is top-level in the nav, so it cannot 404 at launch.

Build a minimal launch version only: the page shell, the headline, and browse categories driven by the Topic taxonomy from 5C, linking into the existing Sanity Library. Real entries confirmed for eventual inclusion: A Better Chance, Management Leadership for Tomorrow, Meredith G Consulting (meredithgconsulting.com), and getscoresignal.com.

**RAISE, DO NOT GUESS:** the Topic list needs a consolidation pass before it does its dual job of browse categories plus article tagging. The current twelve items are a working draft. The goal is simplicity and coherence: combine overlapping topics and eliminate redundancy rather than let the list grow. Do not run that pass yourself. Ask Akil, then build.

**Build checklist for Batch 5:**

1. Book page, all five sections.
2. Projects page, four projects, static launch stats.
3. Writing archive with filter UI.
4. Three pre-built filtered views.
5. Article template with Read next and newsletter CTA.
6. Lightbox, vanilla JS.
7. 404 page.
8. Resources launch shell.
9. Port `/crisis-construction/` from v3 with paths updated.
10. Port `/tutoring/` with `noindex`, out of nav, off the sitemap.
11. Generate the real `sitemap.xml`, excluding tutoring and the type specimen.

**Stop and report.**

---

# BATCH 6: Verification Pass

Do not skip this and do not self-certify. Produce an actual written report.

1. **Crawl every internal link.** Zero 404s. Every link ends in a trailing slash and resolves to a directory index.
2. **Verify no `.html` file serves at a path where the extensionless version also serves.** This is the specific v3 bug being fixed.
3. **Keyboard-only pass through the whole site.** Every reveal, dropdown, accordion, lightbox, and form. Every hover state has a focus equivalent.
4. **Screen-reader pass** on Landing, Advice, and Contact at minimum. Verify heading order, `aria-expanded` on the accordion and dropdown, and that decorative layers are `aria-hidden`.
5. **Reduced-motion pass.** Confirm every animation and transition is disabled, not merely shortened.
6. **Contrast audit.** All body text at 4.5:1, all large text at 3:1. Check white and dim text on cobalt specifically.
7. **Responsive pass** at 1440, 1024, 820, 560, and 375. Confirm no image shows white bars anywhere.
8. **Mobile Safari check on the hero scroll effect.** This is the one most likely to break.
9. **Copy audit.** Grep for em dashes outside quoted material. Grep for "X" used to mean Twitter. Grep for orange hex values. Grep for any eyebrow element sitting above a heading.
10. **Voice audit.** Confirm first person everywhere except formal bios. Confirm "Writing" not "Blog," "In Public" not "Media," "highly rejective" not "selective."
11. **Fact audit.** Confirm SUNY appears as the current role and FairTest never does. Confirm no book blurbs or supporters are presented as confirmed. Confirm no family details appear.
12. **Testimonial audit.** No verbatim duplicates across pages.
13. **Confirm the Spotlight Strip mechanism is live** and renders nothing with an empty array.
14. **Confirm GA is firing** and `robots.txt`, `sitemap.xml`, and `_redirects` are correct. Confirm tutoring is `noindex` and absent from the sitemap.
15. **Lighthouse** on Landing, Advice, and Speaking. Report the scores, do not just assert they are fine.

---

# Open Questions to Raise Before Launch

Do not resolve these yourself. Bring them to Akil.

1. **Typography is not locked.** Space Grotesk, Archivo, General Sans, and Neue Montreal all need a real side-by-side comparison. That is what the type specimen page is for.
2. **Kourtney Cockrell's title needs a display trim.** The full version is too long for the Landing testimonial and no trimmed version has been decided.
3. **The "36 colleges / 21%" First Black Graduate figure** sits in a different register from the other aggregate stats. It needs its own call on whether it goes public.
4. **The Topic taxonomy needs a consolidation pass.** Twelve items is a working draft, and the list feeds two systems.
5. **News vs. Noise scope at launch:** five pairings are content-ready. Whether all five ship or it trims to three is a scope call.
6. **The Contact page's locked copy contains an em dash** ("...admissions landscape — reach out"), and so does the Research & Consulting card ("...data actually shows — or why..."). This conflicts with the site-wide no-em-dash rule. Either the copy needs a light edit or the rule needs an explicit exception. Do not silently change locked copy to resolve it.
7. **First Black Graduate's launch form:** the spec says "a simpler static page (or stays a regular post)." Which one is not decided.
8. **GitHub is not set up**, and the dataset publishing layer depends on it. Not on the launch critical path, but flag it the moment a build decision needs a real link target.
9. **Which speaker kit file is the public download.** The folder holds five: a 4.8 MB `Akil_Bello_Speaker_Kit.pdf`, a duplicate of it, a `.docx` version, and two 17 MB "Speaker Book" PDFs. Do not pick one, and do not serve 17 MB without compressing it.
10. **Video hosting for the Speaking page.** Every source video is between 26 MB and 2.5 GB, and none of it can go in a static repo. YouTube or Vimeo embed is the likely answer, but that is Akil's call and it blocks the video section of Speaking.

---

## Where Everything Lives

Section 8 is the complete map of the client folder: the five files to read, what each supporting folder is good for, and what not to open. It is the first thing to check when you need an asset, a paper, an article body, or a piece of copy and are not sure where it lives.

---

## Build Log

Version format: `v4.0.<batch-increment>`. Increment on every build, per Section 1.

### v4.0.1 — Batch 1: Foundation and Global Chrome — 2026-08-19

Built: directory skeleton, `/assets/css/site.css` (tokens, type scale, layout, motion, reveal-card, cobalt-block, header/nav, spotlight strip, footer, newsletter), `/assets/js/site.js` (SPOTLIGHT config + renderer, nav dropdown + mobile menu, reveal-card touch handling), `/_template/index.html`, `/type-specimen/index.html`, `robots.txt`, `sitemap.xml` (stub), `_redirects` (carried forward from v3.69 + extended), logo/favicon/outlet-logo assets in `/assets/img/`.

Decisions made mechanically (technical, not content/voice — proceeded without asking per Section 1):
- Reused v3.69's live Google Analytics property (`G-YN2QV7H2JM`) rather than leaving GA unwired or inventing a placeholder ID. Flag if a fresh property is wanted instead.
- Touch-device reveal behavior (Section 4): implemented as expand-on-first-tap / navigate-on-second-tap, applied at ≤820px, since the spec said pick one and apply consistently.
- Newsletter form posts to Substack's public embed endpoint (`https://akilbello.substack.com/api/v1/free`), matching Substack's standard non-JS embed pattern. **Not verified against a live Substack embed snippet** — confirm this against the actual publication's embed code before launch, since a wrong endpoint would silently fail signups.
- Did not `git init` the repo. Batch 1's checklist doesn't call for it and no instruction specified a remote; raised as an open item below rather than assumed.

Open questions surfaced, not resolved: see "Stop and report" note after this batch, and the running "Open Questions to Raise Before Launch" list above (none of which block Batch 1).

Akil's calls on the two items raised after this batch:
- Footer's `/privacy/` and `/copyright/` links (not in Section 2's directory structure, not in any batch checklist): build minimal stub pages later. **Do in Batch 5**, alongside 404 and Resources, since that's where the other launch-shell/stub pages get built. Real policy copy still comes from Akil.
- Repo was not git-initialized during the build itself; initialized immediately after with an initial commit, per Akil's go-ahead.

### v4.0.2 — Batch 2: Landing Page — 2026-08-19

Built `/index.html`: cobalt hero (sticky-pinned photo layer + normal-flow overlay content, matching saragoldrickrab.com's actual technique, checked directly rather than guessed at) with the locked headline, ghosted "ABC" letterforms, the three reveal cards with locked copy, testimonial, newsletter. No footer link block, per the Landing exception.

Verified visually (headless browser, not just markup review) at 1440/375, mid-scroll, hover, keyboard-focus, and reduced-motion. Caught and fixed one real bug before it shipped: pre-expanded mobile cards made the hero overlay taller than 100vh, so `align-items: flex-end` pushed the overflow up behind the header. Mobile now gets a simpler static photo-banner-then-content layout instead of the absolute-overlap technique — no sticky pin at that scale, which wasn't buying much anyway on a screen that short.

Decisions made:
- **Hero photograph**: `NACAC talk IMG_6373.JPG` from `Media/Akil photos/Akil Presenting and Action Shots/` — standing, mid-gesture, 4000×2667 source (resized to 2000px, JPEG+WebP, both under 500KB). Picked over several other candidates, including a genuinely better "pointing" gesture shot (`Npea conference presentating.jpg`) that was only 561×561 in every copy of it found in the client folder, too low-res for a full-bleed hero. **This is an easy swap, not a locked choice** — flagged per Section 1, since photo selection wasn't explicitly delegated.
- **Kourtney Cockrell's testimonial byline**: rendered exactly as the locked copy gives it ("Kourtney Cockrell, JPMorgan Chase & Co.") rather than guessing at a trim of her full title, per the batch's own instruction to "render the short form... and ask Akil for the trim." Still open if he wants the fuller title shown instead.
- Spotlight Strip mount included on Landing (renders nothing today, array is empty) — Section 5.2's language on whether the strip appears on Landing itself is genuinely ambiguous, and an empty array makes the question moot until Akil actually populates it.

### v4.0.3 — Batch 3: The Three Hubs — 2026-08-19

Built `/research/`, `/research/library/`, `/insights/`, `/advice/`, and `/in-public/` (the last one specified in Batch 3 but living under Insights, so building it now rather than leaving Insights' "In Public" teaser linking nowhere). Read the required `v4-content-and-page-specs_aug_9.md` (hadn't gotten to it before Batch 2 shipped; Batch 1/2 didn't need it, this batch did). Pulled real source material rather than inventing any: the five papers' real descriptions/authors from v3.69's `research.html`, meta descriptions from the actual ported posts, and — for In Public — parsed `Akil Bello - Media Presentations Articles cv.xlsx` directly (openpyxl) into 45 real, deduplicated entries (22 bylined articles, 17 media citations with real quotes, 9 video/TV appearances, 1 podcast), rather than hand-transcribing or sampling a few.

For the News vs. Noise pairings, verified every noise-source citation by web search before using it — caught and fixed two fabricated-URL mistakes this way (an invented Inside Higher Ed URL for the AI-essays piece, and a bare-domain guess for Jon Boeckenstedt's blog) before they shipped. Researched and cited real HBCU-affordability data (Century Foundation, CollegeVine) for the one pairing the spec explicitly flagged as needing data at build time.

Decisions made:
- **Only 4 News vs. Noise pairings, not 5.** The spec's own 5th item is explicitly reassigned to Insights (the Leonhardt/IHE pairing), so only 4 real Advice pairings exist. Built all 4 — a legitimate "build-time scope" call per the spec, and all 4 had strong enough real sourcing for self-contained 2–3 paragraph News sides. Nothing was cut for insufficient data.
- **Noise side uses a styled "clipping" card (real venue, real headline, real date), not a live screenshot.** Reliably and legally screenshotting current, often-paywalled third-party news pages (NYT, Forbes, CNN) wasn't something to do safely inside this build. Flagging this as a deviation from the literal spec — real screenshots can replace these later if Akil wants to capture them himself.
- **Past Issues accordion**: built the mechanism (shared `.accordion`/`.accordion-item` CSS already exists in site.css from the component work), but did not fabricate fake historical issues to populate it — this is genuinely the first issue of a new site. The section says so plainly instead of pretending to have a past.
- **In Public**: excluded ~4 rows from the CV spreadsheet that were self-citations of Akil's own site (mislabeled as "citations" in the source sheet) rather than genuine third-party coverage, since that's not what "In Public" is for.
- **Insights' featured-article image** and the **Advice News-side "Noise" screenshots**: no image assets exist for either (Chronicle's own commissioned art, and third-party news sites respectively), so both use typographic/clipping treatments instead of a missing or fabricated photo.
- Corrected one date: the Washington Post "Two key questions..." piece is tagged 2022 in the CV spreadsheet but its own URL embeds `/2019/05/01/`; used 2019 as the more reliable evidence. Mechanical fix, not a content call.

Nothing to flag per the batch's own "stop and report" instruction — no pairing had insufficient supporting data once real sources were found. Multiple pages forward-reference pages that don't exist until Batch 5 (`/writing/[slug]/`, `/crisis-construction/`, `/tutoring/`) and Batch 5 shell pages (`/resources/`) — expected 404s until then, not bugs.

---
> Source: [anchordigitalco/akilbello](https://github.com/anchordigitalco/akilbello) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
