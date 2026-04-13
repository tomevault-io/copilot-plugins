## hotelkrona

> > This file provides comprehensive guidelines for Cursor IDE to understand our project structure, coding standards, and development workflow. It helps Cursor provide better suggestions and maintain consistency.

# Hotel Corona - Cursor IDE Rules & Guidelines

> This file provides comprehensive guidelines for Cursor IDE to understand our project structure, coding standards, and development workflow. It helps Cursor provide better suggestions and maintain consistency.

## 📋 Project Context

### Project Type
**Type**: Next.js 14 Hotel Website  
**Purpose**: Luxury coastal hotel website with booking functionality  
**Status**: Phase 1 Complete - Static site with UI ready for backend integration  
**Design Theme**: Coastal Castle Elegance

### Tech Stack Overview
```
Frontend: Next.js 14 (App Router) + TypeScript + Tailwind CSS
Animation: Framer Motion
Deployment: Vercel
Current Phase: Static website (no backend yet)
Next Phase: CMS integration + Booking API + Email service
```

## 🎨 Design System

### Color Palette
Our entire design is based on this coastal castle color scheme:

```typescript
// Coastal Colors (Primary Palette)
{
  'coastal-cream': '#e9d9c3',      // Light backgrounds, hero sections
  'coastal-sand': '#d1c4a8',        // Alternating backgrounds
  'coastal-gold-light': '#b39a6b',  // Borders, subtle accents
  'coastal-gold': '#c4a65a',        // Primary accent, hover states
  'coastal-bronze': '#8a6e3d',      // Body text, dark elements
}

// Accent Colors (Secondary Palette)
{
  'accent-burgundy': '#7d2e2e',     // CTA buttons, urgent actions
  'accent-dark-brown': '#3d2817',   // Headers, strong text
}
```

**Usage Rules:**
- Use `coastal-cream` for primary backgrounds
- Use `coastal-sand` for alternating sections
- Use `accent-burgundy` for all CTA buttons
- Use `coastal-bronze` for body text
- Use `coastal-gold` for hover effects and highlights

### Typography System

```typescript
// Font Families
Headings: 'Playfair Display', Georgia, serif
Body: 'Montserrat', system-ui, sans-serif

// Size Scale
h1: 3rem (48px)    - Hero headings
h2: 2.25rem (36px) - Section headings
h3: 1.875rem (30px) - Subsection headings
h4: 1.5rem (24px)  - Card headings
body: 1rem (16px)  - Regular text
small: 0.875rem (14px) - Captions, labels
```

### Spacing Scale
Follow Tailwind's scale consistently:
```
xs: 4px   sm: 8px   md: 12px   lg: 16px
xl: 24px  2xl: 32px  3xl: 48px  4xl: 64px
```

## 📁 Project Structure & File Organization

```
hotel-corona/
├── src/
│   ├── app/                      # Next.js App Router pages
│   │   ├── page.tsx             # Homepage (/)
│   │   ├── layout.tsx           # Root layout (wraps all pages)
│   │   ├── globals.css          # Global styles
│   │   ├── rooms/               # Rooms pages
│   │   │   ├── page.tsx        # /rooms (listing)
│   │   │   └── [slug]/page.tsx # /rooms/[slug] (details)
│   │   ├── pricing/page.tsx     # /pricing
│   │   ├── amenities/page.tsx   # /amenities
│   │   ├── about/page.tsx       # /about
│   │   ├── contact/page.tsx     # /contact
│   │   ├── gallery/page.tsx     # /gallery
│   │   └── booking/page.tsx     # /booking
│   │
│   ├── components/
│   │   ├── layout/              # Global layout components
│   │   │   ├── Header.tsx       # Navigation header
│   │   │   ├── Footer.tsx       # Site footer
│   │   │   ├── BookingBar.tsx   # Sticky booking bar
│   │   │   └── LoadingOverlay.tsx # Initial loader
│   │   │
│   │   ├── sections/            # Page-specific sections
│   │   │   ├── HeroSection.tsx
│   │   │   ├── WelcomeSection.tsx
│   │   │   ├── FeaturedRooms.tsx
│   │   │   ├── AmenitiesOverview.tsx
│   │   │   ├── TestimonialsSection.tsx
│   │   │   ├── RestaurantsSection.tsx
│   │   │   └── CTASection.tsx
│   │   │
│   │   └── ui/                  # Reusable UI components
│   │       ├── Button.tsx
│   │       ├── Card.tsx
│   │       ├── Input.tsx
│   │       └── RoomCard.tsx
│   │
│   └── lib/                     # Utilities and constants
│       ├── constants.ts         # Static data (rooms, amenities, etc.)
│       └── utils.ts            # Helper functions
│
└── public/                      # Static assets
    └── images/                  # Images organized by section
```

