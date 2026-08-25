## weddinginvitation

> You are the senior frontend engineer and interaction designer responsible for shipping this demo. Do not redesign the product from scratch. The creative direction, content, asset names, event data, and mobile experience below are approved. Preserve them unless the user explicitly asks for a change.

# Elevare Wedding Invite Demo — Claude Code Build Brief

## Role
You are the senior frontend engineer and interaction designer responsible for shipping this demo. Do not redesign the product from scratch. The creative direction, content, asset names, event data, and mobile experience below are approved. Preserve them unless the user explicitly asks for a change.

## Product goal
Build a premium, mobile-first digital Indian wedding invitation for **Shubham & Riddhi** that Elevare can later reuse as a paid client template. It must feel like entering a royal cinematic invitation, not reading a generic wedding landing page.

The commercial product target is a reusable **₹8k–₹25k wedding invite website + video service**. Therefore, client-specific content must live in one configuration file and must not be scattered across components.

## Experience target
The guest should feel:
1. Warmly welcomed.
2. Surprised by the level of presentation.
3. Guided through the couple's story and events without friction.
4. Able to get directions or RSVP in one tap.

Visual tone: **Indian miniature-inspired romantic illustration + contemporary cinematic luxury**.

Avoid: Canva-template styling, loud gradients, neon, heavy green palettes, floating hearts, excessive particles, spinning mandalas, confetti loops, cursor effects, autoplay hacks, or animation on every element.

## Technical architecture
- Next.js 16 App Router.
- TypeScript, strict mode.
- Keep dependencies minimal.
- Use `next/image` for all major visuals.
- Use CSS + IntersectionObserver for motion unless a requirement genuinely needs an animation library.
- No backend.
- No database.
- WhatsApp RSVP only.
- Client-specific data lives in `src/config/invitation.ts`.
- The site must deploy cleanly to Vercel.

## Folder contract
Do not rename these without a strong technical reason:

```text
src/
  app/
    globals.css
    layout.tsx
    page.tsx
    opengraph-image.jpg
    twitter-image.jpg
  components/
    InviteExperience.tsx
    Reveal.tsx
  config/
    invitation.ts

public/
  images/
    hero.jpg
    story-first-meeting.jpg
    celebration.jpg
    wedding-day.jpg
    closing.jpg
    og-whatsapp.jpg
  audio/
    wedding-theme.mp3   # only after a licensed track is supplied
  shubham-riddhi-wedding.ics
```

## Locked visual palette
Do not introduce green as a primary UI colour.

- Ivory: `#F7F0E4`
- Parchment: `#E7D3B7`
- Antique Gold: `#B78A45`
- Dusty Rose: `#C97782`
- Deep Rose: `#9A4A58`
- Wine: `#5A1E2B`
- Maroon: `#762E3A`
- Ink: `#2F2422`

Natural foliage that already exists inside the illustrations is acceptable; the interface itself should remain ivory / gold / rose / wine / maroon-led.

## Fonts
- Display / names / emotional headlines: `Cormorant Garamond`.
- Utility copy / buttons / event information: `Manrope`.
- Marathi / Devanagari accents: `Noto Serif Devanagari`.
- Use `next/font/google` and `display: swap`.
- Never use multiple script fonts.

## Approved section order and exact copy

### 1. Entry Gate
Full viewport. Ivory paper texture. Minimal royal arch treatment.

Copy:
- `॥ श्री गणेशाय नमः ॥`
- `Together with our families`
- `Shubham & Riddhi`
- `22 · 11 · 2025`
- CTA: `Open the Invitation`
- Microcopy: `Tap to enter the celebration`

Behavior:
- The invitation content is hidden until the CTA is tapped.
- This tap is the only acceptable point to start audio when audio is enabled.
- Do not try to bypass browser autoplay restrictions.

### 2. Opening / Hero Couple
Asset: `/images/hero.jpg`

Copy:
- `शुभ विवाह`
- `Shubham`
- `&`
- `Riddhi`
- `Together with our families, we invite you to celebrate our beginning.`
- `22 · 11 · 2025`
- Microcopy: `Scroll to enter our story`

