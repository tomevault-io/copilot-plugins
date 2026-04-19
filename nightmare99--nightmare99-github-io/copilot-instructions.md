## nightmare99-github-io

> This is a modern, animated portfolio website built with React, TypeScript, Vite, and TailwindCSS. The site features a gradient color scheme (blue → purple → pink) with smooth animations and a responsive design.

# Development Guidelines - Vishal Kumar Portfolio

## Project Overview
This is a modern, animated portfolio website built with React, TypeScript, Vite, and TailwindCSS. The site features a gradient color scheme (blue → purple → pink) with smooth animations and a responsive design.

## Technology Stack
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite
- **Styling**: TailwindCSS
- **Animations**: Framer Motion
- **Icons**: Lucide React
- **UI Components**: Custom components with shadcn/ui patterns

## Project Structure

```
vishalkumar.com/
├── src/
│   ├── components/
│   │   ├── sections/          # Main page sections
│   │   │   ├── Hero.tsx       # Landing section with wavy background
│   │   │   ├── Experience.tsx # Work experience cards
│   │   │   ├── Projects.tsx   # Project showcase grid
│   │   │   ├── Skills.tsx     # Skills categorized by domain
│   │   │   ├── Education.tsx  # Educational background
│   │   │   ├── Achievements.tsx # Awards and recognitions
│   │   │   └── Contact.tsx    # Contact information
│   │   ├── ui/                # Reusable UI components
│   │   │   ├── card.tsx       # Card component variations
│   │   │   ├── badge.tsx      # Badge component
│   │   │   ├── button.tsx     # Button component
│   │   │   ├── wavy-background.tsx # Animated SVG wave background
│   │   │   └── wave.tsx       # Wave SVG separator components
│   │   └── Navbar.tsx         # Navigation bar
│   ├── lib/
│   │   └── utils.ts           # Utility functions (cn helper)
│   ├── App.tsx                # Main app component
│   ├── main.tsx               # Entry point
│   └── index.css              # Global styles and custom animations
├── public/                    # Static assets
├── package.json               # Dependencies and scripts
├── tailwind.config.js         # TailwindCSS configuration
├── tsconfig.json              # TypeScript configuration
└── vite.config.ts             # Vite build configuration
```

## Component Hierarchy

### Main App Flow
```
App.tsx
├── Navbar (sticky navigation)
└── Main Content
    ├── Hero (full-screen landing)
    ├── Experience (work history)
    ├── Projects (portfolio items)
    ├── Skills (technical skills)
    ├── Education (academic background)
    ├── Achievements (awards)
    └── Contact (contact details)
```

### Section Components Pattern
All section components follow this structure:
1. **Container**: `<section>` with unique `id` for navigation
2. **Wave Separator**: Optional `Wave` component for visual separation
3. **Motion Wrapper**: Framer Motion animations for scroll-triggered effects
4. **Title & Underline**: Consistent heading with gradient underline
5. **Content Grid/List**: Cards or items with data mapping

## Data Locations

### Inline Data (within components)
All data is currently **hardcoded within the section components** as constant arrays:

- **Experience Data**: `src/components/sections/Experience.tsx` (lines 6-52)
  - Array: `experiences[]`
  - Fields: `title`, `company`, `period`, `location`, `achievements[]`

- **Projects Data**: `src/components/sections/Projects.tsx` (lines 15-60)
  - Array: `projects[]`
  - Fields: `title`, `description`, `icon`, `tech[]`, `highlights[]`

- **Skills Data**: `src/components/sections/Skills.tsx` (lines 13-75)
  - Array: `skillCategories[]`
  - Fields: `title`, `icon`, `skills[]`

- **Education Data**: `src/components/sections/Education.tsx` (lines 6-21)
  - Array: `education[]`
  - Fields: `degree`, `field`, `institution`, `period`, `cgpa`

- **Achievements Data**: `src/components/sections/Achievements.tsx` (lines 5-38)
  - Array: `achievements[]`
  - Fields: `title`, `description`, `organization`, `date`, `icon`, `color`

- **Contact Info**: `src/components/sections/Contact.tsx` (lines 7-29)
  - Array: `contactInfo[]`
  - Fields: `icon`, `label`, `value`, `href`

### Configuration Files

- **Tailwind Config**: `tailwind.config.js`
  - Custom colors
  - Font family configurations (Playfair Display, signature fonts)
  - Extended theme settings

- **TypeScript Config**: `tsconfig.json`
  - Path aliases (`@/` points to `./src`)
  - Compiler options