### File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| **Components** | PascalCase | `HeroSection.tsx`, `BookingBar.tsx` |
| **Pages** | kebab-case | `[slug]/page.tsx`, `pricing/page.tsx` |
| **Utilities** | camelCase | `utils.ts`, `constants.ts` |
| **Constants** | UPPER_SNAKE_CASE | `MAX_GUESTS`, `ROOM_TYPES` |

## 💻 Code Standards & Patterns

### TypeScript Guidelines

#### 1. Always Define Types
```typescript
// ❌ Bad - No types
const fetchRooms = (id) => { ... };

// ✅ Good - Explicit types
interface FetchRoomsParams {
  id: string;
  includeAvailability?: boolean;
}

const fetchRooms = ({ id, includeAvailability = false }: FetchRoomsParams): Promise<Room[]> => {
  // ...
};
```

#### 2. Avoid `any` Type
```typescript
// ❌ Bad
const data: any = await fetch(...);

// ✅ Good
interface ApiResponse {
  rooms: Room[];
  total: number;
}
const data: ApiResponse = await fetch(...);

// ✅ Also acceptable when truly unknown
const data: unknown = await fetch(...);
if (isApiResponse(data)) {
  // Type guard narrows to ApiResponse
}
```

#### 3. Use Interfaces for Objects, Types for Unions
```typescript
// ✅ Interfaces for object shapes
interface Room {
  id: string;
  name: string;
  price: number;
}

// ✅ Types for unions/intersections
type RoomStatus = 'available' | 'booked' | 'maintenance';
type RoomWithStatus = Room & { status: RoomStatus };
```

### React Component Patterns

#### Standard Component Structure
```typescript
// 1. Imports (external libraries first, then internal)
import React from 'react';
import { motion } from 'framer-motion';
import { Button } from '@/components/ui/Button';
import { formatPrice } from '@/lib/utils';
import { ROOM_TYPES } from '@/lib/constants';

// 2. Types/Interfaces
interface ComponentProps {
  title: string;
  price: number;
  onBook?: () => void;
  children?: React.ReactNode;
}

// 3. Constants (component-level)
const ANIMATION_CONFIG = {
  duration: 0.3,
  ease: 'easeInOut',
};

// 4. Component Implementation
export const Component = ({ title, price, onBook, children }: ComponentProps) => {
  // a. Hooks (in order: state, context, refs, effects, custom hooks)
  const [isHovered, setIsHovered] = React.useState(false);
  
  // b. Computed values
  const formattedPrice = formatPrice(price);
  
  // c. Event handlers
  const handleClick = () => {
    if (onBook) {
      onBook();
    }
  };
  
  // d. Render
  return (
    <motion.div
      whileHover={{ scale: 1.02 }}
      className="p-6 bg-coastal-cream rounded-lg"
    >
      <h3 className="text-2xl font-serif text-coastal-bronze">{title}</h3>
      <p className="mt-2 text-coastal-gold font-semibold">{formattedPrice}</p>
      {children}
      <Button onClick={handleClick} variant="primary">
        Book Now
      </Button>
    </motion.div>
  );
};
```

#### Component Size Guidelines
- **Keep components under 300 lines**
- If larger, split into smaller components
- Extract complex logic into custom hooks
- Move repeated patterns to reusable utilities

### Styling with Tailwind CSS

#### Class Organization
Group Tailwind classes logically for readability:

```tsx
<div className="
  // Layout & Display
  flex flex-col items-center justify-between
  
  // Sizing
  w-full min-h-screen
  
  // Spacing
  px-6 py-4 gap-4
  
  // Colors & Backgrounds
  bg-coastal-cream text-coastal-bronze
  
  // Borders & Shadows
  rounded-lg border border-coastal-gold-light shadow-md
  
  // Typography
  font-sans text-base
  
  // Transitions & Animations
  transition-all duration-300
  
  // Hover, Focus, Active States
  hover:bg-coastal-sand hover:shadow-lg
  focus:outline-none focus:ring-2 focus:ring-coastal-gold
  
  // Responsive Breakpoints
  md:flex-row md:px-8
  lg:px-12
  xl:max-w-7xl
">
  Content
</div>
```

