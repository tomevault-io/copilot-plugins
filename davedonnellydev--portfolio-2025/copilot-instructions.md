## portfolio-2025

> Outlines the step-by-step plan to build this portfolio website

# Portfolio Development Plan

Based on the guidelines, here's a structured plan to transform this Mantine template into your portfolio website:

## Phase 1: Project Structure & Foundation

### 1.1 Directory Structure Setup
```
/app
  /(pages)
    /page.tsx                    # Home
    /projects
      /page.tsx                  # Projects list
      /[slug]
        /page.tsx                # Individual project
    /about
      /page.tsx                  # About page
  /layout.tsx                    # Root layout with Header/Footer

/components
  /layout
    /Header
      /Header.tsx                # Sticky header with auto-hide
      /Header.module.css
      /Navbar.tsx                # Navigation with active state
    /Footer
      /Footer.tsx                # Sticky footer with auto-show
      /Footer.module.css

  /home
    /Hero
      /Hero.tsx
      /Hero.module.css
    /SignatureProjects
    /WhyHireMe
    /SocialProof
    /MicroBio

  /projects
    /ProjectCard
      /ProjectCard.tsx           # Reusable project card
    /ProjectFilters
    /ProjectSearch

  /shared
    /CTAButton
    /TechChip                    # Tech stack chips
    /Logo

/data
  /projects.ts                   # Project data (eventually from CMS)
  /about.ts                      # About content

/lib
  /analytics.ts                  # Analytics tracking
  /seo.ts                        # SEO utilities

/public
  /images
    /projects                    # Project screenshots
    /logos                       # Education sector logos, etc.
  /cv.pdf                        # Downloadable CV
```

### 1.2 Dependencies to Add
- `gray-matter` - for MDX/markdown if using for project content
- `reading-time` - for blog estimation
- `next-sitemap` - for sitemap generation
- Analytics (e.g., `@vercel/analytics` or Google Analytics)
- Consider: `contentlayer` or `mdx-bundler` for rich project case studies

## Phase 2: Core Layout Components

### 2.1 Header Component
**Features:**
- Logo (link to home)
- Navbar with links: Home, Projects, About, Blog (external)
- Active page highlighting
- Scroll behavior:
  - Sticky to top
  - Hide on scroll down (when past threshold)
  - Show on scroll up
  - Persist if mouse is within header
  - Auto-hide after 3s of no scroll (unless mouse inside)

**Implementation:**
- Use Mantine's `AppShell.Header` as base
- Custom scroll hook: `useScrollDirection` + `useHeaderVisibility`
- FontAwesome icons for any nav icons

### 2.2 Footer Component
**Features:**
- Email link (track click)
- Download CV button (track click)
- Location badge ("Sydney")
- Response time indicator
- GitHub icon/link (track click)
- LinkedIn icon/link (track click)

**Behavior:**
- Fixed to bottom
- Show when mouse approaches bottom viewport
- Auto-hide after 3s unless mouse inside
- Custom hook: `useFooterVisibility`

**Implementation:**
- Use FontAwesome Pro icons (Brands for GitHub/LinkedIn)
- Mantine `Group` and `ActionIcon` components

## Phase 3: Home Page Sections

### 3.1 Hero Section
- **Layout:** Grid (12-column)
- **Content:**
  - Headline (1-liner value prop)
  - Professional photo (optimized with `next/image`)
  - 2-3 credibility bullets
  - CTAs:
    - Primary: "Book 15-min intro" (link to calendar)
    - Secondary: "View top projects" (scroll to section)
    - Tertiary: "Download CV"
- **Styling:** Generous whitespace, accent color on primary CTA

### 3.2 Signature Projects Section
- **Component:** `SignatureProjects.tsx`
- **Content:** 3-4 featured projects
- **Card elements:**
  - Title
  - 1-sentence description
  - Outcome metric (highlighted)
  - Screenshot (optimized image)
  - Tech stack chips (filterable later)
  - 3 action links: "Read case study" | "Live" | "Code"
- **Footer:** "See all projects" → `/projects`
- **Layout:** Mantine Grid, responsive (1 col mobile, 2-3 desktop)

### 3.3 Why Hire Me Section
- **Content:** 3 pillars as cards or columns
  - Ship quickly
  - Communicate clearly
  - Leverage AI responsibly
- Map to role needs with icons
- Use FontAwesome icons for each pillar

### 3.4 Social Proof Section
- Education sector logos (optimized images)
- Short testimonial (quote component)
- GitHub activity sparkline (consider embedding or static image)

### 3.5 Micro Bio Section
- 2-3 sentences
- "More about me" link → `/about`

## Phase 4: Projects Page

### 4.1 Projects List Page (`/projects`)
- **Components:**
  - `ProjectFilters` - Filter by tech stack (chips)
  - `ProjectSearch` - Search by title/description
  - `ProjectCard` (grid) - Same cards as home

**Features:**
- Client-side filtering and search
- URL params for sharable filtered views (optional)
- Responsive grid

