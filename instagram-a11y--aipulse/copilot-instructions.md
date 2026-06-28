## aipulse

> cat package.json 2>/dev/null || echo "Project not initialized yet"

# AIPulse — Website Brand Guide & Build Instructions
# For Claude Code — Read this file at the start of every session

---

## 0. FIRST STEPS (every session)

```bash
# 1. Read this file completely before writing any code
# 2. Check existing project state:
ls -la
cat package.json 2>/dev/null || echo "Project not initialized yet"
# 3. Continue from where the last session ended — never restart from scratch
```

---

## 1. PROJECT OVERVIEW

| Key | Value |
|-----|-------|
| Company | AIPulse |
| Domain | aipulse.ca |
| Type | AI Consulting & Automation — Canada |
| Languages | English (default, LTR) + Farsi/Persian (RTL) |
| Feel | Luxury tech consultancy — McKinsey meets OpenAI |
| Primary tagline | *"Your vision, our intelligence, one pulse."* |
| Farsi tagline | *«چشم‌انداز شما، هوش ما، یک نبض مشترک»* |

---

## 2. TECH STACK

```
Framework:      Next.js 14 (App Router, TypeScript)
Styling:        Tailwind CSS + CSS custom properties
Animations:     Framer Motion
i18n:           next-intl (locale routing: /en/... and /fa/...)
Forms:          React Hook Form + Zod
Booking:        Cal.com embed (@calcom/embed-react)
Images:         next/image (WebP, lazy load)
Fonts:          next/font/google
Icons:          lucide-react
Deployment:     Vercel
```

### Initialize (only if project doesn't exist yet)
```bash
npx create-next-app@latest aipulse \
  --typescript --tailwind --eslint --app --src-dir=false
cd aipulse
npm install framer-motion next-intl react-hook-form zod \
  @hookform/resolvers @calcom/embed-react lucide-react
```

---

## 3. TAILWIND CONFIG

Replace `tailwind.config.ts` with exactly this:

```ts
import type { Config } from 'tailwindcss'

const config: Config = {
  content: ['./app/**/*.{ts,tsx}', './components/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        navy: {
          DEFAULT: '#0B1622',
          mid:     '#111F2E',
          soft:    '#1A2E42',
        },
        gold: {
          DEFAULT: '#C9A84C',
          light:   '#E8D08A',
          pale:    '#F5EDD0',
        },
        cream:  '#F8F7F3',
        silver: '#A8B4C0',
        ink:    '#1A2535',
      },
      fontFamily: {
        display: ['var(--font-cormorant)', 'Georgia', 'serif'],
        sans:    ['var(--font-inter)', 'Arial', 'sans-serif'],
        farsi:   ['var(--font-vazirmatn)', 'Arial', 'sans-serif'],
      },
      letterSpacing: {
        widest2: '0.3em',
        widest3: '0.4em',
      },
    },
  },
  plugins: [],
}
export default config
```

---

## 4. CSS VARIABLES

`app/globals.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --navy:        #0B1622;
  --navy-mid:    #111F2E;
  --navy-soft:   #1A2E42;
  --gold:        #C9A84C;
  --gold-light:  #E8D08A;
  --gold-pale:   #F5EDD0;
  --cream:       #F8F7F3;
  --silver:      #A8B4C0;
  --ink:         #1A2535;
  --white:       #FFFFFF;
}

/* Gold rule utility */
.gold-rule {
  height: 1px;
  background: var(--gold);
  opacity: 0.4;
}

/* Section label style */
.section-label {
  font-size: 10px;
  letter-spacing: 0.35em;
  text-transform: uppercase;
  color: var(--gold);
  font-weight: 500;
}

/* RTL support */
[dir="rtl"] {
  font-family: var(--font-vazirmatn), Arial, sans-serif;
}
[dir="rtl"] .gold-rule-left {
  margin-right: 0;
  margin-left: auto;
}
```

---

## 5. FONTS

`lib/fonts.ts`:

```ts
import { Cormorant_Garamond, Inter, Vazirmatn } from 'next/font/google'

export const cormorant = Cormorant_Garamond({
  subsets: ['latin'],
  weight: ['300', '400', '600'],
  style: ['normal', 'italic'],
  variable: '--font-cormorant',
  display: 'swap',
})

export const inter = Inter({
  subsets: ['latin'],
  weight: ['300', '400', '500'],
  variable: '--font-inter',
  display: 'swap',
})

export const vazirmatn = Vazirmatn({
  subsets: ['arabic'],
  weight: ['300', '400', '700'],
  variable: '--font-vazirmatn',
  display: 'swap',
})
```

