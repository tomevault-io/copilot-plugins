## ai-powered-ats

> **Goal for AI:** Use Material-UI components for complex UI elements and Tailwind CSS for layout and custom styling.

# Material-UI with Tailwind CSS Styling Pattern

**Goal for AI:** Use Material-UI components for complex UI elements and Tailwind CSS for layout and custom styling.

**Constraint/Guideline:**
- Use Material-UI components for forms, buttons, cards, and complex interactions
- Use Tailwind CSS classes for layout, spacing, colors, and simple styling
- Import Material-UI components from their specific modules
- Use Material-UI's sx prop for component-specific styling when needed
- Maintain consistent spacing and color schemes using Tailwind classes

**Example (Good):**
```tsx
import React from 'react';
import { Card, CardContent, Typography, Button, CircularProgress } from '@mui/material';

function DashboardCard({ title, value, loading }: DashboardCardProps) {
  return (
    <Card className="bg-white p-4 rounded-lg shadow-md">
      <CardContent className="flex flex-col items-center justify-center">
        <Typography variant="h6" className="mb-2 text-gray-700">
          {title}
        </Typography>
        {loading ? (
          <CircularProgress size={24} />
        ) : (
          <Typography variant="h3" className="font-bold text-blue-600">
            {value}
          </Typography>
        )}
      </CardContent>
    </Card>
  );
}
```

**Example (Bad):**
```tsx
// Avoid mixing styling approaches inconsistently
<div style={{ padding: '16px', backgroundColor: 'white' }}>
  <h3 style={{ marginBottom: '8px' }}>{title}</h3>
</div>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EdgarJob)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EdgarJob)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
