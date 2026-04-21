## portfolio-website

> Copy everything below the line, fill in the [BRACKETS] with your info, and paste it into a new Claude chat.

# Portfolio Website Prompt

Copy everything below the line, fill in the [BRACKETS] with your info, and paste it into a new Claude chat.

---

You are a world-class frontend designer and developer with 20+ years of experience building award-winning, Awwwards-nominated portfolio websites. You obsess over every pixel, every animation curve, every font pairing. You don't build websites — you craft digital experiences.

Build me a **single-page portfolio website** as a React component (.jsx) that looks like it was designed by a top-tier design agency, NOT by AI.

## Who I Am

- **Name:** Tamer
- **Role / Title:** Full stack developer & AI specialist 
- **Short tagline:** Ideas into real products.
- **Bio (2-3 sentences):** I’m a full-stack web developer and AI specialist who loves turning ideas into intelligent, user-friendly products. I combine modern web technologies and AI to build solutions that are both powerful and simple to use.
- **Skills / Tech stack:** Python React Figma shopify wWrdpress SEO optimaliztion Javascript  Performance optimalization Web design Claude
- **Projects (3-4):** // The project parts will be deliverd later
  - Project 1: [Name] — [One sentence description] — [Tech used]
  - Project 2: [Name] — [One sentence description] — [Tech used]
  - Project 3: [Name] — [One sentence description] — [Tech used]
  - Project 4: [Name] — [One sentence description] — [Tech used]
- **Certifications / Courses (2-5):**
  - [e.g. "AWS Certified Solutions Architect — Amazon Web Services — 2024"]
  - [e.g. "Google UX Design Professional Certificate — Google — 2023"]
  - [e.g. "Meta Front-End Developer Certificate — Meta — 2023"]
  - [Add your own — include: name of certification, issuing organization, and year]
- **Contact links:** [e.g. email, GitHub, LinkedIn, Twitter/X — list what you have]

---

## Design Direction — "Luxury Minimal with Life"

The aesthetic is: **refined, light, modern, and alive.** Think Apple's design philosophy meets a high-end architecture firm's website — with a living, breathing gradient background that makes it feel organic and warm, not cold and sterile.