`app/layout.tsx`:

```tsx
import { cormorant, inter, vazirmatn } from '@/lib/fonts'
import { NextIntlClientProvider } from 'next-intl'
import { getMessages, getLocale } from 'next-intl/server'

export default async function RootLayout({ children }: { children: React.ReactNode }) {
  const locale = await getLocale()
  const messages = await getMessages()
  const dir = locale === 'fa' ? 'rtl' : 'ltr'

  return (
    <html lang={locale} dir={dir}
      className={`${cormorant.variable} ${inter.variable} ${vazirmatn.variable}`}>
      <body className="bg-cream text-ink antialiased">
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  )
}
```

---

## 6. PULSE CREST LOGO COMPONENT

`components/PulseCrest.tsx`:

```tsx
'use client'
import { motion } from 'framer-motion'

interface Props {
  size?: number
  variant?: 'light' | 'dark' | 'gold'
  animate?: boolean
}

export function PulseCrest({ size = 80, variant = 'dark', animate = true }: Props) {
  const strokeColor = variant === 'gold' ? '#0B1622' : '#C9A84C'
  const fillColor   = variant === 'light' ? '#F0EFE8' : '#111F2E'
  const gemColor    = variant === 'gold' ? '#0B1622' : '#C9A84C'

  return (
    <motion.svg
      width={size}
      height={size * 1.15}
      viewBox="0 0 130 150"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
      animate={animate ? { scale: [1, 1.02, 1] } : {}}
      transition={{ duration: 3, repeat: Infinity, ease: 'easeInOut' }}
    >
      {/* Outer halo */}
      <ellipse cx="65" cy="72" rx="58" ry="62"
        stroke={strokeColor} strokeWidth="0.5" opacity="0.2"/>
      {/* Shield body */}
      <path d="M65 12 L108 30 L108 82 Q108 122 65 140 Q22 122 22 82 L22 30 Z"
        fill={fillColor} stroke={strokeColor} strokeWidth="1.4"/>
      {/* Inner shield line */}
      <path d="M65 22 L98 37 L98 80 Q98 112 65 128 Q32 112 32 80 L32 37 Z"
        fill="none" stroke={strokeColor} strokeWidth="0.4" opacity="0.35"/>
      {/* Pulse waveform */}
      <polyline
        points="34,76 46,76 53,52 60,100 66,63 72,83 79,76 96,76"
        fill="none" stroke="#C9A84C" strokeWidth="2.2"
        strokeLinecap="round" strokeLinejoin="round"/>
      {/* Top gem */}
      <circle cx="65" cy="12" r="5.5" fill={gemColor}/>
      <circle cx="65" cy="12" r="9"
        fill="none" stroke={gemColor} strokeWidth="0.5" opacity="0.4"/>
      {/* Corner gems */}
      <circle cx="108" cy="30" r="3" fill={gemColor} opacity="0.55"/>
      <circle cx="22"  cy="30" r="3" fill={gemColor} opacity="0.55"/>
    </motion.svg>
  )
}
```

---

## 7. WORDMARK COMPONENT

`components/Wordmark.tsx`:

```tsx
interface Props {
  onDark?: boolean
  size?: 'sm' | 'md' | 'lg'
}

const sizes = { sm: 'text-2xl', md: 'text-4xl', lg: 'text-6xl' }

export function Wordmark({ onDark = true, size = 'md' }: Props) {
  return (
    <span className={`font-display font-light tracking-tight ${sizes[size]}`}>
      <span className={onDark ? 'text-white' : 'text-navy'}>AI</span>
      <span className="text-gold">Pulse</span>
    </span>
  )
}
```

---

## 8. NAVBAR

`components/Navbar.tsx`:

```tsx
'use client'
import { useEffect, useState } from 'react'
import { motion, useScroll } from 'framer-motion'
import Link from 'next/link'
import { useLocale, useTranslations } from 'next-intl'
import { PulseCrest } from './PulseCrest'
import { Wordmark } from './Wordmark'

export function Navbar() {
  const [scrolled, setScrolled] = useState(false)
  const [menuOpen, setMenuOpen] = useState(false)
  const locale = useLocale()
  const t = useTranslations('nav')

  useEffect(() => {
    const handler = () => setScrolled(window.scrollY > 40)
    window.addEventListener('scroll', handler)
    return () => window.removeEventListener('scroll', handler)
  }, [])

  const navLinks = [
    { href: '/',            label: t('home')        },
    { href: '/about',       label: t('about')       },
    { href: '/services',    label: t('services')    },
    { href: '/work',        label: t('work')        },
    { href: '/appointment', label: t('appointment') },
    { href: '/contact',     label: t('contact')     },
  ]

  return (
    <motion.header
      className={`fixed top-0 left-0 right-0 z-50 transition-all duration-300 ${
        scrolled ? 'bg-navy/95 backdrop-blur-sm shadow-lg' : 'bg-transparent'
      }`}
    >
      <nav className="max-w-7xl mx-auto px-6 h-20 flex items-center justify-between">
        {/* Logo */}
        <Link href="/" className="flex items-center gap-3">
          <PulseCrest size={36} animate={false} />
          <Wordmark onDark size="sm" />
        </Link>

        {/* Desktop links */}
        <div className="hidden lg:flex items-center gap-8">
          {navLinks.map(link => (
            <Link key={link.href} href={link.href}
              className="text-silver hover:text-gold transition-colors text-sm tracking-wide
                         relative after:absolute after:bottom-0 after:left-0 after:w-0
                         after:h-px after:bg-gold after:transition-all hover:after:w-full">
              {link.label}
            </Link>
          ))}
        </div>

        {/* Right side */}
        <div className="flex items-center gap-4">
          {/* Language toggle */}
          <Link href={locale === 'en' ? '/fa' : '/en'}
            className="text-xs tracking-widest text-silver hover:text-gold transition-colors border
                       border-silver/30 hover:border-gold/50 px-3 py-1.5 rounded-sm">
            {locale === 'en' ? 'فا' : 'EN'}
          </Link>

          {/* CTA */}
          <Link href="/appointment"
            className="hidden lg:inline-flex items-center gap-2 border border-gold text-white
                       text-xs tracking-widest uppercase px-5 py-2.5 hover:bg-gold hover:text-navy
                       transition-all duration-300">
            {t('cta')}
          </Link>

          {/* Hamburger */}
          <button className="lg:hidden text-white" onClick={() => setMenuOpen(!menuOpen)}>
            <span className="block w-6 h-px bg-current mb-1.5 transition-all"/>
            <span className="block w-4 h-px bg-current mb-1.5 transition-all"/>
            <span className="block w-6 h-px bg-current transition-all"/>
          </button>
        </div>
      </nav>

      {/* Mobile menu */}
      {menuOpen && (
        <motion.div
          initial={{ opacity: 0, y: -10 }} animate={{ opacity: 1, y: 0 }}
          className="lg:hidden bg-navy border-t border-navy-soft px-6 py-8 flex flex-col gap-6">
          {navLinks.map(link => (
            <Link key={link.href} href={link.href}
              className="text-silver hover:text-gold text-lg font-display font-light"
              onClick={() => setMenuOpen(false)}>
              {link.label}
            </Link>
          ))}
          <Link href="/appointment"
            className="border border-gold text-white text-xs tracking-widest uppercase
                       px-5 py-3 text-center hover:bg-gold hover:text-navy transition-all mt-4">
            {t('cta')}
          </Link>
        </motion.div>
      )}
    </motion.header>
  )
}
```

---

## 9. REUSABLE ANIMATION COMPONENT

`components/FadeUp.tsx`:

```tsx
'use client'
import { motion } from 'framer-motion'
import { ReactNode } from 'react'

interface Props {
  children: ReactNode
  delay?: number
  className?: string
}

export function FadeUp({ children, delay = 0, className = '' }: Props) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 28 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true, margin: '-60px' }}
      transition={{ duration: 0.7, delay, ease: [0.25, 0.1, 0.25, 1] }}
      className={className}
    >
      {children}
    </motion.div>
  )
}
```

---

## 10. PAGES — FULL SPECIFICATIONS