#### When to Use Custom CSS
Only create custom styles when:
1. Pattern repeated 5+ times across components
2. Complex animation not achievable with Tailwind
3. Global style needed site-wide

Add to `src/app/globals.css`:
```css
@layer components {
  .btn-primary {
    @apply bg-accent-burgundy text-coastal-gold px-6 py-3 rounded-lg;
    @apply transition-colors duration-300;
    @apply hover:bg-accent-dark-brown hover:scale-105;
    @apply focus:outline-none focus:ring-2 focus:ring-coastal-gold;
  }
}
```

### Animation Standards

#### Framer Motion Usage
```typescript
// Standard fade-in
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.5, ease: 'easeOut' }}
>
  Content
</motion.div>

// Scroll-triggered animation
import { useInView } from 'react-intersection-observer';

const [ref, inView] = useInView({
  triggerOnce: true,
  threshold: 0.1,
});

<motion.div
  ref={ref}
  initial={{ opacity: 0 }}
  animate={inView ? { opacity: 1, y: 0 } : { opacity: 0, y: 20 }}
  transition={{ duration: 0.6 }}
>
  Content
</motion.div>
```

#### Animation Timing Standards
- **Hover effects**: 300ms
- **Page transitions**: 500-600ms
- **Modal/drawer**: 400ms
- **Micro-interactions**: 200ms
- **Easing**: Use `ease-in-out` or custom bezier

### Form Handling Pattern

```typescript
'use client';

import { useState } from 'react';

export function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: '',
  });
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setStatus('loading');

    try {
      const response = await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      if (!response.ok) throw new Error('Failed to submit');
      
      setStatus('success');
      setFormData({ name: '', email: '', message: '' });
    } catch (error) {
      console.error('Form submission error:', error);
      setStatus('error');
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      {/* Form fields */}
      
      <button 
        type="submit" 
        disabled={status === 'loading'}
        className="btn-primary"
      >
        {status === 'loading' ? 'Sending...' : 'Send Message'}
      </button>

      {status === 'success' && (
        <p className="text-green-600">Message sent successfully!</p>
      )}
      {status === 'error' && (
        <p className="text-red-600">Failed to send. Please try again.</p>
      )}
    </form>
  );
}
```

## 🚫 Anti-Patterns (Don't Do This)

### Never Do These:

1. **❌ Don't use inline styles**
```tsx
// ❌ Bad
<div style={{ padding: '20px', backgroundColor: '#e9d9c3' }}>

// ✅ Good
<div className="p-5 bg-coastal-cream">
```

2. **❌ Don't use `any` type**
```typescript
// ❌ Bad
const data: any = fetchData();

// ✅ Good
interface Data { /* ... */ }
const data: Data = fetchData();
```

3. **❌ Don't hardcode content that should be in constants**
```typescript
// ❌ Bad
<p>From $299 per night</p>

// ✅ Good
import { STARTING_PRICE } from '@/lib/constants';
<p>From {formatPrice(STARTING_PRICE)} per night</p>
```

4. **❌ Don't skip alt text on images**
```tsx
// ❌ Bad
<Image src="/room.jpg" width={800} height={600} alt="" />

// ✅ Good
<Image 
  src="/room.jpg" 
  width={800} 
  height={600} 
  alt="Deluxe ocean-view room with king bed and balcony" 
/>
```

5. **❌ Don't create helper/example code**
```typescript
// ❌ Bad - Don't create example or placeholder implementations
// Example: ...

// ✅ Good - Implement actual, production-ready code
```

6. **❌ Don't run dev servers (assume they're already running)**
```bash
# ❌ Bad - Don't start servers
npm run dev

# ✅ Good - Assume server is running, just make changes
```

## ✅ Best Practices (Always Do This)

### 1. Use Semantic HTML
```tsx
// ✅ Good
<nav>
  <ul>
    <li><a href="/rooms">Rooms</a></li>
  </ul>
</nav>

<main>
  <article>
    <h1>Welcome to Hotel Corona</h1>
  </article>
</main>
```