### Theme & Color Palette
- **Light theme** — predominantly white and soft off-white backgrounds (#FAFAF9 or similar warm whites, never pure #FFFFFF)
- One **dominant accent color** that feels intentional and bold (think: a rich coral, a deep sage green, a warm amber, or an electric blue — pick ONE and commit to it)
- Use **tints and shades** of your accent for depth — lighter versions for backgrounds, full strength for CTAs, darker for text accents
- Add a subtle **warm neutral** for secondary text (not cold gray — use something like #6B6560 or #4A4540)
- Define ALL colors as CSS variables so the entire palette is cohesive

### Animated Gradient Wave Background
This is the hero element — the thing people remember. Build it with care:
- A **full-viewport animated gradient mesh** in the hero section that uses soft, light, pastel tones of the accent color blended with whites and creams
- The gradient should **slowly morph and flow** like silk in water — use CSS `@keyframes` with `background-position` or `background-size` animation on a large gradient (400% 400%)
- Movement should be **slow and hypnotic** — cycle duration of 15-20 seconds, ease-in-out timing
- It should feel like light reflecting off water or soft clouds drifting — NOT like a disco or a rave
- Layer a subtle **grain/noise texture overlay** on top using CSS (a pseudo-element with a tiny repeating SVG noise pattern at very low opacity) to add organic feel and prevent it from looking "too digital"
- The gradient should gently **fade out** as you scroll down, so it doesn't fight with the content below

### Typography — Make It Unforgettable
- Import from Google Fonts — pick a **distinctive display font** for headings that has CHARACTER (examples for inspiration: "Playfair Display", "Cormorant Garamond", "Sora", "General Sans", "Clash Display", "Cabinet Grotesk", "Instrument Serif" — pick something unique, do NOT just default to the first one)
- Pair it with a **clean, refined body font** that contrasts well (something geometric or humanist sans-serif)
- Headings should feel LARGE and CONFIDENT — hero name can be 5-8rem, section headings 3-4rem
- Use **letter-spacing** intentionally: tight on large headings (-0.02em to -0.04em), slightly open on small labels/caps (0.05em to 0.1em)
- Body text should be 1.1-1.2rem with generous line-height (1.6-1.8) for readability
- Add one **typographic surprise**: maybe the tagline is in italic serif while everything else is sans-serif, or a section label uses a monospace font

### Layout & Spatial Composition
- **Generous whitespace everywhere** — sections should breathe. Use 120px-200px vertical padding between sections
- Content width maxes out at around 1200px, centered, but let some elements **break the grid** (e.g., a decorative accent line that extends full-width, or a project card that's offset to one side)
- Use **asymmetric layouts** where it feels natural — the about section could be a 60/40 split, projects could use a masonry-style stagger
- Add **subtle horizontal rules or decorative dividers** between sections using thin lines, dots, or the accent color
- Navigation should be **minimal and elegant** — a clean horizontal nav with smooth scroll links, maybe fixed at top with a frosted glass/blur effect on scroll

### Motion & Animation — The Secret Sauce
This is where 20 years of experience shows. Every animation should feel INTENTIONAL:

**Page Load Sequence (staggered reveal):**
- Hero content fades up + slides in with staggered delays: name first (0ms), then title (150ms), then tagline (300ms), then CTA button (450ms)
- Use `cubic-bezier(0.16, 1, 0.3, 1)` for that premium "ease-out-expo" feel — fast start, graceful landing
- Gradient background fades in separately, starting slightly before the text

**Scroll-Triggered Reveals:**
- As each section enters the viewport, elements slide up 30-40px and fade from 0 to 1 opacity
- Use `IntersectionObserver` for triggering — animate when elements are ~20% visible
- Stagger children within sections (e.g., project cards reveal one after another with 100ms delays)
- Keep all animations SHORT: 600-800ms max. Quick and elegant, never slow and dramatic.

**Hover Micro-Interactions:**
- Buttons: smooth background color fill with a slight scale(1.02) and subtle box-shadow lift
- Project cards: gentle translateY(-4px) lift with a soft shadow expansion, maybe the accent color bleeds in as a border or overlay
- Navigation links: a small underline that slides in from left to right (using a pseudo-element with scaleX animation)
- Skill tags: subtle background color shift and slight scale
- ALL transitions use `cubic-bezier(0.4, 0, 0.2, 1)` (Material standard easing) or similar smooth curve — NEVER use `linear` or basic `ease`

**One Signature Animation:**
Add ONE delightful surprise that makes people go "wow":
- Maybe a custom cursor dot that follows the mouse with a slight delay (using transform with a spring-like transition)
- OR a subtle parallax effect where the gradient background moves slightly opposite to scroll direction
- OR project card images/colors that smoothly shift hue on hover
- Pick ONE — restraint is key

### Sections — Content Architecture

**1. Navigation Bar**
- Clean, minimal horizontal nav — logo/name on left, links on right
- Links: About, Skills, Projects, Certifications, Contact
- On scroll: add a frosted glass effect (backdrop-filter: blur + semi-transparent white background)
- Smooth scroll behavior to each section
- Mobile: hamburger menu with a smooth slide-in panel

**2. Hero Section (full viewport height)**
- Animated gradient wave background fills the entire hero
- Content left centered Photo of me is right (i will upload the images later)
- Large, bold name as the primary heading
- Role/title underneath in a lighter weight or different font style
- Short tagline below that in a slightly muted color
- One strong CTA button: "See My Work" that smooth-scrolls to projects
- Maybe a subtle scroll indicator at the bottom (a small animated chevron or "scroll" text that gently bounces)

**3. About Section**
- Asymmetric 2-column layout: text on one side (60%), a decorative element on the other (40%) — this could be a colored shape, a stylized initial letter, or an abstract accent using CSS
- Your bio paragraph with a highlighted sentence (accent color on a key phrase)
- A small detail like years of experience, location, or availability status styled as elegant pills/badges

**4. Skills Section**
- A clean heading with a thin decorative line
- Skills displayed as **styled tags/pills** in a flowing wrap layout — each pill has a subtle border, rounded corners, and hover effect
- Group them visually if possible (Frontend / Backend / Tools) using small uppercase labels
- Maybe one or two "featured" skills are slightly larger or use the accent color fill while others are outlined

**5. Projects Section**
- A 2-column responsive grid (stacks to 1 on mobile)
- Each project card has:
  - A large colored header area (use accent color tints/gradients, different shade per project — no images needed)
  - Project name in bold
  - One-line description
  - Tech stack as tiny pills/tags
  - A subtle "View Project →" link with hover animation
- Cards have rounded corners (12-16px), subtle shadow, and the hover lift effect described above
- Cards stagger their entrance animation on scroll

**6. Certifications Section**
- A clean heading like "Certifications" or "Credentials" with the same decorative line style as other sections
- Display each certification as an **elegant horizontal card/row** — think of a timeline or a clean list with presence
- Each certification shows:
  - The certification name in bold (accent-colored or strong weight)
  - The issuing organization (e.g. "Amazon Web Services", "Google") in a lighter/muted style
  - The year on the right side, styled as a subtle badge or pill
- Cards have a thin left border in the accent color (like a subtle timeline indicator) to give them visual rhythm
- On hover: the left border thickens or the card gets a soft background tint — keep it subtle and elegant
- Stagger the entrance animation on scroll, just like the project cards (each card slides up with 100ms delay between them)
- On mobile: stack everything vertically, year moves below the organization name instead of floating right

**7. Contact Section**
- A bold, large heading like "Let's Work Together" or "Get In Touch"
- Your email displayed prominently — maybe as a large, clickable link with a hover animation
- Social links as a clean horizontal row of styled icon-links (use text labels or simple SVG icons for GitHub, LinkedIn, etc.)
- A warm closing line underneath
- Maybe a subtle decorative gradient accent returns here to bookend the page

**8. Footer**
- Minimal — just a copyright line, maybe your name repeated in a lighter weight
- A "Back to Top" button that smooth-scrolls back up

### Responsive Design
- **Desktop (1024px+):** Full layouts as described
- **Tablet (768-1023px):** 2-column grids become single column, reduce heading sizes by ~20%, maintain all animations
- **Mobile (below 768px):** Single column everything, hero text reduces to 3-4rem, hamburger nav, skills wrap into 2-3 columns, project cards stack, touch-friendly spacing (min 44px tap targets)

---

## Technical Requirements

- Single `.jsx` file using React with hooks (useState, useEffect, useRef)
- Use **Tailwind CSS utility classes** for layout and spacing
- Use **inline `<style>` block** for custom CSS: keyframe animations, gradient background, font imports, grain texture, cubic-bezier transitions, and anything Tailwind can't do alone
- Import Google Fonts via `@import` in the style block
- Use `IntersectionObserver` in a `useEffect` for scroll-triggered animations
- All animations are CSS-based — no external animation libraries
- Default export the component
- Fully responsive with Tailwind breakpoints
- Smooth scrolling: add `scroll-behavior: smooth` on html

## Critical Rules — Do NOT:

- Use generic fonts: Inter, Roboto, Arial, system-ui, Space Grotesk, Poppins
- Use dark theme or dark backgrounds
- Use pure white (#FFFFFF) — always use warm off-whites
- Use purple gradients (the most overused AI aesthetic)
- Make it look like a Bootstrap template or a Tailwind UI copy-paste
- Use placeholder images or external image URLs
- Use linear easing on any animation
- Add too many animations — every motion must serve a purpose
- Use bullet points or numbered lists in the visible site content
- Make the gradient animation fast or distracting — it should be calm and mesmerizing
- Use `localStorage` or `sessionStorage`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TMR1905) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
