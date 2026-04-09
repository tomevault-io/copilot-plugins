## meridian

> 1. **Repository Configuration**

# Project Meridian Blog - Development Notes

## 🎯 What We Accomplished

### Initial Setup & Foundation
1. **Repository Configuration**
   - Created git repository properly rooted at `/Users/jasonwilliamson/Desktop/meridianblog`
   - Configured remote for `https://github.com/jw41784/meridian.git`
   - Set up GitHub Pages deployment with Actions

2. **Astro Foundation**
   - Astro v4.16 with TypeScript strict mode
   - Tailwind CSS with custom brand colors (#4A2532, #607D8B, #E0E4E8)
   - MDX support for rich content authoring
   - React integration for interactive components

3. **Core Features Implemented**
   - ✅ Content Collections with type-safe schema
   - ✅ File-based routing (/articles, /tags, /about)
   - ✅ Client-side search with Lunr.js
   - ✅ RSS feed generation
   - ✅ XML sitemap for SEO
   - ✅ Responsive navigation with search
   - ✅ Article cards with tag system
   - ✅ GitHub Actions CI/CD pipeline
   - ✅ URL helper utility for proper BASE_URL handling

### Content & Brand Development (May 2025)
4. **Page Creation & Content**
   - ✅ Created About, Work, Contact pages with engaging copy
   - ✅ Refined brand voice to be human-centered and conversational
   - ✅ Balanced confidence with approachability (Culture Amp/TikTok style)
   - ✅ Added comprehensive transformational leadership article (3000+ words)
   - ✅ Proper academic citations with References sections

5. **Navigation & Site Structure**
   - ✅ Simplified navigation to focus on core content (Home, Articles, About)
   - ✅ Hidden Work/Contact pages to streamline user experience
   - ✅ Added contact email (info@eudexio.com) throughout articles
   - ✅ Updated all CTAs to use email instead of contact forms
   - ✅ Clean footer with working links only

6. **Technical Improvements**
   - ✅ Fixed navigation link issues with proper URL concatenation
   - ✅ Updated article dates to May 2025
   - ✅ Added explanations for technical references (AlphaGo, Move 37)
   - ✅ Removed business development tools to focus on thought leadership

### Visual Identity & Brand Integration (May 26, 2025)
7. **Logo Integration & Visual Identity**
   - ✅ Added Meridian compass logo to navigation (32px height)
   - ✅ Integrated logo in footer center (48px height)
   - ✅ Updated favicon to use Meridian compass logo
   - ✅ Implemented proper cache-busting for favicon updates
   - ✅ Fixed favicon URL paths using getUrl() helper for GitHub Pages

8. **Brand Messaging & Philosophy**
   - ✅ Rewrote homepage to emphasize "finding true north" concept
   - ✅ Integrated meridian line metaphor throughout messaging
   - ✅ Positioned against "best practice" as status quo/consensus thinking
   - ✅ Updated about page to explain meridian philosophy and navigation principles
   - ✅ Emphasized context over convention, values-based leadership
   - ✅ Added "Question the Given," "Context Over Convention," "Navigate by Values" principles

### Content & Feature Enhancements (June 1, 2025)
9. **New Content Addition**
   - ✅ Added "The Vanishing Point" article (2,640 words) about organizational memory
   - ✅ Explores antimemetic knowledge and recursive forgetting
   - ✅ Formatted in MDX with proper metadata and tags
   - ✅ Integrated into the content collection successfully

10. **Dark Mode Implementation**
   - ✅ Configured Tailwind CSS with class-based dark mode (`darkMode: 'class'`)
   - ✅ Created ThemeToggle React component with localStorage persistence
   - ✅ Added theme initialization script to prevent flash of unstyled content
   - ✅ Respects system color scheme preference as default
   - ✅ Updated all components with comprehensive dark mode styles
   - ✅ Fixed article prose text to use light gray (gray-300) in dark mode
   - ✅ Custom typography configuration for both light and dark themes

11. **Reading Time Feature**
   - ✅ Created reading time calculation utility (200 words per minute)
   - ✅ Strips MDX/Markdown formatting for accurate word count
   - ✅ Displays reading time on all article cards and detail pages
   - ✅ Format: "X min read" (singular/plural handled correctly)
   - ✅ Integrated across homepage, articles page, and tag pages

12. **Table of Contents for Long Articles**
   - ✅ Interactive TOC component that extracts H2, H3, H4 headings
   - ✅ Only shows for articles > 5 minutes reading time
   - ✅ Desktop: Sticky sidebar in left column (3/12 grid layout)
   - ✅ Mobile: Collapsible details element with smooth arrow animation
   - ✅ Active section highlighting with left border indicator
   - ✅ Smooth scrolling with offset for fixed header
   - ✅ Independent scrolling with custom styled scrollbar
   - ✅ Max heights: Desktop (viewport - 10rem), Mobile (24rem)
   - ✅ Automatically generates IDs for headings without them

### Visual Design & Code Quality Enhancements (January 10, 2025)
13. **Prettier Integration**
   - ✅ Installed Prettier with Astro and Tailwind plugins
   - ✅ Created `.prettierrc` configuration file
   - ✅ Added `.prettierignore` for excluded files
   - ✅ Added npm scripts: `format` and `format:check`
   - ✅ Formatted entire codebase for consistency

14. **Enhanced Visual Design**
   - ✅ **Article Cards**: Modern hover effects, image zoom, author avatars with initials
   - ✅ **Navigation**: Glassmorphism effect, sticky header, active link indicators
   - ✅ **Footer**: Gradient backgrounds, email CTA button, improved link animations
   - ✅ **Homepage**: Animated background blobs, fade-in/slide-up animations, larger typography
   - ✅ **Buttons**: Enhanced with gradient overlays and shadow effects
   - ✅ **Tags**: Changed to rounded pills with hashtags and hover scaling

15. **Animation System**
   - ✅ Added CSS keyframe animations (float, fadeIn, slideUp)
   - ✅ Implemented staggered animations for article cards
   - ✅ Added hover lift effects throughout
   - ✅ Smooth transitions on all interactive elements
   - ✅ Hardware-accelerated transforms for performance

16. **Typography & Spacing Improvements**
   - ✅ Enhanced article page metadata display with icons
   - ✅ Improved prose styling with better link colors
   - ✅ Better visual hierarchy with font weights and sizes
   - ✅ Consistent spacing using Tailwind utilities
   - ✅ Line-clamp for article descriptions

## 🚀 Recommended Next Steps

### Phase 1: Enhanced User Experience (Week 1-2)

1. **Dark Mode Support** ✅ COMPLETED
   - ✅ Configured Tailwind with `darkMode: 'class'`
   - ✅ Created theme toggle component with sun/moon icons
   - ✅ Stored preference in localStorage
   - ✅ Added comprehensive dark mode color variants

2. **Reading Experience** (PARTIALLY COMPLETED)
   - ✅ Added reading time estimation (200 WPM)
   - ✅ Implemented table of contents for long articles
   - ⏳ Add "Back to top" button
   - ⏳ Create related articles section

3. **Performance Optimizations** (PENDING)
   - ⏳ Implement image optimization with Astro's `<Image>` component
   - ⏳ Add lazy loading for article cards
   - ⏳ Configure Partytown for analytics scripts
   - ⏳ Implement view transitions between pages

### Phase 2: Email Collection & Content Management (Week 3-4)

1. **Newsletter Integration** (PRIORITY)
   - **Recommended Service**: ConvertKit (~$29/month for <1k subscribers)
     - Built for creators/bloggers, great automation
     - RSS-to-email automation for new post notifications
     - Professional forms that match brand
   - **Alternative**: Buttondown ($9/month, developer-friendly)
   - **Implementation**:
     - Add signup forms to articles footer and homepage
     - Connect RSS feed for automatic new post emails
     - Create welcome email sequence
     - Add unobtrusive popup/banner after reading time
   - Add newsletter archive page

2. **Author Profiles**
   - Create author collection in content/
   - Add author bio pages
   - Display author info on articles
   - Create author archive pages

3. **Content Features**
   - Draft preview system with URL parameters
   - Series/collection support for related articles
   - Code syntax highlighting themes
   - Copy code button for code blocks

### Phase 3: Analytics & SEO (Week 5-6)

1. **Analytics Setup**
   - Integrate Plausible/Fathom Analytics
   - Add custom events for search, clicks
   - Create analytics dashboard page
   - Track popular content

2. **Advanced SEO**
   - Implement JSON-LD structured data
   - Add Open Graph images per article
   - Create XML sitemap for images
   - Implement breadcrumb navigation

3. **Social Features**
   - Add social sharing buttons
   - Implement Twitter/X cards
   - Create social media preview tool
   - Add LinkedIn article format

### Phase 4: Advanced Features (Month 2)

1. **Comments System**
   - Integrate Giscus (GitHub Discussions)
   - Or build custom with Supabase
   - Add moderation capabilities
   - Create comment notifications

2. **Search Enhancements**
   - Add search filters (date, author, tags)
   - Implement search suggestions
   - Add keyboard shortcuts (cmd+k)
   - Create search analytics

3. **API Development**
   - Create JSON API endpoints
   - Add pagination support
   - Implement content webhooks
   - Create integration documentation

### Phase 5: Scaling & Optimization (Month 3)

1. **Multi-language Support**
   - Implement i18n routing
   - Create translation workflow
   - Add language switcher
   - Localize dates/times

2. **Performance Monitoring**
   - Set up Lighthouse CI
   - Add bundle size tracking
   - Implement error tracking (Sentry)
   - Create performance dashboard

3. **Content Automation**
   - Auto-generate social media posts
   - Create content calendar
   - Implement scheduled publishing
   - Add content versioning

## 🔧 Technical Debt & Maintenance

### Immediate Priorities
1. **Add missing meta tags for articles**
   - Twitter card images
   - Article-specific descriptions
   - Canonical URLs for all pages

2. **Improve build process**
   - Add pre-commit hooks (Husky)
   - Implement linting (ESLint, Prettier)
   - Add commit message standards
   - Create PR templates

3. **Testing Strategy**
   - Add Playwright for E2E tests
   - Implement visual regression tests
   - Create content validation tests
   - Add accessibility tests

### Code Quality
1. **Component Library**
   - Extract reusable components
   - Create Storybook documentation
   - Add component tests
   - Document component APIs

2. **TypeScript Improvements**
   - Add stricter type checking
   - Create type utilities
   - Document type patterns
   - Add JSDoc comments

## 📊 Success Metrics

Track these KPIs:
- Page load time < 1s
- Lighthouse score > 95
- Search success rate > 80%
- Zero runtime errors
- SEO visibility growth
- User engagement metrics

## 🛠️ Development Commands

```bash
# Development
npm run dev          # Start development server
npm run build        # Build for production
npm run preview      # Preview production build

# Code Formatting
npm run format       # Format all files with Prettier
npm run format:check # Check formatting without changes

# Future Commands (to be implemented)
npm run lint         # Run linting
npm run typecheck    # Run TypeScript checks
npm run test         # Run tests
npm run test:e2e     # Run E2E tests
npm run analyze      # Check bundle size
```

## 🔐 Security Considerations

1. Add Content Security Policy headers
2. Implement rate limiting for API endpoints
3. Add CORS configuration
4. Set up dependency scanning
5. Create security.txt file

## 💡 Innovation Ideas

1. **AI Integration**
   - Auto-generate article summaries
   - Smart content recommendations
   - Semantic search capabilities
   - Writing assistant for authors

2. **Interactive Features**
   - Live code playgrounds
   - Interactive diagrams
   - Polls and surveys
   - Real-time collaboration

3. **Mobile App**
   - PWA implementation
   - Offline reading support
   - Push notifications
   - Native app consideration

## 📝 Notes for Future Development

- Keep performance as top priority
- Maintain clean, semantic HTML
- Follow accessibility guidelines (WCAG 2.1)
- Document all major decisions
- Regular dependency updates
- Monitor Core Web Vitals

## 📧 Current Site Status (January 10, 2025)

**Live URL**: https://jw41784.github.io/meridian/

**Current Structure**:
- **Navigation**: Home, Articles, About (simplified from 5 to 3 pages)
- **Contact Method**: info@eudexio.com (consistent across site)
- **Content**: 4 articles with proper citations and explanations
  - "The Vanishing Point" (14 min) - Organizational memory and forgetting
  - "Everyone Wants a Transformational Leader" (16 min) - Leadership insights
  - "Why Every Company Now Looks the Same" (14 min) - Corporate convergence
  - "Welcome to Project Meridian" (1 min) - Introduction
- **Brand Voice**: Values-driven, challenging status quo, navigation-focused
- **Visual Identity**: Meridian compass logo integrated throughout (navigation, footer, favicon)

**Key Features**:
- **Dark Mode**: Full theme support with toggle, respects system preference
- **Reading Time**: Displayed on all article cards and pages with clock icon
- **Table of Contents**: Auto-generated for articles > 5 minutes with sticky sidebar
- **Responsive Design**: Optimized for mobile, tablet, and desktop
- **Search**: Client-side search across all articles
- **Modern UI**: Glassmorphism, gradients, animations, and hover effects
- **Code Quality**: Prettier formatting with consistent code style

**Brand Philosophy**:
- **Core Concept**: Meridian line as "true north" for authentic leadership
- **Value Proposition**: Move beyond "best practice" consensus toward context-specific solutions
- **Key Messaging**: "Question the Given," emphasize values-based navigation over convention

**Technical Notes**:
- BASE_URL properly handled with getUrl() helper in `/src/utils/links.ts`
- Favicon properly configured with cache-busting and correct GitHub Pages paths
- All links tested and working
- No broken references to hidden pages
- Clean, focused user experience with consistent visual identity

## 🔧 Technical Implementation Details

### Dark Mode Architecture
- **Theme Toggle**: React component with useEffect hooks for hydration safety
- **Storage**: localStorage with fallback to system preference
- **Styling**: Tailwind class-based dark mode with comprehensive prose-invert
- **Prevention of Flash**: Inline script in head executes before page render

### Reading Time Calculation
- **Algorithm**: 200 words per minute (industry standard for complex content)
- **Text Processing**: Strips MDX syntax, code blocks, links, and HTML
- **Integration**: Calculated at build time for all articles

### Table of Contents
- **Extraction**: Client-side extraction of H2-H4 headings
- **Visibility**: Conditional rendering based on reading time (> 5 minutes)
- **Responsiveness**: Grid layout (desktop) vs collapsible details (mobile)
- **Scrolling**: Independent scroll container with custom scrollbar styling
- **Active Tracking**: Intersection Observer API with 20% top margin

### Visual Design System
- **Colors**: Meridian brand colors with proper dark mode variants
- **Animations**: Float, fade-in, slide-up with staggered delays
- **Effects**: Glassmorphism, gradients, shadows, hover transforms
- **Components**: Modern card design, rounded corners, consistent spacing
- **Typography**: Improved hierarchy, better link styling, icon integration

### Code Quality Tools
- **Prettier**: Configured with Astro and Tailwind plugins
- **Formatting**: Consistent code style across all files
- **NPM Scripts**: format and format:check commands
- **Git Hooks**: Ready for pre-commit hooks integration

---

**Last Updated**: January 10, 2025
**Next Review**: February 1, 2025

**Development Techniques**
- Use puppeteer to check work
- NEVER assume libraries are available - always check package.json first
- Always use TodoWrite tool for planning multi-step tasks
- Test all features in both light and dark modes
- Ensure mobile responsiveness for all new components

Remember: The goal is to create a best-in-class blog platform that serves as both a content hub and a technical showcase for Project Meridian.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jw41784)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jw41784)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
