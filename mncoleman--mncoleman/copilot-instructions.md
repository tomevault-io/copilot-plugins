## mncoleman

> enableSpotlight={true}

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is Matthew Coleman's personal website built with Next.js 16 (App Router) as a static site. It features:

- **Blog** - Notion-powered blog with markdown rendering
- **Resume** - Professional resume/CV from Notion
- **Resource Library** - Curated links to websites and resources from Notion

The site is deployed to GitHub Pages at `https://mncoleman.github.io/mncoleman/`.

## Development Commands

```bash
npm run dev      # Start development server at http://localhost:3000
npm run build    # Build static site (outputs to out/)
npm run start    # Start production server (for testing build)
npm run lint     # Run ESLint
```

### Environment Setup

Create `.env.local` in the root directory with:

```env
# Notion integration token (starts with "ntn_")
NOTION_TOKEN=ntn_your_integration_token_here

# Blog database ID
NOTION_DATABASE_ID=your_blog_database_id_32_characters

# Resources database ID (separate database)
NOTION_RESOURCES_DATABASE_ID=your_resources_database_id_32_chars

# Resume page ID
NOTION_RESUME_PAGE_ID=your_resume_page_id_32_characters
```

**Important**: Notion tokens start with `ntn_`, not `secret_`. The code validates for placeholder values to gracefully fall back to sample data during development.

## Architecture

### Next.js Configuration

- **Output**: Static export (`output: 'export'`)
- **Base Path**: `/mncoleman` in production (GitHub Pages subpath)
- **Images**: Unoptimized (required for static export)
- **Trailing Slash**: Enabled for better compatibility

### Content Management Architecture

Uses a **two-layer adapter pattern** for all Notion content:

1. **lib/notion.ts** - Direct Notion API integration for blog posts
2. **lib/blog.ts** - Thin adapter for blog data
3. **lib/resources.ts** - Adapter for Resources database
4. **lib/resume.ts** - Adapter for Resume page

#### Credential Validation Pattern

All Notion data-fetching functions validate credentials BEFORE calling `getNotionClient()`:

```typescript
export async function getPublishedPosts(): Promise<NotionPost[]> {
    const databaseId = getDatabaseId();
    const token = process.env.NOTION_TOKEN;

    // Validate credentials before attempting connection
    if (!databaseId || !token || token === 'ntn_your_integration_token_here') {
        console.warn('Returning sample data because Notion credentials are not configured');
        return [/* sample data */];
    }

    try {
        const notion = getNotionClient();
        // ... API call
    } catch (error) {
        console.error('Error fetching posts:', error);
        return [/* sample data */];
    }
}
```

This pattern enables graceful degradation with sample data during development.

### Notion Data Sources

#### Blog Database

- Database ID: `your_blog_database_id_32_characters`
- Properties: Title, Slug, Date, Tags, Published, Featured, Excerpt, Author

#### Resources Database (separate database)

- Database ID: `your_resources_database_id_32_chars`
- Properties: Name, URL, Category, Description, Published

#### Resume Page

- Page ID: `your_resume_page_id_32_characters`
- Content: Markdown blocks to be converted

> **Important**: These are three separate Notion data sources. Blog and Resources are separate databases (not the same database with different views).

### Component Organization

