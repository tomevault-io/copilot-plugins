## my-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Install dependencies (uses pnpm)
pnpm install

# Run development server with Turbopack
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Run linting
pnpm lint

# Format code with Prettier
pnpm format
```

## Architecture Overview

This is a Next.js 15.3.2 application for Studio Garcia Beauty Academy - a comprehensive beauty education center and salon complex in Itaquaquecetuba. The website uses the App Router pattern with TypeScript and Tailwind CSS v4.

### Core Technologies

#### Framework & Build

- **Next.js 15.3.2** with App Router (`/app` directory)
- **React 19** with extensive use of client components
- **TypeScript** in strict mode with path alias `@/*`

#### Styling & UI

- **Tailwind CSS v4** with PostCSS configuration
- **shadcn/ui** components (New York style, with CVA for variants)
- **tw-animate-css** for additional animation utilities
- **Custom theme tokens** mapped to CSS variables in `globals.css`
- **cn() utility** from `lib/utils.ts` for conditional class merging

#### Animation & Graphics

- **GSAP** with React integration for complex animations
  - Menu animations with timeline control
  - Text animations with character splitting
  - Proper cleanup in useEffect hooks
- **Split-type** for text splitting and animation
- **OGL** for WebGL shader-based backgrounds
  - Custom vertex and fragment shaders
  - Mouse interaction support
  - Hex to vec4 color conversion

### Component Architecture

#### Key Patterns

- **Client-First Components**: Heavy use of 'use client' directive for interactivity
- **Composition Pattern**: Components use composition (e.g., Button with asChild prop)
- **Animation Integration**: Most components include GSAP or OGL animations
- **Responsive Design**: Custom breakpoint handling, especially in Menu component
- **Type Safety**: Full TypeScript with proper interface definitions

#### Component Structure

- `/components/ui/` - shadcn/ui components with CVA variants
- `/components/menu/` - Navigation with GSAP animations and responsive behavior
- `/components/` - Feature components (AnimatedText, DynamicBackground, balatro)

### Animation Implementation Details

#### GSAP Usage

- Timeline-based animations with proper pause/play control
- Cleanup patterns: Always kill timelines on unmount
- Mouse interaction handling with normalized coordinates
- Stagger effects for sequential animations

#### OGL/WebGL Implementation

- Renderer setup with proper resize handling
- Uniform-based shader parameters for real-time control
- Performance optimizations with pixelFilter parameter
- WebGL context cleanup on component unmount

### Styling Conventions

- Utility-first approach with Tailwind classes
- Responsive utilities throughout (`lg:` breakpoint = 1024px)
- Dark mode support via `.dark` class variants
- oklch color space for modern color management
- Consistent spacing and typography scales

### Important Configuration

- **components.json** - shadcn/ui configuration with gray base color
- **ESLint** with Next.js Core Web Vitals and Prettier integration
- **Prettier** with Tailwind CSS plugin for class sorting
- Images from `via.placeholder.com` allowed in Next.js config

### Project Context - Studio Garcia Beauty Academy

#### Business Model Evolution

Studio Garcia is transforming from a general commercial complex into a specialized Beauty Academy and beauty center, focusing on:

- Professional beauty education and certification programs
- High-end beauty services and treatments
- Strategic positioning for the 2025 beauty market in Itaquaquecetuba
- Integration of education and practical services in one location

#### Website Goals

- Establish Studio Garcia Beauty Academy as the premier beauty education institution
- Generate qualified leads for beauty courses and professional training
- Showcase the complete educational ecosystem and career opportunities
- Build trust through success stories and professional certifications
- Facilitate enrollment and consultation bookings

#### Key Sections Implementation (Legacy)

_Note: These sections reflect the previous commercial complex model and should be updated based on the new Beauty Academy requirements outlined above._

1. **Hero Section**: Split-screen design with Balatro animation and main value proposition
2. **About Section**: Infrastructure benefits with video showcase
3. **Current Professionals**: Card grid showcasing existing tenants
4. **Available Spaces**: Photo gallery with room details and CTA
5. **CTA Section**: Strong call-to-action with subtle Balatro background
6. **Footer**: Complete contact information and quick links

#### Design Principles

- Professional yet aspirational beauty industry aesthetic
- Elegant animations using GSAP ScrollTrigger for premium feel
- Sophisticated color palette with rose gold and purple tones
- Mobile-first responsive design (60%+ mobile traffic expected)
- Multiple conversion points with clear, action-oriented CTAs
- Visual storytelling through student success transformations
- Balance between luxury appeal and accessibility

## Project Requirements - Beauty Academy Landing Page

### 📋 Requirements Documentation

Based on project documentation in `/docs/`:

- **Guia Estratégico para Centro de Beleza em Itaquaquecetuba - Oportunidades e Tendências 2025**
- **Studio Garcia Beauty Academy - Conteúdo Web Completo**

> **Note**: The content below is a comprehensive structure based on the PDF file names and the project's evolution to a Beauty Academy. The specific details from the PDFs should be incorporated when available in text format.

### 🎯 Primary Objectives

1. **Position Studio Garcia as the premier beauty education center** in Itaquaquecetuba
2. **Generate qualified leads** for beauty courses and professional services
3. **Build trust and credibility** through professional design and content
4. **Showcase the complete ecosystem** of education + practice + business opportunities

### 👥 Target Audience & Personas

#### Primary Persona: Aspiring Beauty Professionals

- Age: 18-35 years
- Gender: Predominantly female (80%)
- Income: Lower to middle class seeking career advancement
- Goals: Professional certification, career change, entrepreneurship
- Pain Points: Lack of quality education options, need for practical experience

#### Secondary Persona: Beauty Service Consumers

- Age: 25-55 years
- Income: Middle to upper-middle class
- Goals: High-quality beauty treatments, trusted professionals
- Values: Expertise, hygiene, modern techniques

### 🏗️ Landing Page Structure

#### 1. Hero Section

- **Headline**: Compelling value proposition focusing on transformation and success
- **Subheadline**: Emphasize professional certification and market opportunities
- **CTA**: "Comece Sua Jornada" / "Agende Uma Visita"
- **Visual**: Professional beauty environment with students/professionals
- **Trust Indicators**: Certifications, partnerships, success numbers

#### 2. About Section - "Por Que Studio Garcia?"

- **Unique Differentiators**:
  - Complete ecosystem (learn + practice + earn)
  - Industry-certified instructors
  - State-of-the-art facilities
  - Job placement assistance
  - Business mentorship program

#### 3. Courses & Programs Section

- **Course Categories**:
  - Hair Styling & Coloring
  - Makeup & Aesthetics
  - Nail Art & Design
  - Skincare & Treatments
  - Business Management for Beauty Professionals
- **Format**: Card-based design with duration, price, and certification info
- **CTA**: Individual course inquiry forms

#### 4. Success Stories & Testimonials

- **Student Transformations**: Before/after career stories
- **Graduate Achievements**: Salon openings, employment success
- **Video Testimonials**: Authentic student experiences
- **Metrics**: Graduation rate, employment rate, average income increase

#### 5. Facilities Showcase

- **Virtual Tour**: Interactive gallery or video walkthrough
- **Equipment Highlights**: Professional-grade tools and products
- **Learning Environments**: Classrooms, practice salons, theory rooms
- **Safety & Hygiene**: COVID protocols, sterilization practices

#### 6. Services Section

- **Student Salon**: Discounted services by supervised students
- **Professional Services**: Full-price services by certified professionals
- **Special Packages**: Combined education + service offerings

#### 7. Career Support Section

- **Job Board**: Current opportunities for graduates
- **Business Incubator**: Support for opening own salons
- **Partnerships**: Salons and spas hiring graduates
- **Continuing Education**: Advanced courses and workshops

#### 8. Enrollment Process

- **Step-by-Step Guide**: Clear enrollment journey
- **Requirements**: Documentation, prerequisites
- **Financial Options**: Payment plans, scholarships, financing
- **Calendar**: Course start dates and schedules

#### 9. FAQ Section

- **Course-related**: Duration, certification, requirements
- **Financial**: Costs, payment options, ROI
- **Career**: Job prospects, earning potential
- **Practical**: Schedule, location, materials

#### 10. Contact & Location

- **Multiple Contact Methods**: WhatsApp, phone, email, form
- **Interactive Map**: Google Maps integration
- **Business Hours**: Academy and salon schedules
- **Parking & Transportation**: Accessibility information

### 💬 Content Strategy & Tone of Voice

#### Brand Voice Characteristics

- **Professional yet Approachable**: Expert knowledge delivered warmly
- **Inspirational**: Focus on transformation and possibilities
- **Inclusive**: Welcome all backgrounds and experience levels
- **Action-Oriented**: Clear next steps and calls-to-action

#### Key Messaging Pillars

1. **Education Excellence**: "Learn from the best to become the best"
2. **Practical Experience**: "Practice in real salon environment"
3. **Career Success**: "Your path to financial independence"
4. **Community**: "Join the Studio Garcia family"

### 🔧 Functional Requirements

#### Core Features

1. **Responsive Design**: Mobile-first approach (60%+ mobile traffic expected)
2. **Fast Loading**: Optimized images, lazy loading, CDN usage
3. **SEO Optimized**: Local SEO for "curso de beleza Itaquaquecetuba"
4. **Analytics Integration**: GA4, Facebook Pixel, conversion tracking
5. **Lead Capture Forms**: Multiple touchpoints with CRM integration
6. **WhatsApp Integration**: Direct chat button, click-to-WhatsApp CTAs
7. **Social Proof**: Instagram feed integration, Google Reviews widget

#### Advanced Features

1. **Course Catalog Filter**: By category, duration, price, schedule
2. **Online Pre-Registration**: Form with document upload capability
3. **Virtual Consultation Booking**: Calendar integration for tours
4. **Payment Calculator**: Estimate monthly payments for courses
5. **Blog/Resources Section**: SEO content for beauty tips and trends

### 🎨 Visual Design Direction

#### Color Palette

- **Primary**: Sophisticated rose gold (#B76E79)
- **Secondary**: Deep purple (#6B46C1)
- **Accent**: Soft pink (#FFC0CB)
- **Neutral**: Cream (#FAF0E6) and charcoal (#333333)

#### Typography

- **Headlines**: Modern serif for elegance (Playfair Display or similar)
- **Body**: Clean sans-serif for readability (Inter or Montserrat)
- **CTAs**: Bold sans-serif for impact

#### Imagery Style

- **Photography**: Professional, diverse models, bright and aspirational
- **Icons**: Elegant line icons for services and features
- **Patterns**: Subtle beauty-themed patterns (waves, florals)
- **Videos**: High-quality showcasing techniques and testimonials

### 📊 Performance Metrics & KPIs

#### Primary KPIs

1. **Lead Generation**: Form submissions, WhatsApp clicks
2. **Engagement**: Time on site, pages per session
3. **Conversion Rate**: Visitors to leads, leads to enrollments
4. **SEO Performance**: Local search rankings, organic traffic

#### Secondary KPIs

1. **Social Shares**: Content virality
2. **Return Visitors**: Brand loyalty indicator
3. **Mobile Performance**: Core Web Vitals scores
4. **Cost per Lead**: Marketing efficiency

### 🚀 Launch Strategy

#### Pre-Launch

1. **Content Creation**: All copy, images, and videos ready
2. **Technical SEO**: Schema markup, sitemaps, robots.txt
3. **Testing**: Cross-browser, mobile, form functionality
4. **Legal Compliance**: Privacy policy, terms, cookie consent

#### Launch Phase

1. **Soft Launch**: Limited audience for feedback
2. **Influencer Partnerships**: Local beauty influencers
3. **Social Media Campaign**: Instagram, Facebook, TikTok
4. **Google Ads**: Local search campaigns

#### Post-Launch

1. **A/B Testing**: Headlines, CTAs, form fields
2. **Content Updates**: Blog posts, success stories
3. **Retargeting Campaigns**: Facebook and Google
4. **Email Nurture Sequences**: Lead warming automation

### 📝 Content Sections Detail

#### Homepage Copy Structure

1. **Hero**: 15-20 words headline + 30-40 words subheadline
2. **About**: 150-200 words establishing credibility
3. **Courses**: 50-75 words per course description
4. **Testimonials**: 3-5 testimonials, 50-100 words each
5. **CTA Sections**: 20-30 words per CTA block

#### SEO Content Requirements

- **Target Keywords**:
  - "curso de beleza Itaquaquecetuba"
  - "escola de estética Itaquaquecetuba"
  - "curso de maquiagem profissional"
  - "curso de cabeleireiro com certificado"
- **Meta Descriptions**: Unique for each page, 150-160 characters
- **Alt Text**: Descriptive for all images
- **Header Structure**: Proper H1-H6 hierarchy

### 🔐 Technical Specifications

#### Performance Requirements

- **Page Load Time**: < 3 seconds on 3G
- **Time to Interactive**: < 5 seconds
- **Lighthouse Score**: > 90 for all categories
- **Mobile Usability**: 100% Google Mobile-Friendly

#### Security & Compliance

- **SSL Certificate**: Required for trust and SEO
- **LGPD Compliance**: Cookie consent, privacy policy
- **Form Security**: CAPTCHA, validation, sanitization
- **Backup Strategy**: Daily automated backups

### 📱 Mobile-First Considerations

#### Mobile UX Priorities

1. **Thumb-Friendly Navigation**: Bottom nav or hamburger menu
2. **Click-to-Call CTAs**: Prominent phone number buttons
3. **Simplified Forms**: Multi-step for better completion
4. **Optimized Images**: WebP format, responsive sizes
5. **Touch Targets**: Minimum 44x44px for all interactive elements

### 🎯 Conversion Optimization

#### CTA Strategy

- **Primary CTA**: "Agende Sua Visita" (Schedule Your Visit)
- **Secondary CTAs**:
  - "Baixe o Catálogo de Cursos" (Download Course Catalog)
  - "Fale com um Consultor" (Speak with a Consultant)
  - "Conheça Nossas Instalações" (Tour Our Facilities)

#### Trust Signals

- **Certifications**: MEC recognition, industry partnerships
- **Numbers**: Years in business, graduates, success rate
- **Guarantees**: Job placement assistance, satisfaction guarantee
- **Social Proof**: Reviews, ratings, testimonial videos

### 📈 Growth Features

#### Scalability Considerations

1. **Multi-language Support**: Structure for future Portuguese variants
2. **Course Management System**: Easy addition of new programs
3. **Location Expansion**: Template for multiple campus pages
4. **API Integration**: Ready for future CRM/LMS connections

This comprehensive update to the landing page requirements ensures that Studio Garcia Beauty Academy is positioned as the leading beauty education institution in Itaquaquecetuba, with a clear focus on student success and career transformation.

## 📚 Project Documentation

### Available Documents

The `/docs` folder contains essential project documentation:

1. **Guia Estratégico para Centro de Beleza em Itaquaquecetuba - Oportunidades e Tendências 2025.pdf**

   - Strategic market analysis for beauty centers in Itaquaquecetuba
   - 2025 trends and opportunities in the beauty industry
   - Competitive landscape and positioning strategies
   - Target audience insights and market gaps

2. **Studio Garcia Beauty Academy - Conteúdo Web Completo.pdf**
   - Complete website content strategy
   - Detailed copy for all sections
   - SEO-optimized content structure
   - Brand messaging and value propositions

### How to Use These Documents

To fully implement the requirements:

1. Convert PDFs to text format using external tools
2. Extract specific content for each landing page section
3. Update this CLAUDE.md with exact copy and specifications
4. Ensure all strategic insights are reflected in the implementation

### Key Implementation Notes

- The project represents a strategic pivot from general commercial complex to specialized beauty academy
- Focus on local market positioning in Itaquaquecetuba
- Emphasis on professional education and career transformation
- Integration of education, practice, and entrepreneurship support

## Important Development Rules

### Component Fix Policy

**NEVER create separate "fix" components or example components to demonstrate fixes.** When encountering issues in existing components:

1. **Always fix issues directly** in the problematic component file
2. **Do not create** separate example files, demo components, or implementation guides
3. **Update the existing component** with the necessary fixes inline
4. **Preserve the original component structure** while implementing fixes

This ensures cleaner code organization and prevents unnecessary file proliferation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eieraso)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/eieraso)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
