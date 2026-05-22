## mosc-styling-standards

> MOSC Redesign Project - Comprehensive Styling Standards and Guidelines


# MOSC Redesign Project - Styling Standards

## **Design System Overview**
This project uses a **sacred, reverent design system** inspired by Orthodox Christian traditions with warm earth tones, elegant typography, and thoughtful spacing. All new pages and components must follow these established patterns.

## **Color Palette & CSS Variables**

### **Core System Colors**
- **Background**: `#F5F1E8` (soft cream) - `bg-background`
- **Foreground**: `#2D2A26` (near-black with warm undertones) - `text-foreground`
- **Border**: `rgba(139, 125, 107, 0.2)` (warm earth tone with transparency) - `border-border`
- **Input**: `#FFFFFF` (pure white) - `bg-input`
- **Ring**: `#8B7D6B` (warm earth tone) - `ring-ring`

### **Semantic Colors**
- **Primary**: `#8B7D6B` (warm earth tone) - `bg-primary text-primary-foreground`
- **Secondary**: `#A0926B` (lighter complement) - `bg-secondary text-secondary-foreground`
- **Accent**: `#6B4E3D` (rich brown) - `bg-accent text-accent-foreground`
- **Success**: `#4A6741` (muted sage green) - `bg-success text-success-foreground`
- **Warning**: `#A67C52` (warm amber) - `bg-warning text-warning-foreground`
- **Error/Destructive**: `#8B4A42` (subdued terracotta) - `bg-destructive text-destructive-foreground`
- **Muted**: `#EDE7D3` (lighter complement) - `bg-muted text-muted-foreground`

### **Card & Surface Colors**
- **Card**: `#FFFFFF` (pure white) - `bg-card text-card-foreground`
- **Popover**: `#FFFFFF` (pure white) - `bg-popover text-popover-foreground`

## **Typography System**

### **Font Families**
- **Headings**: `font-heading` (Crimson Text, serif) - For titles, headings, and important text
- **Body**: `font-body` (Source Sans Pro, sans-serif) - For paragraphs and general content
- **Caption**: `font-caption` (Lato, sans-serif) - For small text, labels, and captions
- **Monospace**: `font-mono` (JetBrains Mono, monospace) - For code and technical content

### **Font Usage Patterns**
```jsx
// ✅ DO: Use appropriate font families
<h1 className="font-heading font-semibold text-3xl text-foreground">Main Title</h1>
<p className="font-body text-lg text-muted-foreground">Body text content</p>
<small className="font-caption text-xs text-muted-foreground">Caption text</small>

// ❌ DON'T: Mix font families inappropriately
<h1 className="font-body font-semibold text-3xl">Main Title</h1> // Wrong font for heading
```

## **Spacing & Layout Standards**

### **Container Patterns**
- **Max Width**: `max-w-7xl mx-auto` for main content containers
- **Padding**: `px-4 sm:px-6 lg:px-8` for responsive horizontal padding
- **Sacred Spacing**: `space-sacred` (2rem) for consistent vertical spacing

### **Grid Systems**
- **Cards Grid**: `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8`
- **Feature Grid**: `grid grid-cols-1 md:grid-cols-3 gap-8 lg:gap-12`
- **Stats Grid**: `grid grid-cols-2 gap-4` for statistics

### **Section Spacing**
- **Large Sections**: `py-16` for major content sections
- **Medium Sections**: `py-12` for secondary sections
- **Small Sections**: `py-8` for compact sections

## **Component Styling Patterns**

### **Cards & Panels**
```jsx
// ✅ DO: Use consistent card styling
<div className="bg-card rounded-lg sacred-shadow p-6">
  <div className="flex items-center space-x-3 mb-6">
    <div className="w-8 h-8 bg-primary rounded-full flex items-center justify-center">
      <Icon name="IconName" size={16} color="white" />
    </div>
    <h3 className="text-lg font-heading font-semibold text-foreground">Card Title</h3>
  </div>
  {/* Card content */}
</div>
```

### **Buttons**
- Use the `Button` component with proper variants:
  - `default`, `destructive`, `outline`, `secondary`, `ghost`, `link`
  - `success`, `warning`, `danger` for semantic actions
- Sizes: `xs`, `sm`, `default`, `lg`, `xl`, `icon`

### **Form Elements**
- Use the `Input` component for all form fields
- Include proper labels, descriptions, and error states
- Use semantic colors for validation states

### **Icons**
- Always use the `AppIcon` component with consistent sizing
- Standard sizes: `16px` (small), `20px` (medium), `24px` (large), `32px` (extra large)
- Use semantic colors: `text-primary`, `text-muted-foreground`, etc.

## **Custom CSS Classes & Utilities**