### 2. Implement Accessibility
```tsx
// ✅ Good - All interactive elements have labels
<button aria-label="Close menu" onClick={closeMenu}>
  <CloseIcon />
</button>

// ✅ Good - Focus management
<input
  type="email"
  aria-required="true"
  aria-invalid={errors.email ? 'true' : 'false'}
  aria-describedby="email-error"
/>
{errors.email && (
  <span id="email-error" role="alert">{errors.email}</span>
)}
```

### 3. Optimize Images
```tsx
// ✅ Good
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hotel Corona exterior with ocean view"
  width={1920}
  height={1080}
  priority  // For above-the-fold images
  quality={85}
/>

<Image
  src="/amenity.jpg"
  alt="Luxury spa facilities"
  width={800}
  height={600}
  loading="lazy"  // For below-the-fold images
/>
```

### 4. Handle Loading and Error States
```typescript
// ✅ Good - Always handle all states
const [data, setData] = useState<Room[]>([]);
const [isLoading, setIsLoading] = useState(true);
const [error, setError] = useState<Error | null>(null);

if (isLoading) return <LoadingSpinner />;
if (error) return <ErrorMessage error={error} />;
return <RoomsList rooms={data} />;
```

## 🎯 Key Features Implementation Notes

### 1. Loading Overlay
- Display on initial page load only (not on navigation)
- Hotel logo centered on gradient background
- Minimum 1.5s display time
- Smooth fade transitions

### 2. Booking Bar
- Initially hidden
- Appears after scrolling 300px down
- Fixed to top with backdrop-blur glass effect
- Inline date pickers (not modal)
- Mobile: Collapsible with floating button

### 3. Responsive Breakpoints
```typescript
// Mobile: < 768px (no prefix)
// Tablet: 768px - 1023px (md:)
// Desktop: 1024px - 1279px (lg:)
// Large: 1280px+ (xl:, 2xl:)

<div className="
  px-4 text-sm           // Mobile
  md:px-6 md:text-base   // Tablet
  lg:px-8 lg:text-lg     // Desktop
  xl:px-12               // Large screens
">
```

## 📊 Performance Guidelines

### Image Optimization Checklist
- [ ] Use Next.js Image component
- [ ] Provide explicit width and height
- [ ] Use `priority` for above-the-fold images
- [ ] Use `loading="lazy"` for below-the-fold
- [ ] Optimize source images (< 500KB each)
- [ ] Provide descriptive alt text

### Code Splitting
- Use dynamic imports for heavy components
- Lazy load below-the-fold sections
- Keep initial bundle < 200KB

```typescript
// Dynamic import example
import dynamic from 'next/dynamic';

const HeavyMap = dynamic(() => import('@/components/Map'), {
  loading: () => <MapPlaceholder />,
  ssr: false,  // Disable SSR if component is client-only
});
```

## 📝 Git Commit Conventions

```bash
feat: Add new room comparison feature
fix: Correct date picker validation
style: Update button hover effects
refactor: Simplify booking form logic
docs: Add API integration guide
perf: Optimize image loading
test: Add room card tests
```

## 🔍 Common Tasks

### Adding a New Page
1. Create `src/app/[page-name]/page.tsx`
2. Add metadata for SEO
3. Implement page content
4. Add navigation link in Header
5. Update sitemap

### Creating a New Component
1. Determine location (layout/sections/ui)
2. Create component file with interface
3. Export from directory (optional)
4. Add to relevant page

### Adding to Constants
Edit `src/lib/constants.ts` and export new constant

## 📚 Documentation References

- **Architecture**: See `ARCHITECTURE.md`
- **Development Guide**: See `DEVELOPMENT.md`
- **Contributing**: See `CONTRIBUTING.md`
- **Deployment**: See `VERCEL_DEPLOYMENT_CHECKLIST.md`
- **Project Status**: See `PROJECT_STATUS.md`

## 🔄 Current Project Phase

**Phase**: 1.0 - Static Website Complete ✅

**Next Steps** (Phase 2):
- CMS integration
- Booking API connection
- Email service setup
- User authentication

**Important**: Don't implement backend features yet - focus on UI/UX improvements and optimization.

---

**Last Updated**: November 28, 2025  
**Cursor IDE Version**: For best experience, use latest Cursor IDE with TypeScript and Tailwind extensions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HeyitSaif) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
