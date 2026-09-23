## casa-brigante-website-v2-0-3

> - MENU CARD (music.html roster): was cutting off Elara Bloom on the right. vanta_card.jpg = full-width looking-at-each-other group; roster img now uses object-position center 30% so all four fit.

## VANTA fixes (menu card + hero)
- MENU CARD (music.html roster): was cutting off Elara Bloom on the right. vanta_card.jpg = full-width looking-at-each-other group; roster img now uses object-position center 30% so all four fit.
- HERO (artist-vanta.html): swapped from the looking-at-each-other shot (weird as a hero) to the FORWARD-FACING group (2e272768), with the baked "VANTA" title cropped off the top. Focal center 38%. The looking-at-each-other shot stays as the menu card only, per founder.

## VANTA finalized (bug fix + new photos)
- BUG FIXED: vanta_hero.jpg had been overwritten with an orange/Solana image (that's why the VANTA hero and Music-roster thumb showed Solana). Replaced with the correct clean gold-hall VANTA group shot (no text). Roster thumb auto-fixed (points at vanta_hero.jpg).
- New individual member portraits set from founder's gold-hall solo shots (Aria Vale, Juno Knox, Nyla Starr, Elara Bloom), cropped to the person, name-text excluded. Theme unchanged (gold-on-black gothic luxury), just finalized with better photos.
- VANTA is now DONE. Do not touch. Next: MOONREALM finalize, then Solana finalize (per founder's one-act-at-a-time approach — the right call).

## DEPARTMENT HERO STANDARD (the house formula, founder-ratified)
The MOONREALM/artist hero formula is now the standard for EVERY department front page. Reusable .dept-hero component (in styles.css): full-bleed cinematic image, top+bottom vignette, floating serif title over the lower third, uppercase gold eyebrow, italic one-line tagline with a gold hairline. Set the image per page via inline --dept-img.
- BOOKS uses it now: books-hero.jpg = the five covers arranged as a fanned shelf in warm gold light with edge-lighting and vignette (built from the real cover files; regenerate if covers change, especially when Bury Me in Tucson's real cover exists).
- ROLL-OUT PENDING (apply .dept-hero to each, founder to approve image per department): HAWS, Vault 12, Audio Dramas, Plug & Play, The House, Inquiries. Each needs a hero image: either its own key art or the estate emblem as fallback.
- Books hero image is COMPOSITE-GENERATED, not a photo; when the founder supplies real product/lifestyle photography it can replace books-hero.jpg (CSS already points at it).

## Artist heroes elevated (press-kit -> film-poster)
- All three artist heroes now use CLEAN, TEXT-FREE cinematic key art (not press-kit sheets): vanta_hero.jpg (gothic cathedral group), moonrealm_hero.jpg (gold eclipse duo, bottom text cropped), solana_hero.jpg (golden-hour rust, baked logo cropped out). Music roster thumbnails updated to match.
- Hero treatment: 92vh, cinematic top+bottom vignette, poster-credit hairline under the subtitle, stronger title text-shadow. Each keeps its own accent + motion. Old *_anchor.jpg press sheets remain only for internal reference, not shown as backgrounds.
- If founder supplies even cleaner per-artist key art later, swap the *_hero.jpg files; CSS already points at them.

## Artist parity pass
All three artist pages now share the same section rhythm (hero, statement, world, members/song, credits) and each ends its credits with a "More from the Casa Brigante roster" link back to music.html. VANTA and MOONREALM layouts/colors/motion were NOT otherwise changed (they were founder-approved); only the universe cross-link was added for consistency with Solana.

## SOLANA REYES room built (3rd artist — completes the novel<->music bridge, the proof of concept)
- artist-solana-reyes.html: concrete/rust/paper aesthetic (property accent RUST #b8552e on concrete-gray/paper; gold stays institutional). Near-still motion (no drift, no float) = her signature: patience, confinement. solana.css holds styles.
- Sections: hero ("The voice prison couldn't silence."), Artist Statement ("Music became the one place they couldn't lock me out."), Her World, The Song ("ghost" + discography: ghost/Letters to Sergio/ADC Anthem/Five Life Sentences/Until Then), From the Novel (A Song She Wrote, links to book-a-song-she-wrote.html), Credits.
- SENSITIVE-CONTENT HANDLING (deliberate, per founder + safety): Solana's canon includes incarceration for the livestream killings and 5 life sentences. The public page leads with MUSIC and WRITING, uses the prison setting as atmosphere and mythology, and does NOT dramatize, detail, or glorify the violence. The full story lives in the novel; the page points there. Keep it this way.
- THE BRIDGE IS LIVE BOTH WAYS: Solana artist page -> A Song She Wrote novel page, and the novel is her canon source. This is the interconnected-IP proof the whole Creative Experience Standard was built to validate.
- All THREE artists now live in the Music roster, each a distinct world: VANTA (gold/cold/still), MOONREALM (purple/nocturnal/drift), Solana (rust/concrete/still-but-human).
- PENDING: Solana streaming links when "ghost" is live; confirm anchor/portrait image choices; her canon is heavy (Afro-Latina, Mesa AZ, brother Sergio OD) — never invent beyond the bible.

## MOONREALM room built (2nd artist, proves the multi-room system)
- artist-moon-realm.html = full nocturnal world. Anchor = gold eclipse masked-duo key art (moonrealm_anchor.jpg). Members: The Sovereign (Architect, producer/silent/precise) + The Oracle (Voice, performer/transmission/mask-changes). Statement "We don't follow the light. We create our own." World, Discography (Eclipse/Oblivion/Lunar Codes/Enter the Realm), Credits.
- ATMOSPHERE per Creative Experience Standard: gold stays institutional; MOONREALM's property accent = PURPLE (#6c5ce7 / lilac #8b7fd4) on deep violet-black. MOTION SIGNATURE = slow 22s ambient drift on the hero (vs VANTA's stillness). moonrealm.css holds room styles. This is the proof that two rooms feel completely different while sharing the house frame.
- Credits: Origin Tucson AZ, Label The Commencement Music, Management LA FINCA MANAGEMENT (note: MOONREALM is managed by La Finca, not Casa Brigante directly — per the story bible). The masked duo concept: The Sovereign + The Oracle are CHARACTERS; the humans have a parallel unmasked lifestyle brand. Never unmask them as "real names" on the public site.
- Music roster now has VANTA + MOONREALM live; Solana Reyes still "Arriving soon".
- PENDING: MOONREALM social/streaming URLs (@moonrealmsound known); Eclipse links when live; Solana anchor image to build her room.

## MUSIC ROOM built (VANTA first, per Creative Experience Standard)
- music.html = "The Artists of Casa Brigantè" roster: VANTA live (anchor image, links to artist-vanta.html); MOONREALM + Solana Reyes shown as "Arriving soon" placeholders (Identity before Inventory: no empty pages).
- artist-vanta.html = full world: cinematic hero (anchor = clean seated group landscape, canonical), Statement ("We don't follow the light. We create our own."), The World, The Members (all four: Aria Vale/The Voice, Juno Knox/The Architect, Nyla Starr/The Spark, Elara Bloom/The Dreamer), Music ("Eclipse" coming soon), Credits (Created by The Commencement Music / Produced by Moon Realm / Management Casa Brigante / Publishing Excelsa Cornua).
- NYLA STARR spelled with two R's per founder. Member copy smoothed from the press kit (removed stiff "LEADER · VOCALIST" stacking; role + one discipline line + prose bio instead).
- VANTA accent = gold institutional + black atmosphere + restrained motion (no drift), per the Standard. vanta.css holds room-specific styles.
- The Commencement Music appears ONLY in credits (infrastructure), never as a nav/brand. Consistent with the invisibility rule.
- PENDING from founder: real portrait for MOONREALM/Solana anchors before their pages build; exact social URLs (@vantaofficial / vanta.world known, per-platform links needed); "Eclipse" streaming links when live; confirm Elara portrait crop quality (cropped from press sheet, lower-res than the other three).

# ============================================================
# CASA BRIGANTÈ CREATIVE EXPERIENCE STANDARD v1.0
# The governing standard for every creative property the house
# presents: artists, novels, productions, and future disciplines.
# Ratified by the founder. Rules exist before rooms.
# ============================================================

## THE FOUNDING SENTENCE
Casa Brigantè presents WORLDS, not products. Products may be purchased; worlds are experienced.
This is the reason the book pages read as exhibits instead of storefronts, and it governs Music, Audio Dramas, and every division that follows.

## TWO IDENTITIES, ALWAYS SIMULTANEOUS
Every property balances a constant institutional identity with a variable property identity.

### 1. Institutional Identity (CONSTANT — never changes across any property)
Header · navigation · footer · typography · grid · spacing · editorial tone · GOLD institutional accents · estate watermark · CTA pattern · component behavior.
Visitors must always know they are inside Casa Brigantè.
GOLD RULE: gold is the institutional constant, the equivalent of Penguin orange / Criterion black / Ferrari red. Gold owns the institutional LAYER specifically (header, CB estate mark, dividers, buy/listen actions). It is never reassigned to a property. Lock it.

### 2. Property Identity (VARIABLE — each property owns these, never interchangeable)
- ONE defining visual ANCHOR (see Anchor Principle)
- ONE atmospheric accent color (secondary to gold; owns mood/texture only, never the institutional furniture)
- ONE motion language (see Motion Is Identity)
- ONE environmental texture
- ONE narrative voice
- ONE emotional identity

## MOTION IS IDENTITY
Motion is not decoration. Motion communicates character. Someone should recognize the property even if every page were grayscale.
- VANTA: deliberate, restrained transitions; minimal animation; precision and control.
- MOONREALM: slow ambient drift; light bloom; lunar atmosphere.
- Solana Reyes: almost imperceptible movement; film grain; dust; paper texture; long pauses.
All motion respects prefers-reduced-motion.

## THE ANCHOR PRINCIPLE
Every creative property must have ONE definitive image, chosen with book-cover-level care. It is the canonical visual reference for website hero, press kit, streaming platforms, social, marketing, and internal docs. Everything else may evolve; the anchor remains. It is the artist's equivalent of a book cover.

## IDENTITY BEFORE INVENTORY
A page never exists merely because content might arrive later. A property launches ONLY when it possesses: a complete narrative identity, a canonical visual anchor, a finished world, and meaningful content. Additional releases enrich the experience; they do not define it. Build fully, one property at a time. Never launch empty rooms.

## SOLANA FIRST (rationale)
Solana Reyes is the PROOF OF CONCEPT, not merely the most-documented artist. She demonstrates that Casa Brigantè builds interconnected intellectual property, not a media display: a visitor moves from A Song She Wrote (novel) to Solana Reyes (artist) to future recordings/performances/productions without leaving one narrative universe. If that experience works, the architecture is validated for every future original property. Build the Solana novel<->artist bridge in both directions.

## WHY THIS STANDARD EXISTS (the house method)
Rules before rooms, always: constitution before company, design system before components, component library before pages, governance before operations, brand standards before logos. The Music Room and every future division follow the same order.

## Naming ruling (founder-ratified)
- Navigation stays LITERAL for legibility: Books, Music, Audio Dramas, HAWS, Vault 12, Plug & Play. Evocative names are page HEADINGS, not nav labels (Books page opens "The Collection", etc.). Do not rename nav items to Library/Listening Room/The Stage.
- ONE exception: "About" is renamed "The House" everywhere (nav + page eyebrow), href stays about.html. It reinforces the Casa/house identity without losing clarity. This is the only evocative nav label.
- MUSIC ROOM plan (next build): label roster, not a streaming clone. Framing: "Casa Brigantè develops recording artists." Music index = artist cards (VANTA, MOONREALM, Solana Reyes) -> each opens a distinct artist page mirroring the book-detail structure (hero image, statement, the world, gallery, music/videos when live, upcoming). The Commencement Music entity stays invisible (About only), same rule as Excelsa. CRITICAL cross-link: Solana Reyes bridges novel <-> artist (A Song She Wrote <-> Solana music page <-> back to novel); build both directions. Each artist page must feel like its own world via a per-artist accent color from their brand board (VANTA gold-on-black, MOONREALM lunar purple, Solana concrete-gray/gold) while keeping the shared house frame. Build 2-3 artists FULLY rather than all thinly. Founder bringing key image + live-status per artist.

## Book pages: edition-truth + preview refinements (founder-ratified)
- IMPRINT RULING (founder's call, better than the earlier plan): keep the site in agreement with Amazon and the copyright page. Live books show "Published by TFG Publications" + "Rights administered by Excelsa Cornua Rights Group" (two truths). Bury Me in Tucson shows "Published by Excelsa Cornua Rights Group". When the four are reprinted under Excelsa with new ISBNs, swap the first line to Excelsa and update ISBN + Amazon links together. No historical mismatch.
- Preview section renamed "Read Chapter One" -> "Preview the Novel". Heading now: eyebrow "From the opening pages", then italic "Chapter One · <title>". Makes clear it is an excerpt, not the full chapter.
- Post-excerpt CTA added on the four live titles: "Continue <character>'s story." + Paperback/Kindle buttons (peak-emotion buy moment). Tucson gets a "arrives soon" note instead.

## Book depth added from VERIFIED source files (founder-supplied content package)
Each live book page now carries: two spoiler-free pull quotes (exact wording from the manuscripts), a real metadata block (reading time, pages, ISBN, first-published year, imprint), and a "Read Chapter One" preview (the short promotional sample from the content package, with a drop cap and a note that it is a preview only).
- Imprint per file: October's Change / When Love Was Enough / A Song She Wrote / Loyal 2 the Lie = TFG Publications (as printed in those books). Bury Me in Tucson = Excelsa Cornua Rights Group. NOTE: this conflicts with the earlier site-wide claim that all titles are "published under Excelsa Cornua." Books page "Built to Last" line says Excelsa manages rights/stewardship, which can remain true even if the printed imprint differs, but FOUNDER SHOULD CONFIRM how he wants imprint vs rights-holder presented.
- Bury Me in Tucson: NO ISBN and NO page count shown (manuscript had only placeholder ISBN 9 / 1 and a bogus 1-page DOCX count). Shows "On publication" for both. Never publish the placeholder ISBNs.
- Reading times shown as estimates (250 wpm), per the content package method.
- STILL NOT PROVIDED (do not invent): author's notes / Behind the Story, character rosters (Meet the Players), social proof / sales numbers. The Bury Me in Tucson dedication to India Simpson exists but is a DEDICATION; only use as "Behind the Book" if the founder explicitly approves.
- Rights note from the package: every excerpt is a short promotional sample under all-rights-reserved; founder controls the rights, so use is at his direction.

## Book detail enrichment (round 1, buildable items done)
DONE without new inputs: cover enlarged ~17% with a gentle 6s float (reduced-motion respected); gold glow on the primary purchase button; button sublabels (Paperback/Amazon, Kindle Edition/Amazon Kindle); hierarchical publisher line (Published by / Status); collectible metadata block (Collection, Volume I-V by publication order, Genre, First Edition, Setting); "The Casa Brigantè Collection" reading-order strip on every book page with the current title highlighted and Bury Me in Tucson marked Coming Soon.
NEEDS FOUNDER INPUT before building (do not invent any of these):
  - Author's Notes / "Behind the Story" text per book
  - Character roster ("Meet the Players") per book, names + one-line non-spoiler intros
  - 3-4 spoiler-free pull quotes per book, taken from the actual manuscripts
  - Real metadata: page count, publication date, ISBN, estimated reading time
  - "Read Chapter One" excerpt files
  - Any real social proof (copies sold, reviews) — never fabricate numbers
  - 3D paperback mockup renders if wanted (currently flat covers)
FLAG: parallax on the fixed watermark deferred; background-attachment:fixed already gives a parallax-like feel and true JS parallax risks the mobile-performance rule.

## ✅ LOCKED PAGES (founder sign-off, do not modify without a direct quote from the founder)
- HOMEPAGE: complete and locked.
- BOOKS WING: books.html + all five book detail pages, complete and locked. Compact horizontal catalog, deep detail pages, Paperback + Kindle buttons, related titles, "Built to Last" closing every page, centered estate watermark.
Any future edit to these pages requires the founder's explicit instruction. Only a genuine bug reopens them.
OPEN ROOMS (next work): Music, Audio Dramas, HAWS, Vault 12, Plug & Play, About, Inquiries.
STILL PENDING: founder to verify the four books' Paperback/Kindle links; Bury Me in Tucson cover (flagship blocker); custom domain DNS.

## Books = compact catalog + deep detail pages (v2.2)
- books.html: horizontal rows (96px cover, title, author + genre, one-line hook, "View Book" link), full-width dividers. Fast and elegant; no purchase buttons on the catalog itself.
- Each row opens book-<slug>.html: large cover, full synopsis, Paperback + Kindle Edition buttons, publication line, "More from the Collection" related grid. Bury Me in Tucson detail page shows "Coming Soon", no links.
- "Built to Last" closing editorial appears at the foot of the Books index AND at the end of every individual book page (whichever door a reader enters, they leave on the imprint's permanence note). Tightened ~35%: two short paragraphs, Excelsa reduced to one fine closing line, pulled near the last row.
- SITE WATERMARK: full feathered estate as a FIXED body::before pseudo-element, opacity .05, CENTERED (left 50%, translateX -50%) matching the homepage hero, hidden on mobile. Replaces the old cropped footer image (footer::after disabled).
- FOOTER cleaned: Correspondence and Plug & Play removed from footer nav (still in header where active).

## LINK AUDIT NEEDED (founder must verify)
Three overlapping link lists have been provided across sessions. Current wiring: Paperback buttons use B0FRLJ2KRD/B0FJFJC337/B0FGYDTPYT/B0FJ7JC5HQ; Kindle buttons use B0FRF69VDR/B0FHGJVSLS/B0FGX9TLYT/B0FJ6RBG7N. Founder must click each and confirm edition + title match, because these lists have conflicted between sessions.

## Books page = "The Collection" (v2.1, founder-ratified)
- The catalog reads as a library, not a store: cover + title + author + register line + actions, on a bordered editorial list.
- TWO EDITIONS PER TITLE, both wired: Paperback (primary button) and eBook/Kindle (ghost button). Paperback ASINs: Loyal B0FRLJ2KRD, Song B0FJFJC337, WLWE B0FGYDTPYT, October B0FJ7JC5HQ. Kindle ASINs retained from launch. Bury Me in Tucson shows "Coming Soon", NO link, until publication.
- Closing "Built to Last" editorial replaces the Excelsa logo block: text only, no button, no logo, no CTA. Reads like the last page of a book.
- SITEWIDE FIXES this round: Chronicle removed from all nav; social links removed from all headers/footers (keep visitors inside); Privacy + Terms in every footer. Footer nav locked: Books, Music, Audio Dramas, HAWS, Vault 12, Plug & Play, About, Inquiries, Privacy, Terms.

## Header mark color FINAL
Header estate icon retinted to the CENTER emblem's cream-gold duotone (shadow #12110f, mid #b79860, highlight #e0d6be), not the harder nav gold. It now reads the same cream-on-dark as the hero illustration. This matches; do not change it again.

## Hero 1.4
- Artwork raised 30px (top:45). Headline max trimmed to 4.6rem to buy that height while keeping the guardians' faces clear; the two trade against each other (see measured geometry below).
- Result: the headline crosses the estate, house and balcony, and the guardians sit in clear space directly beneath it.
- Dog highlight lift reduced from +28% to +12%: present, visible, no longer competing for attention. Founder's intent: "there, but not taking too much attention."
- Header mark recolored to sit at the same brightness and contrast as the rest of the site (charcoal to #c9a24b to #dcbd72, no ivory blowout).

## Hero supporting copy FINAL (founder-set)
"Casa Brigantè is where stories, sound, style, and entertainment come to life. Crafted with purpose. Created without compromise. Built to endure."
Locked. Do not alter. Covered by the founder's standing hero-only lexicon exception.

## Hero 1.3: the geometry, measured (read before ANY hero request)
Measured positions inside CB_Hero_Full_v2.png (1320x755):
  balcony 37-40% · mastiff face 38-50% · cream dog face 53-64% · dog bodies end 83% · art ends 100%
CONSEQUENCE: the guardians' faces begin at 38% of the artwork height. The headline bottom must always finish ABOVE that line. Therefore the art cannot be raised without reducing headline height, and the headline cannot be enlarged without lowering the art. They trade against each other. Any future request to "raise the image" must be answered by checking this math first.
- This revision: headline max size reduced 6.4rem to 5.2rem, artwork raised 35px to top:75px. Faces clear the type with ~13px of margin; paragraph still begins below the artwork.
- Founder's rule, absolute: THE DOGS' FACES ARE NEVER COVERED. Words may sit above or below them, never across them.

## HOMEPAGE v1.0 — SIGNED OFF
Final adjustment made: hero illustration raised 30px so the word "transcend" sits across the estate's second-floor balcony, both guardians fully visible beneath the headline. Scale, crop, opacity, dogs, logo, and feathering all UNCHANGED and now final.
NOTHING ON THE HOMEPAGE MOVES AGAIN. No repositioning, no opacity, no logo tweaks, no feathering, no dog adjustments. Only a genuine bug reopens this page, and any request to change it must quote the founder directly.
Remaining work, in priority order: Books, Music, Audio Dramas, HAWS, Vault 12, Plug & Play.

## Hero 1.2.1 (final spacing rule)
- Artwork raised 20px so the guardians sit directly beneath "Ours transcend time."
- STANDING RULE: the estate illustration must END before the supporting copy BEGINS. The artwork supports the headline only. It may never touch or sit behind the paragraph or the CTA. Paragraph margin-top is set per breakpoint to guarantee this clearance.
- LOGO IS FINISHED. The top-left lockup is correct and is not to be changed again.

## HERO: THREE-ZONE COMPOSITION (acceptance-tested, do not redesign)
Structural rule that ends all hero drift: the hero is THREE VERTICAL ZONES, not text layered over art.
  Zone 1 Headline (clear of all artwork except faint roof/upper estate)
  Zone 2 Guardians (protected clear area; NO typography may enter it)
  Zone 3 Supporting paragraph + CTA (below the dogs, no overlap)
- Asset: CB_Hero_Full_v2.png. The COMPLETE estate illustration, uncropped on all four sides, original 1320:755 aspect, background-size contain. Never crop it again.
- Only the outer border band is feathered; the interior, including both guardians, is never masked, faded, or darkened.
- Placement is achieved by (a) .hero-art-full top offset and (b) .lede margin-top, which pushes the copy below the dogs. Both are set per breakpoint.
- ACCEPTANCE TEST for any future change: if a letter touches either dog's face, it FAILS. If any part of the estate is cut off, it FAILS.
- Header, navigation, logo, headline, paragraph, and CTA were NOT touched in this revision.

## HERO: FINAL LOCK. DO NOT TOUCH AGAIN.
Root cause, finally addressed: the artwork was ~11% TOO TALL for the hero. That is why the roof and the guardians could never both land correctly. SCALE FIRST, POSITION SECOND. Never again respond to a hero note by moving the image.
- CB_Hero_Composition_v1.png rebuilt shorter (sky above the tower and flourish below the paws trimmed), aspect 977:604, then reduced ~9% in CSS.
- Anchored to the typography, top -25px: roof behind the top of the headline, doorway mid-block, guardians behind the closing line, nothing above the headline or below the paragraph.
- All four edges dissolved aggressively with a blurred alpha; no rectangular boundary exists.
- Opacity unchanged. Guardians selectively lifted so their faces read; the rest of the illustration untouched.
- Header mark enlarged to 140px and recolored to the true house palette (charcoal shadows, #c9a24b gold, ivory highlights).
- THE HOMEPAGE IS FINISHED. Every remaining hour belongs to Books, Music, Audio Dramas, HAWS, Vault 12, and Plug & Play. Only an actual bug reopens this page.

## Header 1.1.2
- Estate icon enlarged ~23% (96px to 118px). It reads as a publisher's mark now, not decoration. Wordmark size UNCHANGED.
- Spacing between icon and wordmark widened to 1.5rem.
- Whole logo group nudged up 3px (optical alignment; mathematically centered read as low to the eye).
- Header height grown to hold the larger mark without crowding.

## Hero composition 1.1.1 (no further composition changes)
- Illustration raised 25px, scaled down 5%.
- Side feathering deepened significantly (separable falloff, horizontal solid zone only 30% of half-width, plus an 18px gaussian on the alpha). Smoke, not rectangle.
- The GUARDIANS ONLY brightened a further 30% through a soft elliptical mask so their eyes and faces read; the rest of the illustration untouched.
- FOUNDER'S INSIGHT, now standing doctrine: the house creates the setting, the typography creates the philosophy, THE DOGS CREATE THE EMOTION. The dogs are the hero of the hero. Never let them wash out, never let them disappear, never redraw them as anything but the founder's real guardians.

## HERO COMPOSITION LOCK (the breakthrough)
The illustration is a LOGO being used as a BACKGROUND COMPOSITION. Those are two different jobs. Never drop the original logo in as a watermark again.
- Asset: CB_Hero_Composition_v1.png. Purpose-built hero crop (tighter framing, less empty border), brightened ~55% so the guardians' faces read, edges feathered to nothing. Brightness was raised INSTEAD of opacity.
- The art is anchored to the TYPOGRAPHY (.hero-art lives inside .hero-center), not to the browser window. It moves with the text at every screen size. Do not re-anchor it to the section.
- Scaled ~10% smaller than the old watermark. The typography is the focal point; the illustration is atmosphere.
- Intended composition: roof peak behind "creations", doorway behind "for the moment", guardians' heads behind "Ours transcend time", staircase terminating near the bottom of the paragraph.
- The ONLY two dials: .hero-art top (vertical alignment to the type) and width (scale). Text never moves.

## Homepage 1.0.3 (final art direction)
- Estate raised a further 80px (background-position center -125px). Text position UNCHANGED; the founder was explicit: raise the estate, never lower the text.
- Emblem enlarged 5% (872px) now that it sits higher and is no longer cropped by the hero box.
- Composition order, per the founder's diagram: House, then Dogs, then Headline, then Paragraph, then Button.
- Standing principle: the illustration supports the typography. It is atmosphere, not the focal point. The dogs must never compete with the headline.
- The ONLY dial for further tuning: .hero-watermark background-position Y value. Never move .hero-center.

## Homepage 1.0.2
- Estate raised 45px (background-position center -45px) so the tower sits behind the first headline line and the guardians settle near the last.
- Header estate icon 88px to 96px (+9%). Wordmark size unchanged; the icon now leads.
- Navigation item spacing widened to 1.15rem each side of the diamonds.
- Founder's standing note: the site runs three scales (huge, medium, tiny) and that rhythm is intentional. Do not fill empty space.

## Homepage 1.0.1 (founder-authorized unseal: "can we unseal to do that please" re: dogs' faces)
- Hero recomposed: emblem anchored center-top with both guardians' faces in clear space; headline set beneath them, overlapping only the estate steps and grounds. This is the permanent composition: faces above, philosophy below.
- Tuning dial if the founder wants adjustment: .hero-center margin-top clamp (moves text) and .hero-watermark background-size (scales art). Touch nothing else.
- RE-SEALED after this change.

## HOMEPAGE VERSION 1.0 — SEALED (final revision executed)
- Estate raised further (background-position center 10%); both guardians fully visible beneath the headline.
- Edge feather increased ~12%; the illustration melts completely into the charcoal. Opacity untouched.
- Navigation logo: the horizontal estate lockup as built (feathered estate icon + live wordmark). The wordmark stays live type for crispness; this IS the horizontal lockup rendered for the web.
- Logo moved 14px off the browser edge. Luxury breathes.
- Body line-height raised to 1.8 (1.85 in editorial passages).
- FOUNDER'S WORDS: everything else approved, no additional redesigns, Homepage Version 1.0 is LOCKED. Any future homepage request must quote the founder directly before a single pixel moves.

## v3.0.1 final touches
- Hero emblem raised (background-position center 16%, enlarged) so both guardians' faces clear the headline; the reclining dog is no longer hidden behind "transcend." If the founder asks for more/less, adjust ONLY the background-position percentage.
- Header icon recolored to the site's gold duotone (CB_Header_Icon_v2_gold.png): shadows to charcoal, light to --gold-soft #dcbd72, matching the palette exactly. Original warm-sepia version retained in assets.

## FINAL LOCK (COO decision, founder-ratified)
THE HOMEPAGE IS LOCKED. Another revision produces less value than a day on the inner pages. Do not touch the homepage without the founder's explicit direction.
- Header: founder's new horizontal lockup art. Icon = estate + guardians + arched CASA BRIGANTÈ (CB_Header_Icon_v2.png, feathered, ~88px) + live wordmark. No stacked version, no CB, no full crest, no tagline.
- Hero copy FINAL: "Casa Brigantè is where fiction, music, audio dramas, apparel, and original entertainment experiences are developed with intention. Crafted with purpose. Told without compromise. Built to endure." The Commencement removed from the hero (footer + About only). The dash exception is MOOT; that sentence no longer exists. Zero dashes sitewide again.
- Emblem: opacity unchanged, edge dissolve strengthened; no visible rectangle ever.
- Nav hover: gold underline, instant, elegant; no animation beyond a color ease.
- NAV AMENDMENT: VAULT 12 added after HAWS (Jordan customs, New Era fitteds, numbered pieces; vault-12.html; counter link). Nav otherwise locked.
- Every page opens the same way: title, one sentence, generous space, then content. This is the sitewide page-opening pattern.
- Footer: rich, with Privacy and Terms links. privacy.html and terms.html are plain-English placeholders written honestly to current site behavior; FLAG: attorney review before any commerce runs on-site.
- Books/Music use real covers and real artist branding, never placeholders.
- Standing creative rule: restraint, confidence, permanence, editorial sophistication over decoration. When in doubt, remove instead of add.

## Round 5.1: HOMEPAGE COMPLETE (founder's words: perfect)
- Header lockup FINAL: horizontal feathered estate-and-guardians emblem (CB_Header_Emblem_v1.png, ~78px tall) + "Casa Brigantè" wordmark. NO tagline. "The Commencement" appears ONLY in the footer and About; nowhere else on any page, header included.
- Hero emblem raised to 28% desktop / 18% mobile.
- THE HOMEPAGE IS LOCKED AND COMPLETE. No further homepage changes without the founder's explicit direction. All remaining work is the inner pages.

## Round 5: header/nav polish + DASH RULING (founder's own words)
- DASH RULING: the founder granted a dash in the hero support copy but softened it to "a regular dash instead of the comma." An en dash with spaces ("The Commencement – where") is in place. This is a SINGLE-LOCATION exception. Em dashes remain banned everywhere, and no other dashes may be introduced anywhere else without the founder's explicit words.
- Header lockup: Option 4 from the founder's design board: estate icon left + stacked wordmark + tagline "The Commencement Creative House" (lexicon amendment covers "Creative" in this lockup tagline only). Header height ~100px.
- Navigation: ALL CAPS, .1em letter-spacing, weight 500, tiny gold diamond separators, gold underline-fade hover. This supersedes the sentence-case nav rule in the Editorial Style Guide for navigation only.
- Hero ground is charcoal #0b0b0b; emblem at ~23% desktop, feathered, present but subordinate to type.
- CTA: gold with subtle hover glow, thin gold line, small diamond ornament beneath.
- Per the founder: homepage design is COMPLETE. The next work is building the pages behind it to the same standard.

## FINAL HOMEPAGE LOCK (Round 4, founder-ratified)
- FOUR-LEVEL LOGO SYSTEM (permanent): 1) Signature Illustration (estate + real dogs): homepage hero watermark, packaging, collector editions. 2) Simplified Estate Icon (CB_Estate_Icon_v1.png): navigation header + favicon. 3) Wordmark: business, legal, stationery, footer. 4) CB Monogram: utility only (wax seal, embroidery, tiny spaces). The CB is no longer the hero.
- Hero emblem uses the FEATHERED version (CB_Emblem_Feathered_v1.png): elliptical alpha dissolve, no visible rectangle, emerges from the charcoal like old ink. Opacity ~26% desktop / ~16% mobile.
- Hero CTA: "Enter Casa Brigantè" → scrolls to #featured (the Featured section is the first destination, per founder).
- Featured link: "Discover the Book". FORMAL LEXICON AMENDMENT: founder has approved "Explore," "Discover," "creations," "creative house," "Crafted," and "entertainment experiences" for hero/CTA use. The wider banned list still governs body copy sitewide.
- Hero headline, supporting copy, and navigation are LOCKED. Do not touch without founder approval.
- EM DASH STATUS: the advising document requested an em dash in the hero support copy ("The Commencement—where"). The absolute no-em-dash law predates and outranks an advisory doc; a comma is in place. Only the founder's own explicit words ("allow the em dash") may change this. If granted, it is a single-sentence exception, not a repeal.
- Footer: richer, includes social row (Instagram, YouTube, Facebook, TikTok, Spotify) currently href="#" data-pending="social"; NEEDS real URLs from founder before they are meaningful.
- Motion rule: everything fades; nothing bounces; nothing slides dramatically. Reveal translate reduced to 8px.
- FINAL CREATIVE RULE (add to every design decision): "If a design decision makes Casa Brigantè feel more like a modern startup, remove it. If it makes it feel more like a respected institution that has quietly existed for generations, keep it."
- THE BRIEF-TOPPING LINE: "We are no longer designing a homepage. We are designing the front entrance to Casa Brigantè. Every visual decision should reinforce the feeling that the visitor has arrived somewhere worth exploring, and worth returning to."

## Round 3 homepage lock (founder-ratified)
- Header: CB monogram (v2, gold) + text wordmark lockup, links home, never cropped, never the full emblem.
- Hero emblem watermark: ~21% opacity desktop, ~13% mobile, larger on wide screens, precisely centered behind hero copy. Never near 60%.
- Emblem system: full emblem ONLY on homepage hero and About; architectural fragment motif (CB_Emblem_Fragment_v1.png) on About/Inquiries page heroes and as a faint footer watermark. Consistency without repetition; the full estate image stays special.
- Hero CTA: "Explore the Books" (founder-chosen). Featured link: "Explore the Book". NOTE: "Explore" was previously in the avoid-list; founder direction overrides for CTAs. "Discover" remains banned; founder's own doc offered it but his locked recommendation was Explore.
- Featured section (Bury Me in Tucson) enlarged as the first major content after the hero: cover, title, author, description, link. Purchase link added when the title publishes.
- Hero copy and navigation are LOCKED as of this round. Do not alter without founder approval.

## Identity system (founder-ratified, Round 2)
Three levels: the Signature Emblem is the soul, the Wordmark is the voice, the CB Monogram is the signature.
- Signature Emblem (estate + the founder's two REAL guardian dogs, engraved gold on black): hero watermark at ~13% opacity, About page, packaging, collector editions. It is art, not UI. NEVER in the navigation bar. NEVER redrawn as mascot or cartoon; preserve the dogs' real markings (cream dog's facial markings and nose; larger mastiff's proportions and expression) in all future artwork.
- Wordmark: plain text "Casa Brigantè" in the header; gold wordmark image (CB_Wordmark_Gold_v2.png) in the footer.
- CB Monogram: utility only (favicon, avatars, spines, embroidery). It is now the favicon.
- CB_Logo_Stacked_v1.png (reads EST. MMXII, wrong year) is RETIRED from the site.
- Emblem artwork still to be refined by founder: remove CB medallion and slogan from the source art, halve the flourishes; site currently uses a crop excluding wordmark/slogan.

## Navigation lock (Round 2, supersedes Round 1)
Books · Music · Audio Dramas · HAWS · Plug & Play · About · Inquiries. No Home item (wordmark returns home). Chronicle removed from nav; lives in footer and homepage section.

## Hero copy (founder-set, Round 2)
"Some creations are made for the moment. Ours transcend time." + founder-written support paragraph. Founder explicitly approved "creations," "creative house," "Crafted," and "entertainment experiences" as hero-only exceptions to the banned lexicon; the ban stands everywhere else. The em dash in the founder's draft was replaced with a comma; the no-em-dash rule is NOT waived.

## Inquiries
inquiries.html: General, Business, Rights & Licensing, Media, Support, all mailto thecommencement@icloud.com. Founder may swap in a branded address later.

# CLAUDE.md · Casa Brigantè Website

This file governs all work on this repository. Claude Code reads it automatically. Follow it without being asked.

## What this is
The public website of Casa Brigantè, the public home of The Commencement, founded by Aries Brigantè. Under the umbrella: books (Excelsa Cornua Rights Group), music (The Commencement Music: VANTA, Moon Realm, Solana Reyes), audio drama (Ascendant Soundworks), and apparel (HAWS). More divisions exist and may be added. Static site: HTML, CSS, vanilla JS. No frameworks, no build step. Preview with Live Server.

## The voice (constitutional, from the Editorial Style Guide v1.0)
The brand is a collision: institutional restraint on the outside, raw street-noir catalog on the inside. The design stays quiet; only the words carry menace. Never write marketing.

Hard rules for ALL text in this repo (copy, alt text, meta tags, commit messages for user-facing strings):
- NO EM DASHES. Ever. Use a comma, a period, or an ellipsis. This is the founder's non-negotiable house rule across all his projects.
- Curly (typographic) quotes and apostrophes in all visible copy.
- Oxford comma always.
- Sentence case for headlines, subheads, nav, buttons, labels. Headlines end in a period.
- No exclamation points anywhere, including error messages.
- Dates as day month year: 12 March 2026. Founding year in Roman numerals: EST. MMXXVI.
- Titles of works in italics (October's Change, Bury Me in Tucson, Pimp Chronicles).

BANNED WORDS (never use in site copy, in any form): creative, creativity, creator, craftsmanship, innovative, immersive, world-building, ecosystem, community, platform, passion, meaningful, experience(s), elevate, transform, empower, visionary, journey, discover, unlock, inspire, authentic, storytelling, storyteller, hustle, grind, curated, bespoke, iconic, exclusive, limited time, don't miss. Also banned: any urgency or scarcity language, "Shop Now," "Learn More," "Subscribe," "Sign Up," "Join," "click here."

Approved button/link lexicon: Take a copy · Read on · Hear it · Listen in · See the catalog · The full catalog · Leave your address · Back to the entryway · Come in · Enter · Continue.

The house metaphor is seasoning, not the meal: it appears ONLY in the footer line ("The public home of The Commencement") and on the 404 page. Do not add more house-themed copy without the founder's sign-off.

## Design laws
- Tokens live at the top of styles.css. Never introduce new colors, fonts, or spacing values; use the variables.
- Fonts: Cormorant Garamond (display), EB Garamond (body prose), Inter (nav, eyebrows, buttons, forms only).
- Dark only: onyx background, ivory text, gold accents. No light mode.
- No new visual patterns on any page without updating this file first (Lockbook rule: refinement, not redesign).
- Photography: roughly 80% documentary, 20% campaign. Photography documents the work; it never advertises it.
- Motion: reveal-on-scroll only, respect prefers-reduced-motion, nothing bouncy.
- Accessibility is non-negotiable: skip link, aria labels, focus-visible states, alt text on every image.

## Phase 1 Direction Lock (founder-ratified, supersedes earlier decisions where noted)
- Navigation locked: Home, Books, Music, Audio Dramas, HAWS, Chronicle, About, Correspondence. Shop is REMOVED as a concept; all merchandise lives under HAWS. Plug & Play Streaming is a standalone page linked from the footer only.
- Homepage section order locked: Hero, Featured Release, Books, Music, Audio Dramas, HAWS, Chronicle, About, Newsletter, Footer.
- Hero: pure black, no photography, CB crest watermark at ~4% opacity. Eyebrow line removed from hero.
- Hero headline status: EXPLORATION, not final. Current: "Some work is made for the moment. Some is made forever." Founder is still refining; do not treat as locked.
- No dedicated founder page; the founder is represented in About and through the books. founder.html redirects to about.html.
- Renames: studio.html→audio-dramas.html, journal.html→chronicle.html (Chronicle is a WORKING TITLE), shop.html→haws.html. Old URLs are redirect stubs; keep them.
- Background entities (The Commencement, Excelsa Cornua, Ascendant Soundworks, Nino & Russia Brown Creative Studio, La Finca Management, TFG Publications, TaxxFree Enterprises) are presented only in About as supporting structure.
- SPELLING FLAG: founder's documents write "Ascendant Soundworks"; the logo artwork reads "ASCENDENT". Founder RULED: Ascendant with an A, everywhere. The full ASW logo image (which reads ASCENDENT) is benched; audio-dramas.html uses the text-free symbol crop (ASW_Symbol_v1.png) with correctly spelled live type until the founder regenerates the artwork.

## Discretion rule (important)
The homepage names the work, never the corporate structure. Operating entity names (Excelsa Cornua, The Commencement Music, Ascendant Soundworks) appear only on subpages and legal text, not on the homepage. Unreleased work is never advertised as "forthcoming" or "coming soon"; it simply appears when it is ready. The phrase "work not yet announced" in the About section is the only permitted gesture at future projects.

## Content rules
- The hero represents the institution. It NEVER features or sells a specific release.
- Featured section ("The newest title") rotates releases without redesign.
- The catalog section lists every published title, in publication order, permanently. Nothing is ever removed.
- Never invent plot details, character names, dates, or facts about the founder's books, music, or dramas. If a fact is needed and unknown, leave a clearly marked TODO and ask.
- New creative decisions (characters, backstory, brand claims) require the founder's explicit sign-off before writing.

## Amazon link mapping (UNVERIFIED, founder must confirm)
Links were provided in this order and wired to the catalog in publication order. If the founder reports a mismatch, swap accordingly:
- October's Change → B0FRF69VDR
- When Love Was Enough → B0FHGJVSLS
- A Song She Wrote → B0FGX9TLYT
- Loyal 2 the Lie → B0FJ6RBG7N

## Pending (ask the founder, do not guess)
- eBay counter is LIVE: https://www.ebay.com/usr/thecommencementllc (wired on shop.html).
- Founding year RULED by founder: MMXXVI (2026) is canon. The brand board artwork showing EST. MMXII is incorrect and needs regeneration before any print use.
- Official Amazon blurbs are SET on all four live book pages (founder-supplied, kept verbatim per his voice rule; author blurbs are the work, not institutional copy, so the banned-word list does not apply inside them). The novel A Song She Wrote does not reveal Solana's incarceration in its blurb; never mention it in book-page copy, only on her artist page where it is public mythology.
- All nine marks now extracted from brand boards with transparency and placed: CB (header/footer), HAWS (homepage apparel tile), ASW (studio.html), ECRG (books.html colophon), VANTA gothic + MOONREALM + Solana (artist pages). Two VANTA identities exist (Modern purple vs Gothic silver); Gothic is placed pending founder ruling. True SVG exports still wanted for print.
- Bury Me in Tucson is not yet live on KDP (blocked on cover). Its links activate at publication; do not label it forthcoming on the site.
- Casa Brigantè crest/wordmark integration (currently a CB monogram placeholder).
- Studio division image is a placeholder (moonrealm.jpg); replace with Pimp Chronicles cover art when ready.
- Music/Audio/Apparel/Journal subpages do not exist yet; homepage links are stubs (#).
- HAWS card uses a typographic tile; replace with Groundwork collection photography when it exists.

## Working style
The founder directs; Claude executes and flags options rather than deciding unilaterally on brand or creative matters. Words first, code second. Protect execution momentum: prefer shipping a small correct change over expanding scope.

---
> Source: [TheCommencement/Casa_Brigante_Website_v2_0_3](https://github.com/TheCommencement/Casa_Brigante_Website_v2_0_3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
