## home

> Contains the schema of the data objects used in the website, currently have hackathons.json and team.json

# Data Structure and Management

## Overview
The VietRocHack website uses JSON files in the `src/data` directory to store information about hackathons, projects, team members, and other content.

## Data Files

- **hackathons.json** - Contains all hackathon events, their associated projects, and site statistics
- **team.json** - Contains information about all team members

## Data Models

### Hackathon Object

```typescript
interface Hackathon {
  id: string;                  // Unique identifier (e.g., "hophacks-2024")
  name: string;                // Display name (e.g., "HopHacks Fall 2024")
  year: number;                // Year (e.g., 2024)
  date: string;                // Human-readable date (e.g., "September 2024")
  location: string;            // Full location name (e.g., "Johns Hopkins University")
  city: string;                // City (e.g., "Baltimore")
  state: string;               // State abbreviation (e.g., "MD")
  achievement: string;         // Main achievement (e.g., "Winner General Prize")
  description: string;         // Brief description of the hackathon
  mainImage: string;           // Path to main image
  website: string;             // URL to hackathon website or devpost
  photos: Photo[];             // Array of photo objects
  projects: Project[];         // Array of project objects
}
```

### Project Object

```typescript
interface Project {
  id: string;                  // Unique identifier (e.g., "youngheroes")
  name: string;                // Display name (e.g., "Young Heroes")
  tagline: string;             // Short one-liner about the project
  description: string;         // Detailed description
  thumbnail: string;           // Path to thumbnail image
  demoLink: string;            // URL to live demo (can be empty)
  githubLink: string;          // URL to GitHub repository (can be empty)
  devpostLink: string;         // URL to Devpost submission
  youtubeDemo: string | null;  // URL to YouTube demo video (can be null)
  techStack: string[];         // Array of technologies used
  achievements: string[];      // Array of achievements/awards
  teamMembers: string[];       // Array of team member IDs
  screenshots: string[];       // Array of screenshot image paths
  featured: boolean;           // Whether to feature on homepage
}
```

### Team Member Object

```typescript
interface TeamMember {
  id: string;                  // Unique identifier (e.g., "vuong")
  name: string;                // Full name (e.g., "Vuong Nguyen")
  role: string;                // Role title (e.g., "Full-stack Developer")
  school: string;              // School name (e.g., "University of Rochester")
  major: string;               // Major (e.g., "Computer Science")
  graduation: number;          // Graduation year (e.g., 2025)
  bio: string;                 // Brief biography
  photo: string;               // Path to profile photo
  socialLinks: {               // Object containing social links
    github?: string;           // GitHub profile URL
    linkedin?: string;         // LinkedIn profile URL
    twitter?: string;          // Twitter profile URL (optional)
  };
  skills: string[];            // Array of skills/technologies
}
```

### Photo Object

```typescript
interface Photo {
  src: string;                 // Path to image file
  caption: string;             // Caption for the image
  isFeatured: boolean;         // Whether to feature in highlights
}
```

## File Paths and Naming Conventions

### Image Files

- **Project thumbnails**: `/public/projects/{hackathon_id}_{project_id}_thumbnail.{jpg|png}`
  Example: `/public/projects/hophacks_2024_youngheroes_thumbnail.png`

- **Team photos**: `/public/team/{member_id}.jpg`
  Example: `/public/team/vuong.jpg`

- **Hackathon/Event photos**: `/public/team/{hackathon_id}_{descriptive_name}.{jpg|png}`
  Example: `/public/team/hophacks_2024_demo_team_pic.jpg`

- **Project screenshots**: `/public/projects/{project_id}/screenshot{n}.{jpg|png}`
  Example: `/public/projects/youngheroes-screenshot1.jpg`

### Image Path Handling

All image paths in the data files should use a consistent format:

- For paths in the public directory, include the path without the leading slash in the data file
  Example: `"thumbnail": "projects/hophacks_2024_youngheroes_thumbnail.png"`

- Components handle adding the leading slash as required by Next.js Image component
  Example: Component converts `"projects/image.png"` to `"/projects/image.png"`

## Utility Functions

The `src/utils/dataUtils.ts` file provides helper functions to access and manipulate data:

- `getAllHackathons()` - Get all hackathon data
- `getHackathonById(id)` - Get a specific hackathon by ID
- `getHackathonsByDate()` - Get hackathons sorted by date
- `getAllProjects()` - Get all projects from all hackathons
- `getFeaturedProjects(limit?)` - Get featured projects
- `getProjectById(id)` - Get a specific project by ID
- `getAllTeamMembers()` - Get all team members
- `getTeamMemberById(id)` - Get a specific team member by ID
- `getAllPhotos()` - Get all photos from all hackathons
- `getFeaturedPhotos(limit?)` - Get featured photos
- `getStatistics()` - Get overall statistics 
 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VietRocHack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
