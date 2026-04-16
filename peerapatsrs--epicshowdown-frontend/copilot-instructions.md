## epicshowdown-frontend

> Rules and structure for EpicShowdown Frontend (SvelteKit) focused mainly on client-side execution.

@description
Rules and structure for EpicShowdown Frontend (SvelteKit) focused mainly on client-side execution.

@Instruction
This document defines the project structure and coding guidelines for EpicShowdown Frontend. Client-side prioritized. Server-side should only be used if absolutely necessary.

@Rule "Project Structure"
- src/routes/ : Page components
- src/lib/ : UI Components, Utilities, Stores
- src/lib/components/ : Reusable components
- src/lib/stores/ : Global reactive states
- src/lib/utils/ : Utility functions
- src/assets/ : Images, Fonts, Static assets

@Rule "File Naming Conventions"
- Pages (routes) : kebab-case (e.g., user-profile.svelte)
- Components : PascalCase (e.g., UserProfile.svelte)
- Stores/Utils : camelCase (e.g., authStore.ts, formatDate.ts)

@Rule "Coding Style"
- Use TypeScript everywhere (<script lang="ts">)
- Use async/await for all asynchronous operations
- Handle all fetch errors with try/catch
- Prefer const over let
- Auto format with Prettier
- No direct magic numbers/strings; move constants to lib/constants.ts

@Rule "Server-Side Usage"
- Server-side (+page.server.ts, endpoints) must be avoided unless absolutely necessary (e.g., secure authentication)
- All data fetching should be done client-side using fetch inside onMount or load (+page.ts)
- Prioritize fetch from frontend to APIs

@Example
```ts
// +page.ts
export async function load({ fetch }) {
  const res = await fetch('/api/products');
  const data = await res.json();
  return { products: data };
}
```

@Rule "Date and Time"
- Assume all dates from backend are UTC.
- Display dates in user's local timezone using libraries like dayjs.

@Example
```ts
import dayjs from 'dayjs';

const localDate = dayjs.utc(order.createdAt).local().format('YYYY-MM-DD HH:mm');
```

@Rule "Error Handling"
- Always wrap API calls inside try/catch
- Display user-friendly errors
- Do not crash the UI if an API call fails

@Example
```ts
try {
  const res = await fetch('/api/user');
  if (!res.ok) throw new Error('Failed to load user');
  const data = await res.json();
} catch (err) {
  console.error('Error loading user:', err);
}
```

@Rule "Environment Variables (env)"
- All environment variables must be accessed through the `$env/static/public` module.
- Only variables prefixed with `PUBLIC_` are allowed for client-side usage.
- Sensitive keys must never be exposed to the client.

@Example
```ts
// Correct usage
import { PUBLIC_API_URL } from '$env/static/public';

const response = await fetch(`${PUBLIC_API_URL}/products`);
```

@Rule "Security Basics"
- Do not trust client-side input
- Do not store sensitive information directly in localStorage
- Use sessionStorage/localStorage only for non-critical information (e.g., theme preference)

@Rule "Design & Style Guide"
- Primary Colors:
  - Background: #1a1625 (dark purple)
  - Secondary Background: #251f35 (darker purple)
  - Primary Highlight: gradient from #ff6b2b (orange) to #ee0979 (bright pink)
- Gradient Usage:
  - Primary buttons and important components use bg-gradient-to-r from-[#ff6b2b] to-[#ee0979]
  - On hover: Use hover:from-[#ff8f59] hover:to-[#ff2a90] to emphasize interactions
- Text Colors:
  - Primary Text: text-white
  - Secondary Text: text-gray-400
  - Emphasized Text: text-[#ff6b2b]
- Layout:
  - Use Tailwind CSS for responsive design
  - Grid and Flex layout
  - Use rounded corners (rounded-lg, rounded-full) and shadow effects (shadow-lg)
- Components Style:
  - Card: dark background with shadow and border
  - Buttons: rounded-full with gradient
  - Navigation: supports both desktop and mobile
  - Has hover effects consistent with theme
- Animations & Transitions:
  - Use transition for hover effects
  - Has slide show with fade transition
  - Use opacity transitions for slide indicators
- Dark Theme:
  - Dark theme by default
  - Use dark background to emphasize modernity
  - Has gradient from dark to lighter colors for depth
- Responsive Design:
  - Use Tailwind breakpoints (sm:, lg:)
  - Layout adjusts to screen size
  - Navigation separates mobile and desktop
- Special Effects:
  - Has gradient overlays on images
  - Has hover states consistent with theme
  - Use opacity and transitions for smooth effects

Overall, the design of Epic Showdown emphasizes modernity, dark theme, and uses orange-pink as a highlight, suitable for gaming/entertainment platform

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peerapatsrs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
