## gittok-dev

> Project: Svelte5 SvelteKit

# .cursorrules
Project: Svelte5 SvelteKit

1. Documentation & Comments
   - Every page, component, or function must begin with a comment describing its purpose and the context in which it was created.
     Example:
         // Purpose: [What it does]
         // Context: [Why it was created]
   - make sure to use the correct syntax for the language you are using. ie .svelte files use html comments and .js files use js comments.
   - Use the Svelte5 style guide for coding standards.

2. Styling Guidelines
   - Use Tailwind CSS for styling with a focus on TikTok-like UI patterns:
     - Full-screen vertical scroll with snap points
     - Dark mode first design with gradient backgrounds
     - Floating action buttons and stats on content edges
     - Glassmorphic UI elements with backdrop blur
     - Minimalist animations and transitions

   - Color Palette:
     - Primary background: Dark gradients (from-gray-900 to-black)
     - UI elements: Semi-transparent whites and grays (gray-800/50 with backdrop blur)
     - Accent colors:
       - Yellow-400 for stars/important actions
       - Blue-400 for interactive elements
       - Gray-400 for secondary elements

   - Typography:
     - Font Hierarchy:
       - Monospace for metrics, stats, and technical data
       - Serif for main content and descriptions
       - System fonts for UI elements
     - Text sizes should be responsive (sm for mobile, base/lg for desktop)

   - Layout:
     - Snap-mandatory vertical scrolling
     - Content cards should take full viewport height
     - Three-section layout: Header, Main Content, Footer
     - Right-aligned floating action buttons
     - Bottom-aligned user information
     - Consistent padding (p-6) for content containers

3. Code Quality & Structure
   - Follow SvelteKit and Svelte5 best practices for component design and file organization.
   - Adhere to DRY (Don't Repeat Yourself) and modular design principles.
   - Use clear, consistent naming conventions and structured code organization.

4. Version Control
   - Write clear, concise commit messages.
   - Use feature branches with descriptive names and maintain code review practices.

5. Performance & Accessibility
   - Implement lazy loading for images and content
   - Use IntersectionObserver for viewport-based loading
   - Ensure smooth scrolling and transitions
   - Maintain accessibility with proper ARIA labels
   - Optimize images and media content
   - Use semantic HTML elements

6. Security
   - Validate and sanitize all external inputs.
   - Follow established web security best practices.
   - Sanitize markdown and HTML content before rendering

7. Testing & Maintenance
   - Write unit and integration tests for critical functionality.
   - Test scroll behavior and intersection observers
   - Verify responsive design across devices
   - Maintain an up-to-date test suite and documentation for future maintenance.

---
> Source: [BlackShoreTech/gittok.dev](https://github.com/BlackShoreTech/gittok.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
