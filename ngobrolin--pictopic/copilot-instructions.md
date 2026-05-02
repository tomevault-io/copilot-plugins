## pictopic

> **Ngobrolin Web Topic Picker** is a static web application designed for selecting podcast discussion topics. The application provides an interactive interface for browsing and randomly selecting topics from the Ngobrolin podcast's GitHub Discussions repository.

# Ngobrolin Web Topic Picker - Agent Documentation

## Project Overview

**Ngobrolin Web Topic Picker** is a static web application designed for selecting podcast discussion topics. The application provides an interactive interface for browsing and randomly selecting topics from the Ngobrolin podcast's GitHub Discussions repository.

### Key Features

- **Random Topic Picker**: Click-to-select functionality with visual animation
- **Spin the Wheel**: Interactive wheel interface for fun topic selection
- **Topic Browsing**: Grid view of all topics with sorting capabilities
- **Statistics Dashboard**: Real-time stats showing total topics, votes, comments, and contributors
- **GitHub Sync**: Optional sync button to fetch live data from GitHub Discussions
- **Topic History Tracking**: Remembers picked topics across sessions using localStorage
- **Smart Skip Logic**: Automatically excludes previously picked topics from random selection
- **Reset History**: Clear pick history to start fresh
- **Responsive Design**: Mobile-friendly interface using Tailwind CSS

## Technology Stack

### Core Technologies

- **Astro 5.16.15**: Modern static site generator with component-based architecture
- **Tailwind CSS 4.1.18**: Utility-first CSS framework for rapid UI development
  - Uses Vite plugin integration (`@tailwindcss/vite`)
- **TypeScript**: Strict mode enabled for type safety
- **Vite**: Build tool and development server (included with Astro)

### Supporting Libraries

- **@astrojs/node 9.5.2**: Node.js integration for API routes
- **cheerio 1.2.0**: HTML parsing for GitHub Discussions scraping

## Architecture

### Static-First Design Philosophy

The application is built with a static-first approach:

1. **Static Data Source**: Topics are stored in `/src/data/topics.ts` as a TypeScript array
2. **Client-Side Rendering**: All interactivity happens in the browser
3. **Optional API**: An API route exists for live GitHub sync but is not required
4. **Build-Time Optimization**: Can be fully pre-rendered as static HTML

### Data Flow

```
Static Topics (src/data/topics.ts)
    ↓
Browser Load (Client-Side)
    ↓
Display & Interact (index.astro)
    ↓
[Optional] GitHub Sync Button
    ↓
API Route (src/pages/api/topics.ts) [Optional]
    ↓
Update Client State
```

### Key Architectural Principles

1. **Keep It Static**: No dynamic fetching on page load - everything starts from static data
2. **Client-Side Only**: All interactivity, sorting, and selection happens in the browser
3. **Optional Enhancement**: GitHub sync is user-initiated, not automatic
4. **Type Safety**: TypeScript strict mode catches errors at build time
5. **Component-Based**: Reusable Astro components for maintainability

## File Structure

```
ngobrolin-web-topic-picker/
├── public/                      # Static assets (if any)
├── src/
│   ├── components/
│   │   └── TopicCard.astro     # Reusable topic card component
│   ├── data/
│   │   └── topics.ts           # Static topics data (main data source)
│   ├── layouts/
│   │   └── Layout.astro        # Main layout wrapper
│   ├── lib/
│   │   └── topicHistory.ts     # localStorage utility for pick history
│   ├── pages/
│   │   ├── api/
│   │   │   └── topics.ts       # Optional API route for GitHub sync
│   │   └── index.astro         # Main application page
│   └── types/
│       └── topic.ts            # TypeScript type definitions
├── astro.config.mjs            # Astro configuration
├── package.json                # Dependencies and scripts
├── tailwind.config.js          # Tailwind CSS configuration
├── tsconfig.json               # TypeScript configuration (strict mode)
└── AGENTS.md                   # This file
```

### Key Files Explained

#### `/src/data/topics.ts`

- **Purpose**: Single source of truth for all topics
- **Format**: TypeScript array of `Topic` objects
- **Import**: Used directly in `index.astro` for initial render
- **Update**: Manually maintained or updated via GitHub sync

#### `/src/lib/topicHistory.ts`

