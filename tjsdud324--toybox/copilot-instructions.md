## toybox

> This file provides guidance to Claude Code (claude.ai/code) when working with artifacts in this TOYBOX repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with artifacts in this TOYBOX repository.

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run linter
npm run lint

# Preview production build
npm run preview

# Deploy to GitHub Pages
npm run deploy
```

## Artifact Development Workflow

This is a React-based portfolio for showcasing Claude-generated artifacts. All artifacts share common patterns while maintaining creative freedom.

### Required Patterns

All artifacts must use:
- **React** with TypeScript
- **Tailwind CSS** for styling
- **shadcn/ui** component patterns with **Radix UI primitives**
- **ArtifactMetadata** type for metadata export

### Creating New Artifacts

Place artifacts in `src/artifacts/` as either:
- Direct files: `src/artifacts/my-component.tsx`
- Subdirectories: `src/artifacts/my-component/index.tsx` (for complex artifacts with assets)

**Required Structure:**

```typescript
const ComponentName: React.FC = () => {
  return (
    <div className="p-6">
      {/* Your component implementation */}
      {/* Note: Avoid max-width constraints if your artifact needs fullscreen capability */}
    </div>
  );
};

export default ComponentName;
```

### Metadata System

Metadata can be provided in three ways (in priority order):

1. **External TypeScript file** (recommended for complex artifacts):
   - Direct: `src/artifacts/my-artifact.metadata.ts`
   - Subdirectory: `src/artifacts/my-artifact/metadata.ts`

2. **External JSON file**:
   - Direct: `src/artifacts/my-artifact.metadata.json`
   - Subdirectory: `src/artifacts/my-artifact/metadata.json`

3. **Component export** (simplest approach):
   ```typescript
   export const metadata: ArtifactMetadata = { ... };
   ```

**Metadata Interface:**

```typescript
interface ArtifactMetadata {
  title: string;
  description?: string;
  type: 'react' | 'svg' | 'mermaid';
  tags: string[];
  folder?: string;           // Logical grouping without changing file structure
  createdAt: string;         // ISO date string (use `date -u +"%Y-%m-%dT%H:%M:%SZ"`)
  updatedAt: string;         // ISO date string
  hidden?: boolean;          // Hide from gallery (still accessible via direct URL)
  fullscreen?: boolean;      // Auto-fullscreen in standalone mode
  underMaintenance?: boolean; // Show maintenance banner
}
```

**Example with component export:**

```typescript
import { ArtifactMetadata } from '@/lib/artifactLoader';

export const metadata: ArtifactMetadata = {
  title: 'Component Name',
  description: 'Brief description of what this artifact does',
  type: 'react',
  tags: ['interactive', 'demo'],
  folder: 'Category Name',
  createdAt: '2024-01-15T10:00:00Z',
  updatedAt: '2024-01-15T10:00:00Z',
};

const ComponentName: React.FC = () => {
  return <div className="p-6">{/* Your implementation */}</div>;
};

export default ComponentName;
```

**Example with external TypeScript metadata** (`my-artifact.metadata.ts`):

```typescript
import { ArtifactMetadata } from '@/lib/artifactLoader';

export const metadata: ArtifactMetadata = {
  title: 'My Artifact',
  description: 'A description of my artifact',
  type: 'react',
  tags: ['demo', 'example'],
  createdAt: '2024-01-01',
  updatedAt: '2024-01-15',
};
```

**Example with external JSON metadata** (`my-artifact.metadata.json`):

```json
{
  "title": "My Artifact",
  "description": "A description of my artifact",
  "type": "react",
  "tags": ["demo", "example"],
  "createdAt": "2024-01-01",
  "updatedAt": "2024-01-15"
}
```

### Viewing Modes

- **Gallery view** (`/a/:artifactName`): Shows artifact with metadata, tags, and navigation
- **Standalone view** (`/standalone/:artifactName`): Clean presentation with optional fullscreen toggle

### Gallery Features

- **Type filtering**: Filter by React, SVG, or Mermaid
- **Tag filtering**: Filter by any tag present in artifacts
- **Text search**: Search across title, description, and tags
- **Sorting**: By updated date, created date, or alphabetical
- **Per-card error boundaries**: Individual card errors don't crash the gallery

### Styling Guidelines

- Use **Tailwind CSS** classes for all styling
- Follow **shadcn/ui** component patterns for UI elements
- Leverage **Radix UI primitives** for complex interactions
- Import shadcn/ui components from `@/components/ui/`
- Use responsive design patterns (`sm:`, `md:`, `lg:` prefixes)

**Example with shadcn/ui:**

```typescript
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';

