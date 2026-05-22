## conditional-layout-separation

> Pattern for creating separate layouts for different sections/routes in Next.js App Router


# Conditional Layout Separation Pattern

## **Overview**
This pattern enables creating completely separate layouts for different sections of a Next.js application while maintaining a single root layout. Useful for creating public sections (like marketing sites, documentation, or church websites) that need different styling and navigation from the main authenticated application.

## **Problem Solved**
- **Layout Duplication**: Prevents main app header/footer from showing alongside section-specific headers/footers
- **Authentication Separation**: Allows public sections to work without authentication requirements
- **Styling Isolation**: Enables completely different design systems for different sections
- **Hydration Issues**: Avoids Next.js App Router hydration errors from nested HTML tags

## **Implementation Pattern**

### **1. Create ConditionalLayout Component**

```typescript
// src/components/ConditionalLayout.tsx
'use client';

import React from 'react';
import { usePathname } from 'next/navigation';

interface ConditionalLayoutProps {
  children: React.ReactNode;
  header: React.ReactNode;
  footer: React.ReactNode;
}

export default function ConditionalLayout({ children, header, footer }: ConditionalLayoutProps) {
  const pathname = usePathname();

  // Define which routes should use separate layout
  const isSeparateLayoutRoute = pathname?.startsWith("/section-prefix") ?? false;

  // For separate layout routes, just render children (section handles its own header/footer)
  if (isSeparateLayoutRoute) {
    return <>{children}</>;
  }

  // For main app routes, render the full layout with header and footer
  return (
    <>
      {header}
      <div className="flex-1 flex flex-col">
        {children}
      </div>
      {footer}
    </>
  );
}
```

### **2. Update Root Layout**

```typescript
// src/app/layout.tsx
import ConditionalLayout from "../components/ConditionalLayout";
import Header from "../components/Header";
import Footer from "../components/Footer";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <ClerkProvider>
          <TrpcProvider>
            <ConditionalLayout
              header={<Header hideMenuItems={false} />}
              footer={<Footer />}
            >
              {children}
            </ConditionalLayout>
          </TrpcProvider>
        </ClerkProvider>
      </body>
    </html>
  );
}
```

### **3. Create Section-Specific Layout**

```typescript
// src/app/section-prefix/layout.tsx
import React from 'react';
import { Metadata } from 'next';
import SectionHeader from './components/SectionHeader';
import SectionFooter from './components/SectionFooter';
import './section-globals.css'; // Section-specific styles

export const metadata: Metadata = {
  title: {
    template: '%s | Section Name',
    default: 'Section Name',
  },
  description: 'Section-specific description',
};

interface SectionLayoutProps {
  children: React.ReactNode;
}

export default function SectionLayout({ children }: SectionLayoutProps) {
  return (
    <div className="section-layout min-h-screen bg-section-background flex flex-col">
      <SectionHeader />

      <main className="section-main flex-1">
        {children}
      </main>

      <SectionFooter />
    </div>
  );
}
```

### **4. Create Section-Specific Styling**

```css
/* src/app/section-prefix/section-globals.css */
@import '../globals.css';

/* Override main app styles for section */
body {
  background-color: var(--section-background) !important;
  color: var(--section-foreground) !important;
  font-family: 'SectionFont', sans-serif !important;
}

/* Section-specific CSS variables */
:root {
  --section-background: #F5F1E8;
  --section-foreground: #2D2A26;
  --section-primary: #8B7D6B;
  /* ... other section-specific variables */
}

/* Section layout isolation */
.section-layout {
  background-color: var(--section-background) !important;
  color: var(--section-foreground) !important;
}

/* Override any conflicting main app styles */
.section-layout * {
  box-sizing: border-box;
}
```

### **5. Update Middleware for Public Routes**

```typescript
// src/middleware.ts
export default authMiddleware({
  publicRoutes: [
    "/",
    "/section-prefix",
    "/section-prefix/(.*)",
    // ... other public routes
  ],
  // ... rest of middleware config
});
```

## **Key Implementation Details**