- **Purpose**: localStorage utility module for pick history
- **Functions**: getPickedTopicIds(), addPickedTopic(), isTopicPicked(), clearHistory()
- **Usage**: Imported by index.astro for history management

#### `/src/pages/index.astro`

- **Purpose**: Main application interface
- **Contains**: HTML structure, client-side JavaScript, and styling
- **Features**:
  - Topic picker with animation
  - Wheel spinner with canvas rendering
  - Statistics dashboard
  - Sorting controls
  - GitHub sync button
  - History tracking and reset

#### `/src/pages/api/topics.ts`

- **Purpose**: Optional API route for live GitHub data
- **Usage**: Called only when user clicks "Sync from GitHub" button
- **Function**: Scrapes GitHub Discussions and returns updated topics

#### `/src/types/topic.ts`

- **Purpose**: TypeScript interface definition
- **Interface**: `Topic` with properties (id, title, url, author, votes, comments, category)

## Development Setup

### Prerequisites

- Node.js 18+ (recommended)
- npm or yarn package manager

### Installation

```bash
# Install dependencies
npm install
```

### Development

```bash
# Start development server
npm run dev

# Server runs at http://localhost:4321
# Hot reload enabled for rapid development
```

### Build

```bash
# Build for production
npm run build

# Output directory: dist/
# Optimized static files ready for deployment
```

### Preview Production Build

```bash
# Preview production build locally
npm run preview

# Server runs at http://localhost:4321
# Serves from dist/ directory
```

## Topic History Feature

### How It Works

The topic picker remembers which topics have been selected and automatically excludes them from future random picks. History is stored in the browser's localStorage and persists across sessions.

### Storage

- **Location**: Browser localStorage
- **Key**: `ngobrolin-topic-picker-history`
- **Structure**: `{ pickedTopics: number[], lastUpdated: number }`

### Features

1. **Automatic Tracking**: Every time a topic is picked (random or wheel), it's added to history
2. **Visual Indicators**: Picked topics appear grayed out with a "Picked" badge
3. **Reset Button**: Appears when there are picked topics, allowing users to clear history
4. **All Picked Message**: When all topics are picked, a message prompts user to reset

### Reset History

Users can clear all pick history by clicking the "Reset History" button that appears in the sort controls section. A confirmation dialog prevents accidental clearing.

## Adding Topics

### Manual Method (Recommended)

Topics should be added to `/src/data/topics.ts`:

```typescript
import type { Topic } from "../types/topic";

export const topics: Topic[] = [
  {
    id: 95, // Unique discussion ID
    title: "Your Topic Title", // Discussion title
    url: "https://github.com/orgs/ngobrolin/discussions/95",
    author: "username", // GitHub username
    votes: 0, // Current vote count
    comments: 0, // Current comment count
    category: "optional", // Optional category field
  },
  // ... existing topics
];
```

### Topic Object Properties

| Property   | Type     | Required | Description                   |
| ---------- | -------- | -------- | ----------------------------- |
| `id`       | `number` | Yes      | Unique GitHub Discussion ID   |
| `title`    | `string` | Yes      | Discussion title              |
| `url`      | `string` | Yes      | Full URL to GitHub Discussion |
| `author`   | `string` | Yes      | GitHub username of author     |
| `votes`    | `number` | Yes      | Number of upvotes             |
| `comments` | `number` | Yes      | Number of comments            |
| `category` | `string` | No       | Optional category tag         |

### GitHub Sync Method (Optional)

Users can click the "Sync from GitHub" button to fetch live data from GitHub Discussions. This:

- Calls `/api/topics` endpoint
- Scrapes the latest discussions
- Updates the client-side state
- Does NOT modify the static `topics.ts` file

## Agent Instructions

### For AI Agents Working on This Codebase

#### Core Principles

1. **KEEP IT STATIC**

   - Do NOT add automatic fetching on page load
   - Do NOT convert to SSR (Server-Side Rendering)
   - Do NOT add build-time data fetching
   - Topics must come from `/src/data/topics.ts` initially

2. **DATA SOURCE RULES**

   - Default data source: `/src/data/topics.ts`
   - API routes are OPTIONAL enhancements only
   - Never require the API for the app to function
   - The app must work with static data alone