- **components/ui/** - shadcn/ui + ReactBits components
  - `dark-veil.tsx` - WebGL background animation using OGL library
- **components/** - Custom components
- **app/** - Next.js App Router pages
- **worker/** - Cloudflare Worker for Admin Authentication & API

### Styling System

- **Tailwind CSS** with CSS variables for theming
- **Dark mode**: `next-themes` with class-based toggle
- **Component library**: shadcn/ui (New York style) + ReactBits registry
- **Frosted Glass Effect**: `bg-background/40 backdrop-blur-xl` for semi-transparent cards over Dark Veil

### Dark Veil Background Component

The home page uses a WebGL-based animated background (Dark Veil from ReactBits):

#### Key Implementation Details

- Canvas must use `position: fixed` with explicit `100vw/100vh` sizing
- Set `zIndex: -1` to keep background behind content
- Use `window.innerWidth` and `window.innerHeight` for resize calculations (not parent dimensions)
- Add `overflow-x: hidden` to html/body in globals.css to prevent cutoff
- Render directly without wrapper divs to avoid positioning conflicts
- The `resolutionScale` prop only affects render quality, not visual size

#### Example Usage

```typescript
<DarkVeil hueShift={40} speed={0.5} resolutionScale={0.8} />
```

#### Bento Cards Over Dark Veil

Use frosted glass transparency to show background colors:

```typescript
className="bg-background/40 backdrop-blur-xl border border-border/30"
```

---

## IMPLEMENTATION PLANS

The following plans should be executed by an AI model (Claude Sonnet or similar) to update this site.

---

## PLAN 1: Health Check & Optimization

### Findings

#### Current State Assessment

- File structure is clean and minimal
- Dependencies are appropriate for the project
- No major bloat or unnecessary files

#### Issues Identified

1. **Unused Dependency**: `gray-matter` package in dependencies may be unused (was for markdown frontmatter before Notion migration)

2. **Script Not Runnable**: `scripts/generate-icons.js` requires `sharp` but it's not in dependencies (icons already generated, script can be removed)

3. **Documentation Redundancy**: Multiple setup docs (GOOGLE_ANALYTICS_SETUP.md, NOTION_SETUP.md, REFRESH_SETUP.md) could be consolidated into README.md

4. **Dead API Route**: `app/api/refresh-posts/route.ts` doesn't work on GitHub Pages (static export)

5. **Theme Toggle Missing**: No theme toggle button in the layout header

### Cleanup Actions

Execute these steps in order:

#### Step 1: Remove Unused Dependencies

```bash
npm uninstall gray-matter
```

#### Step 2: Remove Unused Files

- Delete `scripts/generate-icons.js`
- Delete `scripts/` directory (empty after removal)
- Delete `app/api/refresh-posts/route.ts`
- Delete `app/api/` directory (empty after removal)

#### Step 3: Consolidate Documentation

- Move essential content from GOOGLE_ANALYTICS_SETUP.md, NOTION_SETUP.md, REFRESH_SETUP.md into README.md under appropriate sections
- Delete the individual setup files

#### Step 4: Verify Build

```bash
npm run build
npm run lint
```

---

## PLAN 2: Magic Bento Home Page Implementation

### Overview

Transform the home page into an interactive Magic Bento grid layout that serves as an information hub for Matthew Coleman's personal website with tiles for:

1. **Hero/Introduction** - Name, tagline, brief intro
2. **Blog** - Latest posts preview
3. **Resume** - Professional summary with link
4. **Resources** - Featured resources preview

### Prerequisites

**Install GSAP** (required by MagicBento):

```bash
npm install gsap
```

**Install MagicBento Component**:

```bash
npx shadcn@latest add @react-bits/MagicBento-TS-TW
```

### Implementation Steps

#### Step 1: Create Resources Data Layer

Create `lib/resources.ts`:

```typescript
import { Client } from '@notionhq/client';

const getNotionClient = () => {
    if (!process.env.NOTION_TOKEN) {
        throw new Error('NOTION_TOKEN is not defined');
    }
    return new Client({ auth: process.env.NOTION_TOKEN });
};

export interface Resource {
    id: string;
    name: string;
    url: string;
    category: string;
    description: string;
    published: boolean;
}

export async function getPublishedResources(): Promise<Resource[]> {
    const databaseId = process.env.NOTION_RESOURCES_DATABASE_ID;
    if (!databaseId) {
        console.warn('NOTION_RESOURCES_DATABASE_ID not set, returning sample data');
        return [
            {
                id: 'sample-1',
                name: 'Sample Resource',
                url: 'https://example.com',
                category: 'Sample',
                description: 'This is a sample resource.',
                published: true
            }
        ];
    }

    try {
        const notion = getNotionClient();
        const response = await notion.databases.query({
            database_id: databaseId,
            filter: {
                property: 'Published',
                checkbox: { equals: true }
            }
        });

        return response.results.map((page: any) => ({
            id: page.id,
            name: page.properties.Name?.title?.[0]?.plain_text || 'Untitled',
            url: page.properties.URL?.url || '',
            category: page.properties.Category?.select?.name || 'Uncategorized',
            description: page.properties.Description?.rich_text?.[0]?.plain_text || '',
            published: page.properties.Published?.checkbox || false
        }));
    } catch (error) {
        console.error('Error fetching resources:', error);
        return [];
    }
}

export async function getResourcesByCategory(): Promise<Record<string, Resource[]>> {
    const resources = await getPublishedResources();
    return resources.reduce((acc, resource) => {
        const category = resource.category;
        if (!acc[category]) acc[category] = [];
        acc[category].push(resource);
        return acc;
    }, {} as Record<string, Resource[]>);
}
```

#### Step 2: Create Resume Data Layer

Create `lib/resume.ts`:

```typescript
import { Client } from '@notionhq/client';
import { NotionToMarkdown } from 'notion-to-md';

const getNotionClient = () => {
    if (!process.env.NOTION_TOKEN) {
        throw new Error('NOTION_TOKEN is not defined');
    }
    return new Client({ auth: process.env.NOTION_TOKEN });
};

export interface Resume {
    title: string;
    content: string;
    lastUpdated: string;
}

export async function getResume(): Promise<Resume | null> {
    const pageId = process.env.NOTION_RESUME_PAGE_ID;
    if (!pageId) {
        console.warn('NOTION_RESUME_PAGE_ID not set');
        return {
            title: 'Resume',
            content: '# Resume\n\nResume content will appear here once configured.',
            lastUpdated: new Date().toISOString()
        };
    }

    try {
        const notion = getNotionClient();
        const n2m = new NotionToMarkdown({ notionClient: notion });

        const page = await notion.pages.retrieve({ page_id: pageId }) as any;
        const mdblocks = await n2m.pageToMarkdown(pageId);
        const mdString = n2m.toMarkdownString(mdblocks);

        return {
            title: page.properties?.title?.title?.[0]?.plain_text || 'Resume',
            content: mdString.parent,
            lastUpdated: page.last_edited_time
        };
    } catch (error) {
        console.error('Error fetching resume:', error);
        return null;
    }
}
```

#### Step 3: Create New Pages

**Create `app/resources/page.tsx`**:

```typescript
import { getResourcesByCategory } from '@/lib/resources';
import Link from 'next/link';
import { ExternalLink } from 'lucide-react';

export default async function ResourcesPage() {
    const resourcesByCategory = await getResourcesByCategory();
    const categories = Object.keys(resourcesByCategory).sort();

    return (
        <div className="container mx-auto px-4 py-16 max-w-4xl">
            <h1 className="text-4xl font-bold mb-8 tracking-tight">Resources</h1>
            <p className="text-lg text-muted-foreground mb-12">
                A curated collection of useful websites, tools, and resources.
            </p>

            {categories.map(category => (
                <section key={category} className="mb-12">
                    <h2 className="text-2xl font-semibold mb-6">{category}</h2>
                    <div className="grid gap-4">
                        {resourcesByCategory[category].map(resource => (
                            <Link
                                key={resource.id}
                                href={resource.url}
                                target="_blank"
                                rel="noopener noreferrer"
                                className="p-4 border rounded-lg hover:bg-accent transition-colors group"
                            >
                                <div className="flex items-start justify-between gap-4">
                                    <div>
                                        <h3 className="font-medium group-hover:text-primary transition-colors">
                                            {resource.name}
                                        </h3>
                                        <p className="text-sm text-muted-foreground mt-1">
                                            {resource.description}
                                        </p>
                                    </div>
                                    <ExternalLink className="h-4 w-4 text-muted-foreground shrink-0 mt-1" />
                                </div>
                            </Link>
                        ))}
                    </div>
                </section>
            ))}
        </div>
    );
}
```

**Create `app/resume/page.tsx`**:

```typescript
import { getResume } from '@/lib/resume';
import ReactMarkdown from 'react-markdown';

export default async function ResumePage() {
    const resume = await getResume();

    if (!resume) {
        return (
            <div className="container mx-auto px-4 py-16 max-w-4xl">
                <h1 className="text-4xl font-bold mb-8">Resume</h1>
                <p className="text-muted-foreground">Resume not available.</p>
            </div>
        );
    }

    return (
        <div className="container mx-auto px-4 py-16 max-w-4xl">
            <article className="prose prose-neutral dark:prose-invert max-w-none">
                <ReactMarkdown>{resume.content}</ReactMarkdown>
            </article>
        </div>
    );
}
```

#### Step 4: Update Layout Navigation

In `app/layout.tsx`, update the navigation to include new pages:

```typescript
<nav className="flex gap-6 items-center">
    <Link href="/" className="font-semibold text-lg hover:text-muted-foreground transition-colors">
        Matthew Coleman
    </Link>
    <Link href="/blog" className="text-sm hover:text-muted-foreground transition-colors">
        Blog
    </Link>
    <Link href="/resources" className="text-sm hover:text-muted-foreground transition-colors">
        Resources
    </Link>
    <Link href="/resume" className="text-sm hover:text-muted-foreground transition-colors">
        Resume
    </Link>
    <Link href="/about" className="text-sm hover:text-muted-foreground transition-colors">
        About
    </Link>
</nav>
```

#### Step 5: Create Magic Bento Home Page

Replace `app/page.tsx` with a new Magic Bento layout:

```typescript
'use client';

import { useRef } from 'react';
import Link from 'next/link';
import { ArrowRight, FileText, BookOpen, Link2 } from 'lucide-react';
import MagicBento from '@/components/ui/magic-bento';

// Bento card data for the personal website
const bentoCards = [
    {
        id: 'hero',
        title: 'Matthew Coleman',
        description: 'Welcome to my personal website. I write about technology, share resources, and document my professional journey.',
        label: 'Introduction',
        color: '#060010',
        span: 'col-span-2 row-span-2', // Large hero tile
        link: '/about'
    },
    {
        id: 'blog',
        title: 'Blog',
        description: 'Thoughts on technology, life, and sometimes just random things.',
        label: 'Articles',
        color: '#060010',
        icon: BookOpen,
        span: 'col-span-1 row-span-1',
        link: '/blog'
    },
    {
        id: 'resources',
        title: 'Resources',
        description: 'Curated collection of useful websites and tools.',
        label: 'Library',
        color: '#060010',
        icon: Link2,
        span: 'col-span-1 row-span-1',
        link: '/resources'
    },
    {
        id: 'resume',
        title: 'Resume',
        description: 'Professional experience and qualifications.',
        label: 'Career',
        color: '#060010',
        icon: FileText,
        span: 'col-span-2 row-span-1',
        link: '/resume'
    }
];

export default function Home() {
    const gridRef = useRef<HTMLDivElement>(null);

    return (
        <div className="min-h-screen flex items-center justify-center py-16">
            <MagicBento
                enableSpotlight={true}
                enableBorderGlow={true}
                enableTilt={false}
                clickEffect={true}
                enableMagnetism={true}
                glowColor="132, 0, 255"
            >
                <div
                    ref={gridRef}
                    className="grid grid-cols-1 md:grid-cols-3 gap-4 p-6 max-w-5xl"
                >
                    {bentoCards.map((card) => (
                        <Link
                            key={card.id}
                            href={card.link}
                            className={`${card.span} card relative overflow-hidden rounded-2xl border border-border/50 bg-card p-6 transition-all hover:border-primary/50`}
                            style={{
                                '--glow-x': '50%',
                                '--glow-y': '50%',
                                '--glow-intensity': '0',
                                '--glow-radius': '200px'
                            } as React.CSSProperties}
                        >
                            <div className="flex flex-col h-full justify-between">
                                <div>
                                    <span className="text-xs font-medium text-muted-foreground uppercase tracking-wider mb-2 block">
                                        {card.label}
                                    </span>
                                    <h2 className="text-2xl font-bold mb-2">{card.title}</h2>
                                    <p className="text-muted-foreground">{card.description}</p>
                                </div>
                                <div className="flex items-center gap-2 mt-4 text-sm font-medium text-primary">
                                    Explore
                                    <ArrowRight className="h-4 w-4" />
                                </div>
                            </div>
                            {card.icon && (
                                <card.icon className="absolute bottom-6 right-6 h-12 w-12 text-muted-foreground/20" />
                            )}
                        </Link>
                    ))}
                </div>
            </MagicBento>
        </div>
    );
}
```

**Note**: The exact MagicBento implementation may need adjustment based on the actual component API after installation. Review the installed component in `components/ui/magic-bento.tsx` and adapt the usage accordingly.

#### Step 6: Update Environment Variables

Add to `.env.local`:

```env
NOTION_RESOURCES_DATABASE_ID=your_resources_database_id_32_chars
NOTION_RESUME_PAGE_ID=your_resume_page_id_32_characters
```

Add to GitHub Secrets:

- `NOTION_RESOURCES_DATABASE_ID`
- `NOTION_RESUME_PAGE_ID`

#### Step 7: Test and Verify

```bash
npm run dev     # Test locally
npm run build   # Verify static build works
npm run lint    # Check for issues
```

### Alternative: Simpler Bento Grid (No Animation) - IMPLEMENTED

The site uses a simpler CSS Grid-based bento layout without GSAP animations. This approach provides better performance and simpler maintenance.

**Implementation** (`app/page.tsx`):

```typescript
'use client';

import Link from 'next/link';
import { ArrowRight, FileText, BookOpen, Link2, User } from 'lucide-react';
import DarkVeil from '@/components/ui/dark-veil';

const bentoCards = [
  {
    id: 'hero',
    title: 'Matthew Coleman',
    description: 'Welcome to my personal website...',
    label: 'Introduction',
    span: 'md:col-span-2 md:row-span-1',
    link: '/about',
    icon: User
  },
  // ... other cards
];

export default function Home() {
  return (
    <>
      <DarkVeil hueShift={40} speed={0.5} resolutionScale={0.8} />

      <div className="min-h-screen flex items-center justify-center py-16 px-4 relative">
        <div className="w-full max-w-5xl relative z-10">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4 auto-rows-fr">
            {bentoCards.map((card) => (
              <Link
                key={card.id}
                href={card.link}
                className={`${card.span} group relative overflow-hidden rounded-2xl
                  border border-border/30 bg-background/40 backdrop-blur-xl p-8
                  transition-all hover:border-primary/50 hover:bg-background/50`}
              >
                {/* Card content */}
              </Link>
            ))}
          </div>
        </div>
      </div>
    </>
  );
}
```

Key features:

- Simple CSS Grid layout with responsive spans
- Frosted glass cards (`bg-background/40 backdrop-blur-xl`)
- Dark Veil animated background
- Hover effects with Tailwind transitions

---

## PLAN 3: Database Schema Reference

### Blog Database Properties

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| Title | title | Yes | Post title |
| Slug | text | Yes | URL-friendly identifier |
| Date | date | Yes | Publication date |
| Tags | multi-select | No | Categories |
| Published | checkbox | Yes | Visibility control |
| Featured | checkbox | No | Pin to top |
| Excerpt | text | No | Short summary |
| Author | text | No | Author name |

### Resources Database Properties

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| Name | title | Yes | Resource name |
| URL | url | Yes | Link to resource |
| Category | select | Yes | Resource category |
| Description | text | No | Brief description |
| Published | checkbox | Yes | Visibility control |

### Resume Page Schema

- Single Notion page with markdown content
- Fetched and converted to markdown at build time

---

## Common Pitfalls

1. **Base Path**: Always account for `/mncoleman` prefix in production. Use Next.js `<Link>` component for navigation.

2. **Image Optimization**: Disabled for static export. Use `<img>` tags or unoptimized Next.js `<Image>` component.

3. **GSAP in SSR**: MagicBento uses GSAP which requires client-side rendering. Always use `'use client'` directive for pages/components using it.

4. **Environment Variables**: Build-time only. All Notion IDs must be in GitHub Secrets for production builds.

5. **Notion Content Updates**: Require rebuild to appear on site. Not real-time.

6. **Notion Token Format**: Tokens start with `ntn_` NOT `secret_`. Update any validation or documentation accordingly.

7. **Notion Credential Validation**: Always validate credentials BEFORE calling `getNotionClient()` to enable graceful fallback to sample data. Check for both undefined values and placeholder strings (e.g., `ntn_your_integration_token_here`).

8. **Dark Veil Canvas Coverage**:
   - Do NOT wrap the canvas in positioned containers
   - Use `position: fixed` with explicit `100vw/100vh` sizing
   - Use `window.innerWidth/innerHeight` for resize calculations, not parent element dimensions
   - Add `overflow-x: hidden` to html/body to prevent scrollbar issues
   - The `resolutionScale` prop affects render resolution only, not visual size

9. **Separate Notion Databases**: Blog, Resources, and Resume use separate database IDs. Ensure each has the correct ID configured. The error "Databases with multiple data sources are not supported" indicates you're trying to use a database with synced blocks or linked databases.

---

## Execution Order for Implementation

When implementing these plans, follow this order:

1. **Run Health Check Cleanup** (Plan 1)
2. **Install Dependencies** (gsap, MagicBento)
3. **Create Data Layers** (lib/resources.ts, lib/resume.ts)
4. **Create New Pages** (resources, resume)
5. **Update Layout Navigation**
6. **Implement Magic Bento Home Page**
7. **Update Environment Variables**
8. **Test and Deploy**

---

## Adding shadcn/ui Components

Two registries configured in `components.json`:

```bash
npx shadcn@latest add button              # shadcn/ui
npx shadcn@latest add @react-bits/avatar  # ReactBits
npx shadcn@latest add @react-bits/MagicBento-TS-TW  # Magic Bento
```

---
> Source: [mncoleman/mncoleman](https://github.com/mncoleman/mncoleman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