- **Vite Config**: `vite.config.ts`
  - Path resolution
  - Build settings

## Design System

### Color Scheme
**Primary Gradient**: Blue → Purple → Pink
- Blue: `#2563eb` (blue-600)
- Purple: `#9333ea` (purple-600)
- Pink: `#db2777` (pink-600)

**Background Colors**:
- Base: `#000000` (black)
- Cards: `#18181b` (zinc-950), `#27272a` (zinc-900)
- Borders: `#3f3f46` (zinc-800)

**Text Colors**:
- Primary: `#ffffff` (white)
- Secondary: `#d4d4d8` (gray-300)
- Muted: `#a1a1aa` (gray-400)

### Typography
- **Headings**: Playfair Display (serif, elegant)
- **Body**: Default system font stack
- **Signature**: "Agbalumo" (handwritten style for name)

### Icon Styling Pattern
**Standard approach for all icons**:
```tsx
<div className="w-[size] h-[size] bg-gradient-to-br from-blue-600 via-purple-600 to-pink-600 rounded-lg flex items-center justify-center">
  <Icon className="h-[icon-size] w-[icon-size] text-white" />
</div>
```
- Container sizes: 8x8, 10x10, 12x12, 16x16 (in 0.25rem units)
- Icon colors: Always white (`text-white`) inside gradient backgrounds
- **Never use** `bg-clip-text` with `text-transparent` on SVG icons (causes invisibility)

### Animation Guidelines

#### Framer Motion Patterns
**Container/Item Animation** (for lists):
```tsx
const container = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: { staggerChildren: 0.1 }
  }
}

const item = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 }
}
```

**Scroll-Triggered Animations**:
```tsx
<motion.div
  initial={{ opacity: 0, y: 20 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true }}
  transition={{ duration: 0.8 }}
>
```

#### Custom CSS Animations
Located in `src/index.css`:
- `@keyframes gradient-flow`: Animated gradient text
- `@keyframes float`: Floating icon effect
- `@keyframes blob`: Blob/bubble morphing
- `@keyframes wave-breathe`: Wave background breathing effect

### Component Styling Patterns

#### Card Components
```tsx
<Card className="bg-gradient-to-br from-zinc-950 to-zinc-900 border-zinc-800 hover:border-pink-600 transition-all duration-300 hover:shadow-xl hover:shadow-pink-600/20">
```

#### Section Titles
```tsx
<h2 className="text-4xl md:text-5xl font-bold mb-4 text-white">
  Section Title
</h2>
<div className="w-24 h-1 bg-gradient-to-r from-blue-600 via-purple-600 to-pink-600 mx-auto rounded-full" />
```

#### Gradient Text
```tsx
<span className="bg-gradient-to-r from-blue-600 via-purple-600 to-pink-600 bg-clip-text text-transparent">
  Gradient Text
</span>
```

## Special Components

### WavyBackground Component
**Location**: `src/components/ui/wavy-background.tsx`

**Purpose**: Animated SVG wave background for Hero section

**Key Properties**:
- Three layered SVG paths with independent animations
- Gradients: `gradient1`, `gradient2`, `gradient3`
- Animation durations: 12s, 10s, 16s
- Blur effect: `blur-2xl`
- Overlay gradient: Fades from transparent to black at bottom

**Customization Points**:
- Wave positions (Y-coordinates in path `d` attribute)
- Color opacity (rgba values in gradients)
- Animation speed (`dur` attribute in `<animate>`)
- Blur intensity (className)

### Wave Separator Components
**Location**: `src/components/ui/wave.tsx`

**Available Waves**:
- `Wave1`: Standard wave separator
- `Wave2`: Alternative wave pattern
- `Wave3`: Third wave variation
- `Wave4`: Fourth wave pattern

**Props**:
- `fill`: SVG fill color
- `className`: Additional Tailwind classes
- `flip`: Boolean to flip wave vertically

**Usage**:
```tsx
<Wave3 
  fill="rgb(0, 0, 0)" 
  className="absolute -top-1 left-0 w-full z-20" 
  flip 
/>
```

## Navigation System

### Navbar Component
**Location**: `src/components/Navbar.tsx`

**Features**:
- Sticky positioning with backdrop blur
- Smooth scroll to sections
- Active section highlighting
- Responsive mobile menu

