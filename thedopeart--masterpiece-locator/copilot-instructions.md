## masterpiece-locator

> Art discovery app with artwork pages featuring descriptions and FAQs. Content interlinks to internal pages (artists, museums, movements) and external LuxuryWallArt.com collections.

# Masterpiece Locator - Content Guidelines

## Project Overview
Art discovery app with artwork pages featuring descriptions and FAQs. Content interlinks to internal pages (artists, museums, movements) and external LuxuryWallArt.com collections.

---

## Workflow

### Step 1: Check the Checklist
Open `ARTWORK-CHECKLIST.md` to see what needs work. Artworks are prioritized by tier:
- **Tier 1**: High-traffic, famous paintings (do first)
- **Tier 2**: Medium priority
- **Tier 3**: Lower priority

Pick artworks marked `TODO`. When working with multiple agents, claim a batch by noting which slugs you're working on to avoid duplicates.

### Step 2: Research Each Artwork
Before writing, gather real information:
1. Check if Wikipedia URL exists in the checklist
2. Use WebFetch to read the Wikipedia page for facts
3. Look for: creation date, what's depicted, history, controversies, current location, interesting facts
4. Only write what you can verify - never invent details

### Step 3: Write Description & FAQs
Follow the guidelines below. Length depends on available content:
- Limited info → 130-150 words
- Rich history → up to 350 words

### Step 4: Update Database
Run Prisma update script for each batch.

### Step 5: Mark Complete
After updating, regenerate the checklist:
```bash
node get-artworks.js
```

---

## Artwork Descriptions

### Length
- **130-150 words** for paintings with limited context
- **Up to 350 words** only if there's genuinely interesting, true content
- Never pad text just because a painting is famous

### Structure
- Use `<p>` tags for paragraphs (typically 2-3 paragraphs)
- No `<br>` tags between paragraphs (CSS handles spacing)
- Vary sentence length dramatically

### Typical Paragraph Flow
1. **Opening paragraph**: Artist link, date, what the painting depicts visually
2. **Middle paragraph**: Historical context, technique, interesting facts, controversies
3. **Closing paragraph**: Artist bio context, current museum location with link

### What to Include (when true and interesting)
- What the painting actually shows (describe the scene)
- When and where it was created
- Historical context or story behind it
- Interesting facts (scandals, thefts, X-ray discoveries, controversies)
- Technical approach (technique, medium, series context)
- Artist's life context relevant to the work
- Current location and how it got there

### What NOT to Do
- Don't pad with generic art appreciation language
- Don't repeat information already in the metadata (title, year shown on page)
- Don't invent facts - only write what you actually know
- Don't write longer just because a painting is famous

### Bold Keywords
- Minimum **1 bold keyword per paragraph**
- Bold text is black unless inside a link
- Use `<strong>` tags

---

## Interlinking

### Internal Links (display in green #028161)
```html
<a href="/apps/masterpieces/artist/artist-slug"><strong>Artist Name</strong></a>
<a href="/apps/masterpieces/movement/movement-slug"><strong>Movement Name</strong></a>
```

### Museum Links (display in gold #C9A84C)
```html
<a href="/apps/masterpieces/museum/museum-slug"><strong>Museum Name</strong></a>
```

### LuxuryWallArt Links (display in gold #C9A84C)
Link to collections when content naturally matches. Great opportunities:

**Colors** (when describing palette):
- Gold `/collections/gold-art` | Black and Gold `/collections/black-and-gold`
- Blue `/collections/blue-wall-art` | Navy Blue `/collections/navy-blue` | Royal Blue `/collections/royal-blue`
- Red `/collections/red-wall-art` | Red and Black `/collections/red-and-black-art`
- Green `/collections/green-wall-art` | Emerald `/collections/emerald-green`
- Yellow `/collections/yellow-paintings` | Orange `/collections/orange-art`
- Pink `/collections/pink-wall-art` | Purple `/collections/purple-paintings`
- Gray/Grey `/collections/grey-art` | Beige `/collections/beige` | Brown `/collections/brown-art`
- Black and White `/collections/black-and-white-artwork` | Colorful `/collections/colorful-artwork`
- Earth Tones `/collections/earth-tones` | Neutral `/collections/neutral-art`

**Dark/Macabre themes** (skulls, death, horror):
- Skeleton & Skulls `/collections/skeleton-skull-art`
- Macabre `/collections/macabre-art`

**Religious/Spiritual** (angels, religious scenes):
- Angel `/collections/angel-art` | Spiritual `/collections/spiritual-art` | Zen `/collections/zen-art`

**Subjects** (when depicted):
- Women `/collections/women-art` | Portrait `/collections/portrait-art` | People `/collections/people-paintings`
- King and Queen `/collections/kings-and-queens` | Crown `/collections/crown-paintings`
- Floral `/collections/floral-art` | Nature `/collections/nature-art` | Tree `/collections/paintings-of-trees`
- Mountain `/collections/mountain-art` | Sunset `/collections/sunset-paintings` | Landscapes `/collections/landscapes`

**Animals** (when depicted):
- Horse `/collections/horse-art` | Dog `/collections/dog-paintings` | Cat `/collections/cats`
- Lion `/collections/lion-wall-art` | Tiger `/collections/tiger-paintings` | Big Cat `/collections/big-cat-art`
- Bird `/collections/bird-art` | Eagle `/collections/eagle-art` | Peacock `/collections/peacock-wall-art`
- Bull `/collections/bull-paintings` | Cow `/collections/cow-art` | Bear `/collections/bear-paintings`

