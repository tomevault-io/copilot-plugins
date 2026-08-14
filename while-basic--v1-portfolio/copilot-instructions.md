## v1-portfolio

> Say Yo Chris, before anything.

Say Yo Chris, before anything.

# Next.js Project Structure Rules
[next]
# Metadata must be in server components (layout.ts/page.ts) without 'use client'
metadata_location = ["layout.ts", "layout.tsx"]
avoid_metadata_in_client = true

# Client Components
client_components = ["page.tsx", "components/*.tsx"]
client_directive_required = true

# Server Components
server_components = ["layout.tsx", "api/*.ts"]
avoid_client_directive = true

# Component Organization
[components]
# Keep shared components in components directory
shared_location = "src/components"
# Page-specific components should be co-located with their pages
page_specific_location = "src/app/**/components"

# Styling
[styling]
# Use Tailwind classes with proper responsive prefixes
responsive_prefixes = ["sm:", "md:", "lg:", "xl:", "2xl:"]
mobile_first = true
# Common breakpoints
breakpoints = {
  "sm": "640px",
  "md": "768px",
  "lg": "1024px",
  "xl": "1280px",
  "2xl": "1536px"
}
# Glass morphism effects
glass_morphism = {
  "base": "backdrop-blur-md bg-white/10",
  "border": "border border-white/20",
  "hover": "hover:bg-white/20"
}

# TypeScript
[typescript]
strict = true
# Use 'as const' for static arrays/objects to ensure type safety
const_assertions = true
# Define prop interfaces/types in the same file as component
props_location = "with_component"
# Use type unions for filter categories
filter_types = "union_types"

# State Management
[state]
# Use React hooks for local state
local_state = "useState"
# Keep state close to where it's used
state_location = "component_level"
# Filter state management
filter_state = {
  "location": "page_level",
  "type": "useState",
  "default": "all"
}

# Component Patterns
[patterns]
# Card components
card = {
  "image_ratio": "aspect-[4/3]",
  "hover_scale": "hover:scale-105",
  "transition": "transition-all duration-300"
}
# Filter components
filter = {
  "mobile_scroll": "overflow-x-auto scrollbar-hide",
  "button_spacing": "space-x-2 sm:space-x-4",
  "active_state": "bg-white text-black"
}

# Grid Layouts
[grid]
# Responsive grid columns
columns = {
  "mobile": "grid-cols-1",
  "tablet": "sm:grid-cols-2",
  "desktop": "lg:grid-cols-3"
}
# Grid gaps
gaps = {
  "mobile": "gap-4",
  "desktop": "sm:gap-6"
}

# Content Display
[content]
# Text truncation
truncate = {
  "description": "line-clamp-2",
  "tags": "max-display: 3"
}
# Image optimization
images = {
  "format": ["webp", "avif"],
  "loading": "lazy",
  "sizes": "responsive"
}

# Project-Specific
[project_portfolio]
# Category organization
categories = {
  "all": "Show all projects",
  "ai-ml": "AI & Machine Learning",
  "web": "Web Development",
  "game": "Game Development",
  "3d": "3D & Animation",
  "healthcare": "Healthcare"
}
# Project card features
project_card = {
  "required_fields": ["title", "description", "technologies", "category"],
  "optional_fields": ["imageUrl", "liveUrl", "featured"],
  "max_technologies": 3
}

[Error Handling and Validation]
# Project validation
project_validation = {
  "required_fields": true,
  "valid_categories": true,
  "image_exists": true
}
# Filter validation
filter_validation = {
  "valid_category": true,
  "fallback": "all"
}

# Performance
[performance]
# Use proper image optimization
next_image = true
# Implement proper loading states
loading_states = true
# Use proper suspense boundaries
suspense = true

# SEO
[seo]
# Keep metadata in layout files
metadata_location = "layout.tsx"
# Define proper titles and descriptions
required_meta = ["title", "description"]

# Routing
[routing]
# Use Next.js App Router conventions
app_router = true
# Keep route handlers in api directory
api_routes = "src/app/api"

# Error Handling
[error_handling]
# Implement error boundaries
error_boundaries = true
# Use proper error pages
error_pages = ["error.tsx", "not-found.tsx"]

# Code Style
[code_style]
# Use consistent naming conventions
naming = {
  "components": "PascalCase",
  "functions": "camelCase",
  "constants": "UPPER_CASE",
  "files": "kebab-case"
}
# Maximum line length
max_line_length = 80

# Mobile Responsiveness
[mobile]
# Always design mobile-first
mobile_first = true
# Use proper touch targets
min_touch_target = "44px"
# Ensure proper viewport settings
viewport_meta = true

# Accessibility
[accessibility]
# Use proper ARIA labels
aria_labels = true
# Ensure proper color contrast
color_contrast = true
# Implement keyboard navigation
keyboard_nav = true

# Testing
[testing]
# Keep tests close to components
test_location = "__tests__"
# Use proper testing libraries
testing_library = "jest"

# Documentation
[documentation]
# Keep documentation up to date
readme_required = true
# Document complex components
component_docs = true

# Version Control
[version_control]
# Ignore proper files
ignore_patterns = [
  "node_modules",
  ".next",
  "build",
  "dist",
  ".env*"
]

Code Style and Structure:
- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns; avoid classes
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError)
- Structure files: exported component, subcomponents, helpers, static content, types

Naming Conventions:
- Use lowercase with dashes for directories (e.g., components/auth-wizard)
- Favor named exports for components

TypeScript Usage:
- Use TypeScript for all code; prefer interfaces over types
- Avoid enums; use maps instead
- Use functional components with TypeScript interfaces

Syntax and Formatting:
- Use the "function" keyword for pure functions
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements
- Use declarative JSX

Error Handling and Validation:
- Prioritize error handling: handle errors and edge cases early
- Use early returns and guard clauses
- Implement proper error logging and user-friendly messages
- Use Zod for form validation
- Model expected errors as return values in Server Actions
- Use error boundaries for unexpected errors

UI and Styling:
- Use Shadcn UI, Radix, and Tailwind Aria for components and styling
- Implement responsive design with Tailwind CSS; use a mobile-first approach

Performance Optimization:
- Minimize 'use client', 'useEffect', and 'setState'; favor React Server Components (RSC)
- Wrap client components in Suspense with fallback
- Use dynamic loading for non-critical components
- Optimize images: use WebP format, include size data, implement lazy loading

Key Conventions:
- Use 'nuqs' for URL search parameter state management
- Optimize Web Vitals (LCP, CLS, FID)
- Limit 'use client':
  - Favor server components and Next.js SSR
  - Use only for Web API access in small components
  - Avoid for data fetching or state management

Follow Next.js docs for Data Fetching, Rendering, and Routing

---
> Source: [while-basic/v1-portfolio](https://github.com/while-basic/v1-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