**Navigation Links** (order matters):
1. Home → `#home`
2. Experience → `#experience`
3. Projects → `#projects`
4. Skills → `#skills`
5. Education → `#education`
6. Achievements → `#achievements`
7. Contact → `#contact`

## Responsive Design

### Breakpoints (TailwindCSS defaults)
- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

### Common Responsive Patterns
```tsx
// Grid layouts
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3"

// Text sizes
className="text-4xl md:text-5xl lg:text-6xl"

// Spacing
className="px-4 md:px-6 lg:px-8"

// Flex direction
className="flex flex-col md:flex-row"
```

## Best Practices

### 1. Data Management
- Keep data arrays at the top of component files
- Use TypeScript interfaces for type safety
- Consider moving to separate data files (`src/data/`) if data grows

### 2. Component Structure
- One section per file
- Keep components focused and single-purpose
- Use composition over complexity

### 3. Styling
- Use Tailwind utility classes
- Maintain gradient consistency (blue→purple→pink)
- Follow existing animation patterns
- Never apply gradient text to SVG icons

### 4. Animation
- Use `viewport={{ once: true }}` for scroll animations
- Stagger children for list animations
- Keep duration reasonable (0.3s - 0.8s)

### 5. Icons
- Always wrap icons in gradient containers
- Use white text color for icons inside gradients
- Maintain consistent sizing (parent/icon ratio ~2:1)

### 6. Performance
- Use `React.lazy()` for code splitting if needed
- Optimize images in `public/` folder
- Keep bundle size minimal

## Common Tasks

### Adding a New Section
1. Create component in `src/components/sections/NewSection.tsx`
2. Define data array within component
3. Import Framer Motion animations
4. Follow existing section structure pattern
5. Add to `App.tsx` in appropriate order
6. Add navigation link to `Navbar.tsx`
7. Ensure section has unique `id` attribute

### Updating Content
1. Locate data array in respective section component
2. Modify object properties
3. Ensure all required fields are present
4. Icons must be from `lucide-react` package

### Changing Colors
1. Update color values in component classNames
2. Maintain gradient pattern: `from-[color] via-[color] to-[color]`
3. Update Wave background gradients in `wavy-background.tsx` if needed

### Adding New Animations
1. Define keyframes in `src/index.css`
2. Apply animation class to element
3. Consider using Framer Motion for scroll-triggered effects

## Environment & Build

### Development
```bash
npm run dev
```
Starts dev server at `http://localhost:5173`

### Build
```bash
npm run build
```
Outputs to `dist/` directory

### Preview Production Build
```bash
npm run preview
```

## Dependencies

### Core
- `react` & `react-dom`: UI framework
- `typescript`: Type safety
- `vite`: Build tool

### Styling & UI
- `tailwindcss`: Utility-first CSS
- `framer-motion`: Animation library
- `lucide-react`: Icon library
- `class-variance-authority`: Component variants
- `clsx` & `tailwind-merge`: Classname utilities

### Type Definitions
- `@types/react`
- `@types/react-dom`
- `@types/node`

## Git Workflow
This project is configured for deployment to GitHub Pages at `Nightmare99.github.io`.

## Notes for AI Agents

1. **Consistency is Key**: Maintain the established gradient color scheme and animation patterns
2. **Data-First**: Always verify data structure before modifying components
3. **Test Responsiveness**: Check mobile, tablet, and desktop views
4. **Icon Visibility**: Never use gradient text clipping on SVG icons
5. **Animation Performance**: Keep animations smooth; avoid excessive motion
6. **Type Safety**: Use TypeScript types for all data structures
7. **Accessibility**: Ensure proper heading hierarchy and ARIA labels where needed

## Future Enhancements to Consider

1. **Data Separation**: Move hardcoded data to `src/data/` directory
2. **CMS Integration**: Connect to headless CMS for dynamic content
3. **Blog Section**: Add blog/articles section with MDX support
4. **Dark/Light Mode**: Currently dark-only; could add theme toggle
5. **Internationalization**: Add i18n support for multiple languages
6. **Performance Metrics**: Add analytics and performance monitoring
7. **SEO Optimization**: Add meta tags, sitemap, robots.txt
8. **Testing**: Add unit tests (Vitest) and E2E tests (Playwright)

## Contact & Maintenance
- GitHub: [nightmare99](https://github.com/nightmare99)
- Email: vishal.s.kumar99@gmail.com
- LinkedIn: [mnq-](https://www.linkedin.com/in/mnq-/)

---
**Last Updated**: October 2025  
**Version**: 1.0.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nightmare99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
