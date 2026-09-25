## manixh

> > Use this repo as a **theme template** to build a new portfolio for a different person.

# AGENTS.md — AI Coding Agent Operating Manual

> Use this repo as a **theme template** to build a new portfolio for a different person.
> Cross-reference: [DESIGN.md](./DESIGN.md), [COMPONENTS.md](./COMPONENTS.md), [ARCHITECTURE.md](./ARCHITECTURE.md), [CONTENT_SCHEMA.md](./CONTENT_SCHEMA.md)

## Summary

This repo is a React 19 + Vite personal portfolio with a dark-mode-only design system. The visual theme (colors, typography, spacing, animations) is fully separated from personal content. To build a new portfolio, **replace content only** — never touch styling or component structure.

---

## What to KEEP Unchanged

These define the visual identity. Do not modify:

- **Color palette** — all values in `DESIGN.md` (e.g. `#0b0d0e` background, `#1a1b1c` surface, `#b3b3b3` text)
- **Typography** — Figtree for UI, JetBrains Mono for code, system font stack for body
- **Spacing system** — 700px max-width containers, 18px padding, responsive breakpoints at 700/420/400/360/354px
- **Border radius** — 2px–10px scale, 999px pills, 50% circles
- **Shadows** — composite multi-layer shadows on buttons/pills
- **Animation** — 60s marquee, heart-beat, tooltip spring easing, loading SVG
- **Component structure** — keep all files in `src/components/`, `src/data/`, `src/pages/`
- **Section order on home page** — Hero → Skills → Experience → Projects → Uses → Analytics → Contact → Footer
- **Blog component system** — all 15 blog components in `src/components/blogs/components/`

---

## What to REPLACE

Replace all personal content listed below. **All content files are in `src/data/` or hardcoded in components.**

### 1. Personal Identity

| What | Where | How |
|------|-------|-----|
| Full name | `src/components/heroSection/HeroSection.tsx` line ~105 | Replace `"Manish Kumar"` |
| Username/handle | `HeroSection.tsx` line ~113 | Replace `@manixh02` and URL |
| Profile photo | `public/images/profile/pfp-latest.jpg` | Replace file (keep name) |
| QR code | `public/images/profile/qr-code.png` | Replace file (keep name) |
| Bio lines | `HeroSection.tsx` lines ~174-196 | Replace the 3 `<li>` items |
| Location | `HeroSection.tsx` line ~148 | Replace `"India"` |
| Status text | `HeroSection.tsx` line ~138 | Replace `"Building ossium.in"` + logo URL |
| Footer quote | `src/components/footer/Footer.tsx` | Replace `"Nothing Is Perfect..."` |
| Footer attribution | `Footer.tsx` | Replace `"Designed & Made with ❤️"` |
| Loading text | `src/components/loading/Loading.tsx` | Replace `"Just a second babe..."` |

### 2. Social Links

Edit `src/data/socialLinks.tsx`:

```typescript
export const socialLinks: SocialLink[] = [
  { name: 'Twitter', url: 'https://x.com/YOUR_HANDLE', icon: <FaTwitter />, color: '#1DA1F2' },
  { name: 'LinkedIn', url: 'https://linkedin.com/in/YOUR_ID', icon: <FaLinkedin />, color: '#0077B5' },
  { name: 'GitHub', url: 'https://github.com/YOUR_USERNAME', icon: <FaGithub />, color: '#333' },
  // ... add/remove as needed
];
```

### 3. Projects

Edit `src/data/projects.ts`:

- Replace `featuredProjects` array (shown on home page)
- Replace `additionalProjects` array (shown on /projects page)
- Replace project screenshot images in `public/images/projects/`
- Update `userImages.projects` in `src/data/images.ts`

### 4. Skills & Tools

Edit `src/components/skillSection/SkillSection.tsx`:

- Replace the `technologies` array (15 items)
- Replace the `tools` array (15 items)
- Each item: `{ name: string, icon: IconType, color: string }`

### 5. Experience

Edit `src/components/experience/Experience.tsx`:

- Replace the hardcoded experience entries
- Each entry: company, role, dates, status, logoUrl, description[], links[]

### 6. Uses (Tools & Gear)

Edit `src/data/uses.ts`:

- Replace both categories (Software + Hardware)
- Each item: `{ label, name, link? }`

### 7. GitHub Calendar

Edit `src/components/calendar/Calendar.tsx`:

- Replace `username="ig-imanish"` with new GitHub username

### 8. Blog Content

- Edit `src/pages/blogs/BlogLayout.tsx` for blog listing
- Create new blog posts as new files in `src/pages/blogs/`
- Edit `BlogAuthor.tsx` defaults if author name/avatar changes