3. **STYLING GUIDELINES**

   - Use Tailwind CSS utility classes
   - Follow existing color scheme (purple/pink gradients)
   - Maintain responsive design patterns
   - Use existing Tailwind classes from `index.astro`

4. **COMPONENT PATTERNS**

   - Create reusable Astro components in `/src/components/`
   - Use `.astro` files for components with HTML/JS
   - Use `.ts` files for pure logic/utilities
   - Follow the `TopicCard.astro` pattern

5. **TYPESCRIPT STRICT MODE**
   - Type all variables and function parameters
   - Use interfaces from `/src/types/`
   - Enable strict type checking
   - No `any` types without explicit justification

#### What Agents SHOULD Do

- Add new features that work with static data
- Improve UI/UX with client-side JavaScript
- Create reusable components
- Add sorting/filtering capabilities
- Enhance accessibility
- Optimize performance
- Add tests for functionality
- Improve type definitions
- Integrate with localStorage for client-side persistence
- Use topicHistory module for history management

#### What Agents Should NOT Do

- Remove or bypass the static data source
- Add API calls to the initial page load
- Convert to SSR or hybrid rendering
- Add database dependencies
- Remove the TypeScript strict mode
- Break the static-first architecture
- Make the app dependent on external APIs

#### Example: Adding a Feature

Good (Static-First):

```typescript
// Add to client-side script
function filterByAuthor(author: string) {
  const filtered = staticTopics.filter((t) => t.author === author);
  renderTopics(filtered);
}
```

Bad (Breaking Architecture):

```typescript
// Don't do this - removes static data
async function loadTopics() {
  const response = await fetch("/api/topics");
  topics = await response.json();
}
```

#### Code Style Guidelines

- **File Naming**: PascalCase for components (e.g., `TopicCard.astro`)
- **Variable Naming**: camelCase for variables and functions
- **Type Naming**: PascalCase for interfaces and types
- **Indentation**: 2 spaces (consistent with Astro default)
- **Quotes**: Single quotes for strings, double quotes for HTML attributes

#### Testing Changes

Before submitting changes:

1. Run `npm run dev` to test locally
2. Verify static data loads correctly
3. Test all interactive features
4. Check mobile responsiveness
5. Ensure TypeScript compiles without errors
6. Build with `npm run build` to verify production build

#### Common Tasks

**Add a new topic:**

1. Open `/src/data/topics.ts`
2. Add new topic object to the array
3. Ensure unique `id` and valid `url`

**Create a new component:**

1. Create file in `/src/components/`
2. Use `.astro` extension for UI components
3. Import and use in pages or other components

**Add client-side interactivity:**

1. Add to `<script>` tag in `.astro` files
2. Use DOM APIs directly (no framework needed)
3. Access static data via imports

**Update styles:**

1. Use existing Tailwind utility classes
2. Check `index.astro` for color/spacing patterns
3. Maintain responsive breakpoints (md:, lg:)

**Reset topic pick history:**

- Click "Reset History" button in sort controls
- Confirms with dialog before clearing
- All topics become available for picking again

## Deployment

### Static Hosting (Recommended)

The app can be deployed to any static hosting service:

- Vercel
- Netlify
- GitHub Pages
- Cloudflare Pages

```bash
# Build and deploy
npm run build
# Upload dist/ directory to your hosting provider
```

### Node.js Hosting

If using the API route for GitHub sync:

- Deploy to Node.js hosting (Vercel, Netlify, Railway)
- Ensure serverless function support
- The app still works statically if API fails

## Troubleshooting

### Common Issues

**Topics not loading:**

- Verify `/src/data/topics.ts` exists and is valid
- Check browser console for errors
- Ensure TypeScript compilation succeeded

**Build errors:**

- Run `npm install` to ensure dependencies
- Check TypeScript strict mode violations
- Verify all imports are correct

**Styling issues:**

- Confirm Tailwind CSS is configured
- Check browser compatibility
- Verify utility class names are correct

### Getting Help

- Check Astro documentation: https://docs.astro.build
- Tailwind CSS docs: https://tailwindcss.com/docs
- GitHub Issues: Check project repository

---

**Last Updated**: January 2026
**Maintained By**: Ngobrolin Team
**License**: See project repository

---
> Source: [ngobrolin/pictopic](https://github.com/ngobrolin/pictopic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