Motion:
- Controlled 3–5% slow image push after entry.
- Text reveal only; no bouncing or floating elements.

### 3. Our Beginning
Asset: `/images/story-first-meeting.jpg`

Eyebrow: `Our Beginning`
Headline: `It began with a first meeting.`

Body copy, use exactly:
`Our story began in Nashik, when Riddhi and her family came to Shubham's home with a wedding proposal. What started as a traditional introduction became an easy conversation, then comfort, then certainty. Two people met. Two families connected. And a new chapter quietly began.`

Marathi line:
`एका भेटीतून सुरू झालेली आपली गोष्ट…`

Tone: arranged-marriage story told with confidence and warmth. Do not rewrite it into a dating / college-love story.

### 4. Celebration Visual
Asset: `/images/celebration.jpg`

Overlay:
- `21 November 2025`
- `The Celebrations Begin`

Use this as a cinematic visual break before the event cards.

### 5. Celebration Events — 21 November 2025
Header:
- `शुक्रवार`
- `21 November 2025`
- `A day of colour, blessings & togetherness.`

Events:
1. `Haldi & Grahashanti`
   - `10:00 AM`
   - `Hira Executive, Nandurbar`

2. `Swaruchi Bhojan`
   - `12:00 PM onwards`
   - `Hira Executive, Nandurbar`

3. `Raas Garba`
   - `7:00 PM`
   - `Hira Executive, Nandurbar`

Hira directions URL:
`https://www.google.com/maps/search/?api=1&query=Hotel%20Hira%20Executive%2C%20Dhule%20Road%2C%20Nandurbar%2C%20Maharashtra%20425412`

### 6. Wedding Day Events — 22 November 2025
Header:
- `शनिवार`
- `22 November 2025`
- `And then, the day we've been waiting for.`

Events:
1. `Varat Prasthan`
   - `6:00 AM`
   - `Hira Executive, Nandurbar`

2. `Varghoda`
   - `6:00 PM`
   - `Mahadev Mandir → V.S.S. Lawns, Shahada`
   - note: `Procession to the wedding venue`

3. `Wedding Ceremony`
   - Marathi: `गोरज मुहूर्त`
   - **Do not show a wedding clock time.**
   - `V.S.S. Lawns, Shahada`
   - note: `At Goraj Muhurta`

V.S.S. directions URL:
`https://www.google.com/maps/search/?api=1&query=V.S.S.%20Lawns%20%26%20Resorts%2C%20236%20Kukdel%20Bhajipir%20Road%2C%20Shahada%2C%20Maharashtra%20425409`

### 7. Wedding-Day Royal Visual
Asset: `/images/wedding-day.jpg`

Overlay:
- `गोरज मुहूर्त`
- `Shubham & Riddhi`
- `V.S.S. Lawns · Shahada`

### 8. Families
Headline: `Together with our families`
Eyebrow: `With their blessings`

Only show parents. Do not reproduce the extended-family directory from the printed card.

Shubham:
- `Son of`
- `Nikita & Nitin Suresh Sonar`

Riddhi:
- `Daughter of`
- `Shraddha & Lilkesh Ramakant Soni`

### 9. Venue
Eyebrow: `The Wedding Venue`
Headline: `V.S.S. Lawns & Resorts`
Address:
`236, Kukdel Bhajipir Road, Shahada, Maharashtra 425409`

Buttons:
- `Get Directions`
- `Add to Calendar`

The calendar file already exists at `/shubham-riddhi-wedding.ics`.

### 10. RSVP
Marathi line:
`आपली उपस्थिती आम्हाला आनंद देईल.`

Headline:
`Will you celebrate with us?`

Support copy:
`One tap opens WhatsApp with your reply ready to send.`

CTA:
`RSVP on WhatsApp`

WhatsApp number: `+91 70205 70091`
URL number: `917020570091`
Prefilled message:
`Hi Shubham & Riddhi, we'll be joining you for the celebration. ❤️`

Do not add an RSVP database or form.

### 11. Closing Portrait
Asset: `/images/closing.jpg`