### 4.2 Individual Project Page (`/projects/[slug]`)
**Structure:**
- SEO metadata (dynamic)
- Title + 1-sentence outcome
- TL;DR block (highlighted box)
- Meta info: Role, timeframe, stack, repo link, live link
- Main content:
  - Problem section
  - Approach section
  - Result section (with metrics)
  - Architecture & key decisions (diagrams/screenshots)
  - AI usage explanation
  - Code excerpts (syntax highlighted)
  - "What I'd do next"

**Implementation:**
- Dynamic routes with `generateStaticParams`
- MDX or rich text for content
- Device frames for screenshots
- Syntax highlighting for code excerpts
- Schema.org `CreativeWork` structured data

## Phase 5: About Page

### 5.1 Content Sections
- **Mission:** Short statement (hero-style)
- **Background:** 2-3 paragraphs with timeline or visual elements
- **Certificates:** Grid of certificate cards with links
  - Certificate name
  - Issuer
  - Date
  - Link to credential
  - Optional badge image

## Phase 6: Technical Features

### 6.1 SEO Implementation
- **Per page:**
  - Dynamic metadata (title, description, OG images)
  - Canonical URLs
  - Structured data (schema.org):
    - `Person` schema (about page)
    - `CreativeWork` schema (project pages)
- **Site-wide:**
  - Sitemap.xml (use `next-sitemap`)
  - robots.txt
  - RSS feed for projects/blog

### 6.2 Performance Optimization
- Image optimization:
  - Use `next/image` everywhere
  - Optimize all screenshots
  - Lazy load below fold
- Font loading strategy (font-display: swap)
- Static generation where possible
- ISR for projects if using CMS
- Target: Lighthouse score ≥ 95

### 6.3 Accessibility
- Semantic HTML (proper heading hierarchy)
- Skip links (to main content)
- Focus states (visible, styled)
- Color contrast (WCAG AA minimum)
- Alt text for all images
- ARIA labels where needed
- Keyboard navigation

### 6.4 Analytics Setup
**Track events:**
- "Book intro" click
- Email click
- CV download
- Repo link clicks
- Live demo clicks
- Project case study views
- Filter usage on projects page

**Implementation:**
- Create `lib/analytics.ts` with tracking functions
- Use Vercel Analytics or GA4
- Privacy-friendly (no cookies if possible)

## Phase 7: Styling & Polish

### 7.1 Theme Configuration
Update `theme.ts`:
- Define accent color (use sparingly)
- 12-column grid system
- Spacing scale (generous whitespace)
- Typography scale
- Focus states
- Dark mode support (toggle already exists)

### 7.2 Micro-interactions
- Card hover states (subtle lift/shadow)
- Button hover/active states
- Smooth scroll to anchors
- Page transitions (subtle)
- Loading states
- No parallax or heavy animations

### 7.3 Component Library
Build with Mantine components:
- `Card` - for project cards, certificates
- `Grid` - 12-column responsive
- `Group` / `Stack` - for layouts
- `Button` / `ActionIcon` - CTAs
- `Badge` - tech chips, location
- `Text` / `Title` - typography
- `Image` - wrapped `next/image`
- `Anchor` - links with tracking

## Phase 8: Content & Data

### 8.1 Project Data Structure
```typescript
interface Project {
  slug: string;
  title: string;
  description: string;
  outcome: string;
  metrics?: string[];
  screenshot: string;
  techStack: string[];
  links: {
    live?: string;
    repo?: string;
    caseStudy: string;
  };
  featured: boolean;
  content?: {
    tldr: string;
    role: string;
    timeframe: string;
    problem: string;
    approach: string;
    result: string;
    architecture?: string;
    aiUsage?: string;
    codeExcerpts?: Array<{
      title: string;
      code: string;
      language: string;
    }>;
    nextSteps?: string;
  };
}
```

### 8.2 Content Sources
- Initial: TypeScript/JSON files in `/data`
- Future: Consider headless CMS (Contentful, Sanity, or MDX)

## Phase 9: Testing & Quality

### 9.1 Testing Strategy
- Component tests (Jest + React Testing Library)
- Accessibility tests (jest-axe)
- E2E tests (optional - Playwright)
- Visual regression (Storybook + Chromatic optional)

### 9.2 Quality Checks
- ESLint (already configured)
- Stylelint (already configured)
- TypeScript strict mode
- Prettier formatting
- Lighthouse CI in GitHub Actions

## Implementation Order

1. **Step 1: Foundation**
   1.1 Set up directory structure
   1.2 Create Header/Footer with scroll behavior
   1.3 Update root layout
   1.4 Define theme and accent color

2. **Step 2: Home Page**
   2.1 Hero section
   2.2 Create reusable ProjectCard component
   2.3 Signature Projects section
   2.4 Why Hire Me section
   2.5 Social Proof + Micro Bio

3. **Step 3: Projects**
   3.1 Projects list page with filtering
   3.2 Individual project page template
   3.3 Add 2-3 real projects with content

4. **Step 4: About & Polish**
   4.1 About page
   4.2 SEO implementation
   4.3 Analytics setup
   4.4 Performance optimization
   4.5 Accessibility audit

5. **Step 5: Testing & Deployment**
   5.1 Write tests for key components
   5.2 Lighthouse optimization
   5.3 Deploy to Vercel
   5.4 Monitor analytics

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davedonnellydev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davedonnellydev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
