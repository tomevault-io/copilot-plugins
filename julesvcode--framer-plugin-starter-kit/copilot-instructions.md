## framer-plugin-starter-kit

> This is a Framer plugin starter kit using React, TypeScript, Vite, and Tailwind CSS.

# Framer Plugin Development Rules

## Project Overview
This is a Framer plugin starter kit using React, TypeScript, Vite, and Tailwind CSS.

## Tech Stack
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite 6
- **Styling**: Tailwind CSS v3
- **Plugin SDK**: framer-plugin v3
- **Dev Server**: HTTPS via vite-plugin-mkcert (required for Framer)

## Important Configuration Notes

### PostCSS & Tailwind Setup
- Use `postcss.config.js` and `tailwind.config.js` with ES module syntax (`export default`)
- These work with `"type": "module"` in package.json
- Always import CSS at the **main entry point** (`src/main.tsx`), not in components
- Port 5173 must be available - kill any conflicting processes before starting dev server

### Development Server
- Run: `npm run dev`
- Server MUST run on port 5173 (default Vite port) with HTTPS
- If port is in use, kill the process: `lsof -ti:5173 | xargs kill -9`
- Clear Vite cache if issues: `rm -rf node_modules/.vite`

## Framer Plugin API

### Core API (`framer` object)

#### UI Configuration
```typescript
import { framer } from "framer-plugin"

// Show plugin UI (call this at the top level, not in a component)
framer.showUI({
    position: "top right" | "top left" | "bottom right" | "bottom left" | "center",
    width: number,
    height: number,
    resizable?: boolean,
    minWidth?: number,
    minHeight?: number,
    maxWidth?: number,
    maxHeight?: number,
})

// Hide the plugin UI
framer.hideUI()
```

#### Selection Management
```typescript
import { CanvasNode } from "framer-plugin"

// Subscribe to selection changes (returns cleanup function)
const unsubscribe = framer.subscribeToSelection((selection: CanvasNode[]) => {
    console.log("Selection changed:", selection)
})

// Get current selection once
const selection = await framer.getSelection()
```

#### Canvas Manipulation
```typescript
// Add SVG to canvas
await framer.addSVG({
    svg: string,           // SVG markup
    name?: string,         // Layer name in Framer
    x?: number,            // Position
    y?: number,
    width?: number,        // Dimensions
    height?: number,
    fill?: string,         // Override fill color
})

// Add image to canvas
await framer.addImage({
    image: string | Uint8Array,  // Data URL or binary data
    name?: string,
    x?: number,
    y?: number,
    width?: number,
    height?: number,
})

// Add component instance
await framer.addComponentInstance({
    url: string,           // Component URL
    name?: string,
    attributes?: Record<string, any>,
    x?: number,
    y?: number,
})

// Clone nodes
await framer.cloneNode(node: CanvasNode)
```

#### Node Operations
```typescript
// Get node by ID
const node = await framer.getNode(id: string)

// Update node properties
await framer.setAttributes(node: CanvasNode, attributes: {
    name?: string,
    x?: number,
    y?: number,
    width?: number,
    height?: number,
    rotation?: number,
    opacity?: number,
    visible?: boolean,
    locked?: boolean,
    // ... many more properties
})

// Delete nodes
await framer.removeNode(node: CanvasNode)
```

#### Project & User Info
```typescript
// Get project information
const project = await framer.getProject()

// Get current user
const user = await framer.getUser()
```

### CanvasNode Type
```typescript
interface CanvasNode {
    id: string
    name: string
    type: "Frame" | "Text" | "SVG" | "Image" | "Component" | "ComponentInstance" | ...
    x: number
    y: number
    width: number
    height: number
    rotation: number
    opacity: number
    visible: boolean
    locked: boolean
    parent?: CanvasNode
    children?: CanvasNode[]
    // ... many more properties
}
```

## Common Patterns

### React Hook for Selection
```typescript
function useSelection() {
    const [selection, setSelection] = useState<CanvasNode[]>([])
    
    useEffect(() => {
        return framer.subscribeToSelection(setSelection)
    }, [])
    
    return selection
}
```

### Async Operations with Error Handling
```typescript
const handleAction = async () => {
    try {
        await framer.addSVG({ svg: "...", name: "My SVG" })
        framer.notify("Success!", { variant: "success" })
    } catch (error) {
        console.error(error)
        framer.notify("Error occurred", { variant: "error" })
    }
}
```

### Plugin Modes
In `framer.json`:
- `"canvas"` - Plugin appears in canvas mode
- `"preview"` - Plugin appears in preview mode
- Both can be specified as an array

## Styling Best Practices

### Tailwind Usage
- Use Tailwind utility classes for styling
- Framer provides base button styles: `framer-button-primary`, `framer-button-secondary`
- Keep plugin UI compact and focused

### Framer CSS Variables (Dark/Light Mode Support)
Framer provides CSS custom properties that automatically adapt to dark and light modes:

```css
/* Color Variables */
--framer-color-tint              /* Primary brand color */
--framer-color-tint-dimmed       /* Dimmed brand color */
--framer-color-tint-dark         /* Dark brand color */

/* Background Colors */
--framer-color-bg                /* Primary background */
--framer-color-bg-secondary      /* Secondary background */
--framer-color-bg-tertiary       /* Tertiary background */

/* Dividers & Borders */
--framer-color-divider           /* Divider/border color */

/* Text Colors */
--framer-color-text              /* Primary text */
--framer-color-text-reversed     /* Reversed text (light on dark) */
--framer-color-text-secondary    /* Secondary text */
--framer-color-text-tertiary     /* Tertiary/muted text */
```