### 10.1 HOME PAGE `app/[locale]/page.tsx`

#### Section A — Hero (full-screen)
```
Background:     bg-navy
Height:         min-h-screen
Layout:         flex flex-col items-center justify-center text-center
Content:
  1. PulseCrest (size=100, animate=true)  — centered, breathing animation
  2. Section label: "AI Consulting · Canada"  — gold, tracked
  3. H1: "Your vision, our"  — Cormorant Light 96px white
       "intelligence,"       — Cormorant Light 96px white
       "one pulse."          — Cormorant Light Italic 96px gold
  4. Farsi tagline below (Vazirmatn, fa only)
  5. Gold rule 56px wide
  6. Two CTAs side by side:
       [Book a Discovery Call]  — gold border, white text, hover fills gold
       [Our Services ↓]         — ghost, silver text, hover silver border
  7. Scroll indicator arrow — animated bounce at bottom
```

#### Section B — Intro Strip
```
Background:   bg-navy-mid
Padding:      py-16
Content:      Single centered line — Cormorant 36px white italic:
              "We don't sell software. We engineer intelligence."
              Gold rule 40px below it
```

#### Section C — Services Grid
```
Background:   bg-cream
Heading:      "What we build for you" — Cormorant 52px navy
Subheading:   Section label above heading
Grid:         3 columns desktop, 1 mobile, gap-6
6 cards:
  [AI Agents]             — icon: ⬡  — "Autonomous agents trained on your data"
  [Workflow Automation]   — icon: ⟳  — "End-to-end process automation"
  [AI Web Applications]   — icon: ◈  — "Intelligent web apps with AI built in"
  [Cloud Solutions]       — icon: △  — "Scalable, secure AI infrastructure"
  [AI Strategy]           — icon: ◎  — "Discovery sessions and AI roadmaps"
  [Custom Applications]   — icon: ◇  — "Mobile and desktop AI-powered apps"

Card style:
  bg-white border-b-2 border-transparent hover:border-gold
  p-8, rounded-none (sharp corners = luxury)
  Icon: text-gold text-2xl mb-4
  Title: font-display text-xl text-navy mb-3
  Body: text-sm text-silver leading-relaxed
  Hover: translateY(-4px) transition-all duration-300 shadow-lg
```

#### Section D — Why AIPulse (3-column trust)
```
Background:   bg-navy
Padding:      py-24
3 columns:
  [Custom, not generic]     — "Every solution built from scratch for your business"
  [Discovery first]         — "We listen before we propose or build anything"
  [Long-term partnership]   — "We stay with you beyond launch"
Each column:
  Gold number (01, 02, 03) — font-display text-5xl opacity-20
  Gold rule 32px
  Title — font-display text-2xl text-white
  Body — text-sm text-silver leading-loose
```

#### Section E — Founder Photo Section
```
Background:   bg-cream
Layout:       2-column — image left (60%), text right (40%)
Image:        /public/images/founder-hero.jpg
              Placeholder if missing: bg-navy-mid h-[500px] with gold "Photo Coming Soon"
Text right:
  Section label: "The founder"
  Name: font-display text-4xl navy (client to provide name)
  Title: "Founder & AI Consultant, AIPulse"
  Bio: 2–3 sentences (see About section below)
  Gold rule
  Link: "Read the full story →" — gold, tracked
```

#### Section F — CTA Banner
```
Background:   bg-navy  with gold radial gradient at center
Padding:      py-28
Content:
  PulseCrest size=60, centered
  H2: "Ready to build your intelligent future?"
  Farsi line (fa mode only)
  Button: [Book a Free 30-Min Discovery Call] — gold fill, navy text, large
```

---

### 10.2 ABOUT PAGE `app/[locale]/about/page.tsx`