### 9. Navbar

Edit `src/components/navbar/Navbar.tsx`:

- Replace GitHub repo URL in the star link
- Replace nav link labels if needed

### 10. SEO & Meta

Edit `index.html`:

- All `<title>`, `<meta>` tags
- Open Graph tags (`og:title`, `og:description`, `og:image`, `og:url`)
- Twitter Card tags
- JSON-LD structured data
- Canonical URL
- Replace OG image in `public/og/og-image.png`
- Replace favicon URL
- Replace resume PDF in `public/resume/resume.pdf`

### 11. Contact

Edit `src/components/heroSection/HeroSection.tsx`:

- Replace `mailto:work.manishh02@gmail.com` with new email
- Replace Discord invite link + server image
- Replace Twitter card data

---

## Step-by-Step Build Instructions

### For AI Agents

```
1. READ these files first:
   - DESIGN.md (understand the visual system)
   - COMPONENTS.md (understand all components)
   - ARCHITECTURE.md (understand structure)
   - CONTENT_SCHEMA.md (understand data shapes)

2. ASK the user for their content:
   - Full name and username/handles
   - Bio (2-3 short sentences)
   - Profile photo
   - Social links (Twitter, LinkedIn, GitHub, Instagram, etc.)
   - Projects (name, description, tech stack, URLs, screenshots)
   - Skills and tools (categorized)
   - Work experience (company, role, dates, description)
   - Resume PDF
   - Contact email
   - GitHub username (for calendar)

3. REPLACE content in data files ONLY:
   - src/data/projects.ts → projects
   - src/data/socialLinks.tsx → social links
   - src/data/uses.ts → tools & gear
   - src/data/images.ts → image paths

4. REPLACE hardcoded content in components:
   - HeroSection.tsx → name, bio, status, email, social cards
   - SkillSection.tsx → technologies and tools arrays
   - Experience.tsx → experience entries
   - Footer.tsx → quote text
   - Calendar.tsx → GitHub username
   - Navbar.tsx → repo URL

5. UPDATE meta tags:
   - index.html → title, description, OG, Twitter, JSON-LD
   - public/og/og-image.png → replace with new OG image
   - public/resume/resume.pdf → replace with new resume

6. REPLACE images:
   - public/images/profile/ → avatar, QR code
   - public/images/projects/ → project screenshots
   - public/images/decorations/ → keep or replace

7. VALIDATE:
   - Run: pnpm typecheck
   - Run: pnpm build
   - Run: pnpm lint
   - Verify all colors/fonts/spacing match DESIGN.md
   - Test on mobile (354px, 360px, 400px, 700px)

8. DO NOT:
   - Change any CSS files
   - Change component props/interfaces
   - Change the routing structure
   - Change animation values
   - Change color tokens
   - Add new dependencies without checking if one already exists
   - Use inline styles for new styling (use existing CSS classes)
```

---

## Constraints / Guardrails

| Rule | Why |
|------|-----|
| **Never modify CSS files** | The design system is complete and consistent |
| **Never change color values** | Dark theme is carefully balanced |
| **Never remove animations** | They define the personality |
| **Never restructure components** | The hierarchy is intentional |
| **Never hardcode content inside components** | Use data files when possible |
| **Never add new npm packages** | The dependency list is minimal and curated (`echarts` was added for the analytics feature) |
| **Never change the 700px container width** | It's the core layout constraint |
| **Never remove responsive breakpoints** | All screen sizes are covered |
| **Never use `any` type** | TypeScript strict mode is enforced |
| **Never commit `.env`** | Secrets must stay local |

---

## Example Prompt for AI Agent

> "Use this repo as a template to build my personal portfolio. Here's my info:
>
> **Name:** [Your Name]
> **Username:** [your_handle]
> **Bio:** [2-3 sentences about yourself]
> **Role:** [Your job title]
> **Location:** [City, Country]
> **Email:** [your@email.com]
>
> **Social Links:**
> - Twitter: [url]
> - LinkedIn: [url]
> - GitHub: [url]
> - Instagram: [url]
>
> **Projects:**
> 1. [Name] — [Description] — [Tech stack] — [GitHub URL] — [Live URL]
> 2. [Name] — [Description] — [Tech stack] — [GitHub URL]
>
> **Skills:** [List your technologies and tools]
>
> **Experience:**
> 1. [Company] — [Role] — [Dates] — [Description bullets]
>
> **GitHub Username:** [for contribution calendar]
>
> Build my portfolio using the exact same theme, animations, and layout. Only replace the content — don't touch any CSS or component structure."

---
> Source: [ig-imanish/manixh](https://github.com/ig-imanish/manixh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