### **Route Detection Logic**
```typescript
// Single section
const isSeparateLayoutRoute = pathname?.startsWith("/mosc") ?? false;

// Multiple sections
const separateLayoutSections = ["/mosc", "/docs", "/marketing"];
const isSeparateLayoutRoute = separateLayoutSections.some(section =>
  pathname?.startsWith(section)
) ?? false;

// Regex pattern matching
const isSeparateLayoutRoute = pathname?.match(/^\/(mosc|docs|marketing)/) !== null;
```

### **Null Safety**
```typescript
// Always handle potential null pathname
const pathname = usePathname();
const isSeparateLayoutRoute = pathname?.startsWith("/section") ?? false;
```

### **Conditional Provider Wrapping**
```typescript
// For sections that don't need authentication
export default function ConditionalLayout({ children, header, footer }: ConditionalLayoutProps) {
  const pathname = usePathname();
  const isPublicSection = pathname?.startsWith("/public-section") ?? false;

  if (isPublicSection) {
    // Render without authentication providers
    return <>{children}</>;
  }

  // Render with full app providers
  return (
    <AuthProvider>
      {header}
      <div className="flex-1 flex flex-col">
        {children}
      </div>
      {footer}
    </AuthProvider>
  );
}
```

## **Benefits**

- **✅ Complete Separation**: Sections can have entirely different designs and functionality
- **✅ No Duplication**: Eliminates header/footer duplication issues
- **✅ Authentication Control**: Public sections work without auth requirements
- **✅ Performance**: No unnecessary components loaded for public sections
- **✅ Maintainability**: Clear separation of concerns between app sections
- **✅ SEO**: Each section can have its own metadata and styling

## **Use Cases**

- **Public Marketing Sites**: `/marketing` with different branding
- **Documentation Sections**: `/docs` with documentation-specific navigation
- **Church/Organization Sites**: `/mosc` with religious/org-specific styling
- **Landing Pages**: `/landing` with conversion-focused design
- **Public APIs**: `/api-docs` with API documentation layout

## **Best Practices**

### **Naming Conventions**
- Use descriptive section prefixes: `/mosc`, `/docs`, `/marketing`
- Keep section layouts in their own directories: `src/app/section-name/`
- Use consistent component naming: `SectionHeader`, `SectionFooter`

### **Styling Isolation**
- Always use `!important` for section-specific overrides
- Import base styles but override as needed
- Use CSS custom properties for section-specific theming
- Prefix section classes to avoid conflicts: `.section-layout`, `.section-header`

### **Route Organization**
- Group related routes under section prefixes
- Use middleware to handle authentication for different sections
- Keep section-specific components in section directories

### **Error Handling**
- Always handle null pathname in route detection
- Provide fallback rendering for edge cases
- Test navigation between sections thoroughly

## **Troubleshooting**

### **Common Issues**
- **Hydration Errors**: Never use `<html>` or `<body>` tags in nested layouts
- **Style Conflicts**: Use `!important` and specific selectors for overrides
- **Route Detection**: Always handle null pathname with optional chaining
- **Authentication Issues**: Ensure public routes are properly configured in middleware

### **Debugging**
```typescript
// Add logging to debug route detection
console.log('Current pathname:', pathname);
console.log('Is separate layout route:', isSeparateLayoutRoute);
```

## **Example: Complete MOSC Implementation**

```typescript
// src/components/ConditionalLayout.tsx - MOSC Example
'use client';

import React from 'react';
import { usePathname } from 'next/navigation';

interface ConditionalLayoutProps {
  children: React.ReactNode;
  header: React.ReactNode;
  footer: React.ReactNode;
}

export default function ConditionalLayout({ children, header, footer }: ConditionalLayoutProps) {
  const pathname = usePathname();

  // MOSC routes use their own layout (no auth, different styling)
  const isMOSCRoute = pathname?.startsWith("/mosc") ?? false;

  if (isMOSCRoute) {
    return <>{children}</>;
  }

  return (
    <>
      {header}
      <div className="flex-1 flex flex-col">
        {children}
      </div>
      {footer}
    </>
  );
}
```

This pattern provides a robust, scalable solution for creating separate layouts while maintaining Next.js App Router best practices.

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