```
Section 1 — Hero
  Full-width image: /public/images/founder-hero.jpg (cover, h-[70vh])
  Dark overlay: bg-navy/60
  Centered text over image:
    Section label: "About AIPulse"
    H1: "Built on precision. Driven by partnership." — font-display white

Section 2 — Story
  2-column: text-heavy left, portrait image right
  Portrait: /public/images/founder-bio.jpg (rounded-none, h-[500px] object-cover)
  Text:
    Section label: "Our Story"
    H2: "The intelligence your business deserves." — font-display navy
    P1: AIPulse was founded in Canada with a single conviction: every business
        deserves AI that is engineered for it — not adapted from a generic template.
    P2: Every engagement begins with a discovery session. We ask about your operations,
        your team, your goals, and the gaps between where you are and where you want to be.
        Only then do we design and build.
    P3: The result is an AI system that fits your business precisely — and a partnership
        that continues long after launch.
    Farsi version of the above (messages/fa.json)

Section 3 — Values (4 cards)
  Background: bg-cream
  4 cards in a 2×2 grid:
    [Precision]     "We engineer, not approximate. Every detail is intentional."
    [Custom-built]  "No templates. No recycled logic. Built for you, only."
    [Partnership]   "We stay engaged well beyond the initial delivery."
    [Long-term]     "We measure success in your growth, not in project completions."
  Card style: bg-white border-t-2 border-gold p-8

Section 4 — Company Facts strip
  Background: bg-navy
  3 stats centered:
    [2024]          "Year founded"
    [Canada]        "Headquarters"
    [AI + Cloud]    "Core expertise"

Section 5 — CTA → Appointment
  "Let's talk about your business" — font-display 48px white
  Button → /appointment
```

---

### 10.3 SERVICES PAGE `app/[locale]/services/page.tsx`

```
Hero:
  bg-navy, centered
  H1: "What we build for you" — font-display 72px white
  Tagline below

6 Service Detail Sections (alternating layout):
  Each section: 2-column — icon+title+description left, use-cases right (or flip)
  Icon: large SVG or lucide icon, gold color, 48px
  Title: font-display 40px
  Description: 3–4 sentences
  Use cases: bulleted list, gold bullet "—", text-silver text-sm
  EN + FA text for each

Process Section:
  Background: bg-cream
  Title: "How we work" — font-display 52px navy
  4-step horizontal flow (desktop) / vertical (mobile):
    [01 Discovery] → [02 Design] → [03 Build] → [04 Deploy]
    Each step: gold number, title, 1-line description
    Connected by gold dashed line (desktop)
  Animated: each step fades in with 0.15s stagger on scroll

Pricing note:
  bg-navy-mid, centered, italic font-display 24px white:
  "Every project is scoped and priced custom — no off-the-shelf plans."

CTA → /appointment
```

---

### 10.4 APPOINTMENT PAGE `app/[locale]/appointment/page.tsx`

```
Hero:
  bg-navy, min-h-[40vh]
  PulseCrest size=60
  H1: "Let's Talk" / «بیایید صحبت کنیم»
  Subtitle: "Book a free 30-minute discovery session."

Booking embed:
  <Cal namespace="discovery" calLink="aipulse/discovery" ... />
  Placeholder if Cal not yet configured:
    bg-navy-mid border border-gold/20 p-12 text-center
    "Booking calendar coming soon — email us at hello@aipulse.ca"

What to expect (3 points below embed):
  — We'll ask about your business and goals
  — We'll share where AI can create real value for you
  — No sales pitch — just an honest conversation

Contact fallback:
  hello@aipulse.ca
  LinkedIn: /in/aipulse (update with real link)
  Response time: "We respond within 24 hours."
```

---

### 10.5 CONTACT PAGE `app/[locale]/contact/page.tsx`

```
2-column layout:
  Left: contact info (email, LinkedIn, location, response time)
  Right: contact form

Form fields:
  - Full name (required)
  - Email (required, validated)
  - Company name
  - Message (textarea, required)
  - Language preference: [English] [Farsi] radio

Form validation: React Hook Form + Zod
Submit action: send to API route /api/contact → email (use Resend or Nodemailer)

Style:
  Inputs: bg-transparent border-b border-silver/30 focus:border-gold
          text-ink pb-3 outline-none w-full transition-colors
  Submit: full-width gold border button → hover fills gold
```

---

## 11. TRANSLATION FILES

