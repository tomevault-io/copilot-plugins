## dm2stay

> UI Component Standards

# UI Component Standards

Standards for UI components, icons, and styling in the project.

<rule>
name: ui_components
description: Standards for UI components, icons, and styling
filters:
  - type: file_extension
    pattern: "\\.(tsx|jsx)$"
  - type: content
    pattern: "import.*from ['\"]@/components.*['\"]|import.*from ['\"](mdc:lucide-react|tailwindcss).*['\"]"

actions:
  - type: suggest
    message: |
      When building UI components:

      1. Component Library:
         - Use shadcn/ui components for all common UI elements
         - Import from @/components directory
         - Customize components through the provided APIs rather than direct CSS modifications

      2. Icons:
         - Use Lucide React for all icons in the application
         - Import icons directly from 'lucide-react'
         Example:
         ```tsx
         import { Search, Menu, Settings } from 'lucide-react'
         ```

      3. Styling:
         - Use Tailwind v4 for all styling needs
         - Follow Tailwind's utility-first approach
         - Utilize Tailwind's built-in design system
         - Use theme colors for everything

examples:
  - input: |
      // Bad: Using non-standard components or icons
      import { FaSearch } from 'react-icons/fa'
      import { Button } from '@mui/material'

      // Good: Using specified libraries
      import { Search } from 'lucide-react'
      import { Button } from '@/components/ui/button'
    output: "Correctly using specified UI libraries"

metadata:
  priority: high
  version: 1.0
</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bjornpagen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