**Usage Examples:**
```css
/* In CSS files */
.my-element {
    background-color: var(--framer-color-bg);
    color: var(--framer-color-text);
    border: 1px solid var(--framer-color-divider);
}

.my-button {
    background-color: var(--framer-color-tint);
    color: var(--framer-color-text-reversed);
}
```

```typescript
// In React inline styles
<div style={{ 
    backgroundColor: 'var(--framer-color-bg-secondary)',
    color: 'var(--framer-color-text-secondary)'
}}>
    Content
</div>
```

**Best Practices:**
- Always use Framer's CSS variables instead of hard-coded colors for theme support
- Use `--framer-color-text-secondary` and `--framer-color-text-tertiary` for less prominent text
- Use `--framer-color-tint` for interactive elements and primary actions
- Use `--framer-color-divider` for borders and separators

### Common Utility Classes
```typescript
<div className="flex flex-col gap-4 p-4">          // Column layout with spacing
<button className="framer-button-primary">         // Primary Framer button
<p className="text-sm text-gray-600">              // Small muted text
```

## Development Workflow

1. **Start Dev Server**: `npm run dev` (must be on port 5173)
2. **Open in Framer**: Plugin auto-reloads with changes
3. **Build for Production**: `npm run build`
4. **Package Plugin**: `npm run pack` (creates .framer-plugin file)

## Common Issues & Solutions

### Tailwind Not Working
- Ensure CSS is imported in `src/main.tsx`, not in components
- Check that PostCSS config is present and correct
- Restart dev server completely
- Clear Vite cache: `rm -rf node_modules/.vite`

### Plugin Won't Open in Framer
- Verify dev server is running on port 5173 (HTTPS)
- Check for port conflicts: `lsof -ti:5173`
- Ensure `vite-plugin-framer` is in plugins array
- mkcert must be configured for HTTPS

### Type Errors
- Import types from "framer-plugin": `CanvasNode`, `Project`, `User`, etc.
- Use `framer-plugin` v3 for latest API

## File Structure
```
src/
  ├── main.tsx          # Entry point, imports CSS
  ├── App.tsx           # Main plugin component
  ├── App.css           # Tailwind imports + custom styles
  └── vite-env.d.ts     # Vite type definitions
framer.json             # Plugin metadata
vite.config.ts          # Vite configuration
postcss.config.js       # PostCSS with Tailwind
tailwind.config.js      # Tailwind configuration
```

## Documentation Links
- Official Docs: https://www.framer.com/developers/plugins/introduction
- API Reference: https://www.framer.com/developers/plugins/api
- Examples: https://github.com/framer/plugins

## Monetization & Paywalls

If you want to add premium features or a paywall to your Framer plugin, you can integrate payment providers. Here are the recommended options:

### Payment Provider APIs

#### Lemon Squeezy
- **Overview**: Merchant of record platform, handles taxes and compliance automatically
- **Website**: https://www.lemonsqueezy.com
- **API Docs**: https://docs.lemonsqueezy.com/api
- **Developer Guide**: https://docs.lemonsqueezy.com/guides/developer-guide
- **Best For**: SaaS subscriptions, digital products, global sales with automatic tax handling

#### Polar
- **Overview**: Built for developers, subscription and one-time payments
- **Website**: https://polar.sh
- **API Docs**: https://api.polar.sh/docs
- **SDK**: https://github.com/polarsource/polar
- **Best For**: Developer tools, open-source monetization, subscription management

#### Stripe
- **Overview**: Full-featured payment platform with extensive customization
- **Website**: https://stripe.com
- **API Docs**: https://stripe.com/docs/api
- **Payment Links**: https://stripe.com/docs/payment-links
- **Checkout**: https://stripe.com/docs/payments/checkout
- **Billing**: https://stripe.com/docs/billing
- **Best For**: Complex payment flows, custom billing logic, maximum flexibility

### Implementation Considerations

**Security Best Practices:**
- Never store API keys in client-side code
- Use serverless functions or backend API for payment processing
- Validate webhooks to verify payment status
- Store license keys/subscriptions in a secure database

**Recommended Architecture:**
```typescript
// Client (Framer Plugin) → API Route → Payment Provider
// User clicks "Upgrade" → Call your API → Create checkout session
// Payment successful → Webhook → Activate license → Update database
// Plugin checks license status → Your API → Returns access level
```

**License Validation Pattern:**
```typescript
// Example: Check if user has premium access
const checkLicense = async (email: string) => {
    const response = await fetch('https://your-api.com/check-license', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email })
    })
    const { hasAccess, tier } = await response.json()
    return { hasAccess, tier }
}
```

**Common Paywall Patterns:**
- Feature gating: Limit certain features to premium users
- Usage limits: Free tier with X uses per month
- Time-based trials: 14-day free trial, then payment required
- Freemium: Basic features free, advanced features paid

## Code Style
- Use TypeScript strict mode
- Prefer async/await over promises
- Use functional components with hooks
- Keep components small and focused
- Handle errors gracefully with try/catch
- Use descriptive variable names
- Comment complex logic

## Security Notes
- Plugins run in a sandboxed environment
- Be careful with external API calls
- Validate user input
- Don't expose sensitive data in client code

---
> Source: [julesvcode/framer-plugin-starter-kit](https://github.com/julesvcode/framer-plugin-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