const MyArtifact: React.FC = () => {
  return (
    <div className="p-6 space-y-6">
      {/* Example layout - adjust max-width/centering based on your artifact's needs */}
      <div className="max-w-4xl mx-auto">
        <Card>
          <CardHeader>
            <CardTitle>My Interactive Component</CardTitle>
          </CardHeader>
          <CardContent className="space-y-4">
            <Button variant="default" size="lg">
              Click me
            </Button>
            <Badge variant="secondary">Demo</Badge>
          </CardContent>
        </Card>
      </div>
    </div>
  );
};
```

### State Management

- Use **React hooks** (useState, useEffect) for local component state
- For complex state, consider **useReducer** or context patterns
- Avoid external state management libraries unless absolutely necessary
- State should be self-contained within each artifact

### Development Best Practices

1. **Type Safety**: Use TypeScript strict mode - properly type all props and state
2. **Accessibility**: Follow Radix UI accessibility patterns
3. **Responsive Design**: Test on mobile and desktop viewport sizes
4. **Performance**: Avoid heavy computations in render loops
5. **Error Boundaries**: Handle errors gracefully within components

### Testing and Debugging

**Development Testing:**
```bash
# Start dev server and test in browser
npm run dev
# Navigate to http://localhost:5173/a/your-artifact-name
```

**Production Testing:**
```bash
# Test with production base path
npm run test-production
# Navigate to http://localhost:4173/a/your-artifact-name
```

**Browser Automation Testing:**
This template includes Playwright MCP server integration for automated browser testing. Use Playwright MCP server tools to:
- Take screenshots and verify visual behavior
- Test user interactions (clicks, form submissions, navigation)
- Verify responsive design across viewport sizes
- Automate testing workflows for artifact debugging

**Debugging Philosophy:**
- **NEVER assume optimistically** that a fix will work
- **ALWAYS verify** with actual browser testing using Playwright MCP server
- Take screenshots to document behavior before and after changes
- Test across different viewport sizes using browser automation
- Verify routing and navigation work correctly with automated testing

### Common Artifact Patterns

**Interactive Demo:**
```typescript
const [count, setCount] = useState(0);

return (
  <Card className="w-full max-w-md mx-auto">
    <CardContent className="pt-6">
      <div className="text-center space-y-4">
        <div className="text-2xl font-bold">{count}</div>
        <Button onClick={() => setCount(c => c + 1)}>
          Increment
        </Button>
      </div>
    </CardContent>
  </Card>
);
```

**Data Visualization:**
```typescript
const data = [/* your data */];

return (
  <div className="p-6 space-y-6">
    {/* Full-width layout for data visualizations */}
    <h2 className="text-2xl font-bold">Data Visualization</h2>
    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
      {data.map((item, index) => (
        <Card key={index}>
          <CardContent className="pt-6">
            {/* visualization content */}
          </CardContent>
        </Card>
      ))}
    </div>
  </div>
);
```

**Form Example:**
```typescript
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';

const [formData, setFormData] = useState({ name: '', email: '' });

return (
  <Card className="w-full max-w-lg mx-auto">
    <CardHeader>
      <CardTitle>Contact Form</CardTitle>
    </CardHeader>
    <CardContent className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="name">Name</Label>
        <Input
          id="name"
          value={formData.name}
          onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
        />
      </div>
      <Button className="w-full">Submit</Button>
    </CardContent>
  </Card>
);
```

### Path Resolution

- Use `@/` alias for imports from `src/`
- Co-locate assets with complex artifacts in subdirectories
- All artifacts automatically get routes at `/a/artifact-name`

### Artifact Discovery

Artifacts are automatically discovered by the build system. Simply create a new `.tsx` file with proper metadata export and it will appear in the gallery.

---
> Source: [tjsdud324/toybox](https://github.com/tjsdud324/toybox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