### `messages/en.json`
```json
{
  "nav": {
    "home": "Home",
    "about": "About",
    "services": "Services",
    "work": "Work",
    "appointment": "Appointment",
    "contact": "Contact",
    "cta": "Book a Call"
  },
  "hero": {
    "label": "AI Consulting · Canada",
    "line1": "Your vision, our",
    "line2": "intelligence,",
    "line3": "one pulse.",
    "tagline": "Your vision, our intelligence, one pulse.",
    "cta_primary": "Book a Discovery Call",
    "cta_secondary": "Our Services"
  },
  "about": {
    "label": "About AIPulse",
    "hero_title": "Built on precision. Driven by partnership.",
    "story_label": "Our Story",
    "story_heading": "The intelligence your business deserves."
  },
  "services": {
    "label": "What we offer",
    "heading": "What we build for you",
    "process_heading": "How we work",
    "pricing_note": "Every project is scoped and priced custom — no off-the-shelf plans."
  },
  "appointment": {
    "label": "Get started",
    "heading": "Let's Talk",
    "sub": "Book a free 30-minute discovery session.",
    "expect1": "We'll ask about your business and goals",
    "expect2": "We'll share where AI can create real value for you",
    "expect3": "No sales pitch — just an honest conversation"
  },
  "contact": {
    "heading": "Get in Touch",
    "response": "We respond within 24 hours."
  },
  "footer": {
    "tagline": "Your vision, our intelligence, one pulse.",
    "copy": "© 2025 AIPulse Inc. All rights reserved."
  }
}
```

### `messages/fa.json`
```json
{
  "nav": {
    "home": "خانه",
    "about": "درباره ما",
    "services": "خدمات",
    "work": "نمونه کارها",
    "appointment": "رزرو وقت",
    "contact": "تماس",
    "cta": "رزرو جلسه"
  },
  "hero": {
    "label": "مشاوره هوش مصنوعی · کانادا",
    "line1": "چشم‌انداز شما،",
    "line2": "هوش ما،",
    "line3": "یک نبض مشترک.",
    "tagline": "چشم‌انداز شما، هوش ما، یک نبض مشترک",
    "cta_primary": "رزرو جلسه اکتشافی",
    "cta_secondary": "خدمات ما"
  },
  "about": {
    "label": "درباره AIPulse",
    "hero_title": "بنا شده بر دقت. هدایت‌شده با مشارکت.",
    "story_label": "داستان ما",
    "story_heading": "هوشی که کسب‌وکار شما لایقش است."
  },
  "services": {
    "label": "آنچه ارائه می‌دهیم",
    "heading": "چه می‌سازیم",
    "process_heading": "روش کار ما",
    "pricing_note": "هر پروژه به‌صورت سفارشی قیمت‌گذاری می‌شود — هیچ پلن از پیش آماده‌ای وجود ندارد."
  },
  "appointment": {
    "label": "شروع کنید",
    "heading": "بیایید صحبت کنیم",
    "sub": "یک جلسه اکتشافی ۳۰ دقیقه‌ای رایگان رزرو کنید.",
    "expect1": "در مورد کسب‌وکار و اهداف شما می‌پرسیم",
    "expect2": "می‌گوییم هوش مصنوعی کجا برای شما ارزش واقعی ایجاد می‌کند",
    "expect3": "بدون فروش — فقط یک گفتگوی صادقانه"
  },
  "contact": {
    "heading": "در تماس باشید",
    "response": "ظرف ۲۴ ساعت پاسخ می‌دهیم."
  },
  "footer": {
    "tagline": "چشم‌انداز شما، هوش ما، یک نبض مشترک",
    "copy": "© ۲۰۲۵ AIPulse Inc. تمامی حقوق محفوظ است."
  }
}
```

---

## 12. PHOTOS — PLACEHOLDER SYSTEM

```tsx
// components/PhotoPlaceholder.tsx
interface Props {
  label: string
  aspectRatio?: string
  className?: string
}

export function PhotoPlaceholder({ label, aspectRatio = 'aspect-[4/3]', className = '' }: Props) {
  return (
    <div className={`bg-navy-mid flex items-center justify-center ${aspectRatio} ${className}`}>
      <div className="text-center">
        <div className="w-10 h-px bg-gold mx-auto mb-4 opacity-50" />
        <p className="text-gold/60 text-xs tracking-widest uppercase">{label}</p>
        <div className="w-10 h-px bg-gold mx-auto mt-4 opacity-50" />
      </div>
    </div>
  )
}
```