**Nautical** (sea scenes):
- Coastal `/collections/coastal-decor` | Beach `/collections/beach-artwork` | Wave `/collections/wave-art`
- Boat `/collections/boat-art` | Lighthouse `/collections/lighthouse-paintings`

**Styles** (when matching art movement):
- Abstract `/collections/abstract-wall-art` | Minimalist `/collections/minimalist-art`
- Surrealism `/collections/surrealism-art` | Pop Culture `/collections/pop-culture`
- Tropical `/collections/tropical-art` | Country and Farm `/collections/country-farm-paintings`

```html
<a href="https://luxurywallart.com/collections/[slug]" target="_blank" rel="noopener"><strong>text</strong></a>
```

**ONLY use verified URLs from:**
`/Users/thedopeart/Desktop/luxury-wall-art/Interlinking/Collections Links.csv`

190+ collections available. DO NOT guess URLs - always verify in the CSV first.

### Brand Keywords & Homepage Link
When it fits naturally, weave in brand-related keywords:
- "luxury art"
- "luxury wall art"
- "luxurious decor"

Link to the homepage when using these phrases:
```html
<a href="https://luxurywallart.com" target="_blank" rel="noopener"><strong>luxury wall art</strong></a>
```

**Don't overdo it.** Use sparingly, roughly 1 in every 7-10 artworks. Only include when it flows naturally with the content.

---

## FAQs

### Requirements
- **2-5 FAQs per artwork** (more for famous paintings)
- **25-40 words per answer**
- Every FAQ answer must have at least one interlink
- Bold key terms

### Common FAQ Topics
- Where can I see [painting]? → link museum
- What style/movement? → link movement
- What colors? → link LWA collection if natural fit
- How big is it? (include both metric and imperial)
- When was it painted?
- Who painted it? → link artist
- What does it depict/mean?
- Why is it famous? (for well-known works)
- Interesting facts (theft, scandal, hidden details)

### FAQ Answer Structure
1. Direct answer first
2. Add supporting detail or context
3. Weave in link naturally

**Example:**
```
Q: Where can I see The Tempest?
A: You can view this painting at <a href="/museum/..."><strong>Gallerie dell'Accademia</strong></a> in Venice, Italy. The museum preserves Venetian art spanning five centuries, and The Tempest remains one of its most mysterious treasures.
```

### Link Placement Patterns
- **Artist link**: Usually in first paragraph, first mention of artist name
- **Movement link**: When discussing style or art historical context
- **Museum link**: In description's final paragraph OR in "where to see" FAQ
- **LWA collection link**: When naturally mentioning colors (don't force it)

---

## Writing Style

### Grammar Rules
- Never start sentences with "And"
- Never use em dashes (use commas, periods, parentheses)
- Use contractions: "don't" not "do not"

### Filler Patterns - AVOID
- "The painting is..."
- "This masterpiece..."
- "There is a [quality] to..."
- "What makes X [adjective] isn't... but..."
- "It's important to note..."
- "This approach was typical of..."
- "The [adjective] masterpiece has..."

### Banned Words
| Category | Words |
|----------|-------|
| Superlatives | exceptional, unparalleled, revolutionary, remarkable, transformative, groundbreaking |
| Adjectives | captivating, exquisite, stunning, compelling, intriguing, intricate, comprehensive, pivotal |
| AI favorites | tapestry, meticulous, testament, endeavors, realm, myriad, paramount, bespoke, curated, masterpiece, enigmatic |
| Transitions | furthermore, moreover, additionally, consequently, thus, hence |
| Verbs | unleash, unlock, empower, embark, delve, foster, navigate, harness, elevate |

### Banned Phrases
- "Whether you're..."
- "In today's world..."
- "At its core..."
- "Not only... but also..."
- "Embark on a journey..."
- "A tapestry of..."
- "Transform your space"
- "Make a statement"

### Structure Red Flags
- Three adjectives in a row
- Uniform sentence length
- Starting paragraphs with transitional adverbs
- Perfectly balanced lists

---

## Database

### Schema
```prisma
model Artwork {
  description  String?   // HTML with <p> tags and links
  faqs         Json?     // Array of {question: string, answer: string}
}
```

### Updating Artworks
```javascript
await prisma.artwork.update({
  where: { slug: 'artwork-slug' },
  data: {
    description: `<p>...</p><p>...</p>`,
    faqs: [
      { question: "...", answer: "..." },
      { question: "...", answer: "..." }
    ],
    updatedAt: new Date()
  }
});
```

---

## Pre-Save Checklist

- [ ] Word count: 130-350 based on available true content
- [ ] Bold keywords: at least 1 per paragraph
- [ ] FAQ answers: 25-40 words each with interlinking
- [ ] No banned AI words or phrases
- [ ] No filler text patterns
- [ ] No sentences starting with "And"
- [ ] No em dashes
- [ ] Varied sentence lengths
- [ ] All LWA URLs from verified list above
- [ ] Artist, movement, museum links included naturally
- [ ] Consider brand keyword with homepage link (1 in 7-10 artworks, only if natural)

---
> Source: [thedopeart/masterpiece-locator](https://github.com/thedopeart/masterpiece-locator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