### **Sacred Design Elements**
- **Shadows**: `sacred-shadow`, `sacred-shadow-sm`, `sacred-shadow-lg`
- **Gradients**: `sacred-gradient` (background gradient)
- **Border Radius**: `rounded-sacred` (8px)
- **Transitions**: `reverent-transition` (200ms ease-out)
- **Hover Effects**: `reverent-hover` (subtle scale transform)

### **Circular Elements**
- **Frames**: `circular-frame` for profile images and icons
- **Medallions**: Use `bg-primary rounded-full` with `sacred-shadow-lg`

## **Layout Patterns**

### **Hero Sections**
```jsx
<section className="relative bg-gradient-to-br from-background to-muted min-h-[600px] flex items-center">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 w-full">
    {/* Hero content */}
  </div>
</section>
```

### **Content Sections**
```jsx
<section className="py-16 bg-card">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div className="text-center mb-12">
      <h2 className="font-heading font-semibold text-3xl text-foreground mb-4">Section Title</h2>
      <p className="font-body text-lg text-muted-foreground max-w-3xl mx-auto">Description</p>
    </div>
    {/* Section content */}
  </div>
</section>
```

### **Feature Cards**
```jsx
<div className="text-center group">
  <div className="w-16 h-16 bg-primary/10 rounded-full flex items-center justify-center mx-auto mb-4 group-hover:bg-primary/20 reverent-transition">
    <Icon name="IconName" size={32} className="text-primary" />
  </div>
  <h3 className="font-heading font-medium text-xl text-foreground mb-3">Feature Title</h3>
  <p className="font-body text-muted-foreground leading-relaxed">Feature description</p>
</div>
```

## **Responsive Design Standards**

### **Breakpoint Usage**
- **Mobile First**: Design for mobile, then enhance for larger screens
- **Standard Breakpoints**: `sm:` (640px), `md:` (768px), `lg:` (1024px), `xl:` (1280px)
- **Grid Responsiveness**: Always provide mobile fallback with `grid-cols-1`

### **Typography Scaling**
- **Headings**: Use responsive text sizes (`text-2xl lg:text-3xl`)
- **Body Text**: Use `text-responsive-body` for fluid typography
- **Consistent Hierarchy**: Maintain clear visual hierarchy across all screen sizes

## **Accessibility Standards**

### **Color Contrast**
- All text must meet WCAG AA contrast requirements
- Use semantic colors for interactive elements
- Provide sufficient color contrast for all text combinations

### **Focus States**
- Use `focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2`
- Ensure all interactive elements have visible focus indicators
- Use proper ARIA labels and semantic HTML

## **Animation & Transitions**

### **Standard Transitions**
- **Reverent**: `reverent-transition` (200ms ease-out) for subtle interactions
- **Contemplative**: `transition-contemplative` (300ms ease-in-out) for slower animations
- **Hover Effects**: Use `reverent-hover` for subtle scale transforms

### **Loading States**
- Use consistent loading spinners for async operations
- Provide visual feedback for all user interactions
- Maintain smooth transitions between states

## **File Organization**

### **Component Structure**
- Place all UI components in `src/components/ui/`
- Use consistent naming: PascalCase for components
- Export components as default exports
- Include proper PropTypes or TypeScript types

### **Styling Files**
- Global styles in `src/styles/index.css` and `src/styles/tailwind.css`
- Component-specific styles using Tailwind classes
- Use the `cn()` utility for conditional class merging

## **Quality Checklist**

### **Before Submitting New Pages/Components**
- [ ] Uses correct color palette and semantic colors
- [ ] Follows typography hierarchy with appropriate font families
- [ ] Implements responsive design patterns
- [ ] Uses consistent spacing and layout patterns
- [ ] Includes proper accessibility features
- [ ] Follows component naming conventions
- [ ] Uses the established icon system
- [ ] Implements proper hover and focus states
- [ ] Maintains visual consistency with existing design
- [ ] Uses semantic HTML elements appropriately

## **Common Anti-Patterns to Avoid**

```jsx
// ❌ DON'T: Use hardcoded colors
<div className="bg-gray-100 text-black">Content</div>

// ❌ DON'T: Mix font families inappropriately
<h1 className="font-body">Main Heading</h1>

// ❌ DON'T: Use inconsistent spacing
<div className="p-4 m-8">Content</div> // Inconsistent with sacred spacing

// ❌ DON'T: Skip responsive design
<div className="grid grid-cols-4 gap-4">Content</div> // No mobile fallback

// ❌ DON'T: Use generic shadows
<div className="shadow-lg">Content</div> // Use sacred-shadow instead
```

## **References**
- See `code_clone_ref/mosc_redesign_template/src/styles/tailwind.css` for complete color system
- See `code_clone_ref/mosc_redesign_template/src/components/ui/Button.jsx` for component patterns
- See `code_clone_ref/mosc_redesign_template/src/pages/homepage/components/` for layout examples
- See `code_clone_ref/mosc_redesign_template/tailwind.config.js` for complete theme configuration

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