**When real photos are provided by client:**
- Place in `/public/images/`
- `founder-hero.jpg` — landscape, full-width hero (min 1600×900)
- `founder-bio.jpg`  — portrait, bio section (min 600×900)
- `founder-card.jpg` — square or portrait, homepage card (min 500×500)
- Replace `<PhotoPlaceholder>` with `<Image src="/images/..." fill alt="..." />`
- Always wrap in `<div className="relative overflow-hidden">` for `fill` images

---

## 13. FOOTER COMPONENT

`components/Footer.tsx`:

```tsx
import Link from 'next/link'
import { PulseCrest } from './PulseCrest'
import { useTranslations } from 'next-intl'

export function Footer() {
  const t = useTranslations('footer')
  const links = [
    { href: '/',            label: 'Home'        },
    { href: '/about',       label: 'About'       },
    { href: '/services',    label: 'Services'    },
    { href: '/appointment', label: 'Appointment' },
    { href: '/contact',     label: 'Contact'     },
  ]

  return (
    <footer className="bg-navy border-t border-gold/20">
      <div className="max-w-6xl mx-auto px-6 py-20">
        <div className="flex flex-col items-center text-center gap-6 mb-16">
          <PulseCrest size={48} animate={false} />
          <span className="font-display text-3xl font-light text-white">
            AI<span className="text-gold">Pulse</span>
          </span>
          <p className="font-display italic font-light text-white/50 text-lg">
            "{t('tagline')}"
          </p>
          <p className="text-gold/60 text-xs tracking-widest">aipulse.ca</p>
        </div>

        <div className="flex flex-wrap justify-center gap-8 mb-12">
          {links.map(l => (
            <Link key={l.href} href={l.href}
              className="text-silver hover:text-gold text-xs tracking-widest uppercase transition-colors">
              {l.label}
            </Link>
          ))}
        </div>

        <div className="h-px bg-gold/10 mb-8" />
        <p className="text-white/20 text-xs text-center tracking-wide">{t('copy')}</p>
      </div>
    </footer>
  )
}
```

---

## 14. ANIMATION RULES

| Element | Animation | Settings |
|---------|-----------|----------|
| All section headings | fade up | y: 28, duration: 0.7s |
| Cards | fade up with stagger | delay: i * 0.1s |
| Gold rules | slide in from left/right | scaleX: 0→1, origin: left |
| Pulse Crest (hero) | breathing scale | scale 1→1.02→1, 3s loop |
| Navbar | bg on scroll | bg-transparent → bg-navy/95 |
| CTA button hover | fill transition | border-fill 0.3s |
| Page transitions | fade | opacity 0→1, 0.4s |
| Service cards hover | lift + border | translateY -4px, border-gold |

**Never:** parallax on mobile, autoplay video, WebGL, CSS filters on text

---

## 15. BRAND RULES (enforced in code)

1. Name is always `AIPulse` — one word. Check string literals in all files.
2. Gold max 15% of any composition — never as a large background fill.
3. Cormorant always weight 300 for display, 400 for subheadings, 600 for callouts.
4. All Farsi text: `font-farsi` class, `dir="rtl"` on parent container.
5. Approved backgrounds: `#0B1622`, `#111F2E`, `#F8F7F3`, `#FFFFFF` only.
6. Sharp corners only — `rounded-none` on cards, inputs, buttons (luxury aesthetic).
7. Buttons: outlined gold by default → fills on hover. Never solid fills as default state.
8. No drop shadows on text. Subtle `shadow-lg` only on hovered cards.

---

## 16. SEO

`app/[locale]/layout.tsx` metadata:

```ts
export const metadata = {
  title: { default: 'AIPulse — AI Consulting & Automation | Canada', template: '%s | AIPulse' },
  description: 'Custom AI agents, automation, and intelligent applications built precisely for your business. Based in Canada. Your vision, our intelligence, one pulse.',
  openGraph: {
    title: 'AIPulse',
    description: 'Your vision, our intelligence, one pulse.',
    url: 'https://aipulse.ca',
    siteName: 'AIPulse',
    images: [{ url: '/og-image.png', width: 1200, height: 630 }],
    locale: 'en_CA',
  },
  twitter: { card: 'summary_large_image' },
  robots: { index: true, follow: true },
}
```

---

*AIPulse Brand Guide & Build Instructions — v2.0 — aipulse.ca*
*Read this file completely before starting any build session.*

---
> Source: [instagram-a11y/aipulse](https://github.com/instagram-a11y/aipulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