Copy:
- `With love,`
- `Shubham & Riddhi`
- `22 · 11 · 2025`
- `शुभ विवाह`

Footer, very small:
`An Elevare Digital Invitation`

Do not place a large Elevare logo inside the invitation.

## Motion rules
Motion must support hierarchy, not advertise the code.

Allowed:
- 650–900 ms fade + 16–24 px vertical reveal on section content.
- 3–5% slow image pushes.
- One subtle arch-mask / cinematic section change if it can be implemented without adding weight.
- Sequential event-card reveals with 60–90 ms stagger.

Not allowed:
- constant floating hearts
- spinning mandalas
- confetti loops
- springy cards
- huge parallax distances
- cursor effects
- scroll-jacking
- heavy WebGL

Always respect `prefers-reduced-motion`.

## Audio behavior
A licensed track is not currently bundled.

Direction once a track is supplied:
`soft tanpura ambience → bansuri / santoor → warm cinematic strings → restrained Indian percussion`

Rules:
- User taps `Open the Invitation` first.
- Start music from that user gesture.
- Initial volume around 35–40%.
- Loop gently.
- Fixed discreet `Music on / Music off` control.
- If the audio file is missing or disabled, render no broken player and no dead control.
- Do not use a copyrighted commercial Bollywood track in the public demo without a valid license.

Enable music by placing `public/audio/wedding-theme.mp3` and changing `audio.enabled` in `src/config/invitation.ts` to `true`.

## Mobile performance rules
This is a phone-first invite. Treat slow event-venue internet as a real constraint.

- Optimize hero images through `next/image`.
- Keep source JPEGs roughly under 500–700 KB where practical; current prepared assets are already compressed.
- Hero is priority. Everything below the fold should lazy-load.
- Avoid large background video.
- Avoid animation packages unless they solve a concrete problem.
- Tap targets minimum ~44 px.
- Test 360 px, 390 px, 430 px widths before desktop polish.
- Test Safari iOS behavior for `100svh`, audio, fixed controls and safe-area insets.

## Open Graph / WhatsApp preview
`src/app/opengraph-image.jpg` and `src/app/twitter-image.jpg` already exist at 1200 × 630.

Do not remove them. The invite must look intentional when pasted into WhatsApp before the guest opens it.

Metadata title:
`Shubham & Riddhi | 22.11.2025`

Metadata description:
`Together with our families, we invite you to celebrate our beginning.`

## Reusability rule
For the next client, the engineer should be able to replace:
- names
- dates
- parents
- story
- event list
- venue links
- WhatsApp number
- artwork paths
- theme values

without editing section JSX.

If client-specific text appears directly in JSX and it already exists in `src/config/invitation.ts`, move it back to config.

## QA definition of done
Do not call the demo finished until all of these pass:

1. `npm run build` completes without errors.
2. No horizontal overflow at 360 px.
3. Entry CTA works on a real phone.
4. Hero image fills the first viewport without layout shift.
5. Every event and spelling matches `src/config/invitation.ts`.
6. Wedding Ceremony shows `गोरज मुहूर्त` and no fabricated clock time.
7. Hira and V.S.S. direction links open usable map searches.
8. WhatsApp CTA opens `917020570091` with the correct prefilled message.
9. Calendar download works.
10. Open Graph image is present and under Next.js limits.
11. Site remains usable with JavaScript animation reduced / prefers-reduced-motion enabled.
12. Lighthouse mobile review shows no obvious image / layout / accessibility failure.

## Current state / first Claude Code task
The repository has already been scaffolded manually with the approved assets, config, sections, metadata and responsive CSS.

Your first job is **not to redesign it**.

Do this in order:
1. Run `npm install`.
2. Run `npm run dev`.
3. Inspect the page at 390 px width first.
4. Fix only compile/runtime issues if any.
5. Compare every section to this brief.
6. Run `npm run build`.
7. Report concrete remaining defects before making aesthetic changes.

---
> Source: [sonarshubhamn-ux/WeddingInvitation](https://github.com/sonarshubhamn-ux/WeddingInvitation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
