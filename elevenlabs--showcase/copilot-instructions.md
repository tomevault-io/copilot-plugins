## showcase

> When asked to create a new project, follow these steps:

# Claude Instructions for Creating Projects

## Creating a New Project

When asked to create a new project, follow these steps:

### 1. Project MDX File Structure

Create a new MDX file in `/projects/(root)/` with this frontmatter structure:

```yaml
---
title: [Project Title]
description: [Brief description of the project]
authorIds:
  - [author-id-1] # Must match filename in /authors/ directory (without .yml)
  - [author-id-2] # Optional: multiple authors
categories:
  - agents # Pick from: agents, text-to-speech, speech-to-text, music, voices
  - music # Can have multiple categories
isFeatured: true/false # Whether to feature this project
date: "YYYY-MM-DD" # Project publication date
image: /images/[project-name].png # Project thumbnail/cover image
demoUrl: https://... # Optional: Live demo URL
repoUrl: https://... # Optional: GitHub repository URL
videoUrl: https://... # Optional: Demo video URL
xUrl: https://... # Optional: X/Twitter post URL
---
```

### 2. Author File Structure

If the author doesn't exist, create a new YAML file in `/authors/[author-id].yml`:

```yaml
name: [Full Name]
avatar: "https://..." # Avatar image URL
url: "https://..." # Personal website/portfolio
bio: "Brief bio" # One-line bio
location: "City, Country"
isElevenLabs: true/false # If they work at ElevenLabs
socials:
  - label: X
    url: "https://x.com/username"
  - label: GitHub
    url: "https://github.com/username"
  - label: LinkedIn
    url: "https://linkedin.com/in/username"
```

### 3. Available Categories

These are the valid category IDs to use in the `categories` field:

- `agents` - Agents projects
- `text-to-speech` - Text-to-speech applications
- `speech-to-text` - Speech recognition projects
- `music` - Music generation/audio projects
- `voices` - Voice cloning/synthesis projects

### 4. File Naming Conventions

- Project files: Use kebab-case for MDX files (e.g., `voice-assistant-app.mdx`)
- Author files: Use kebab-case for author IDs (e.g., `john-smith.yml`)
- The author ID in frontmatter must match the filename without extension

### 5. Example Creation Command

When a user says: "Create a new project for a voice assistant app by John Smith"

1. First check if author exists:

   - Look for `/authors/john-smith.yml`
   - If not, create it with the author template above

2. Create the project MDX file at `/projects/(root)/voice-assistant-app.mdx`:

```mdx
---
title: Voice Assistant App
description: An AI-powered voice assistant built with ElevenLabs API
authorIds:
  - john-smith
categories:
  - agents
  - text-to-speech
isFeatured: false
date: "2024-01-31"
image: /images/voice-assistant-app.png
demoUrl: https://voice-assistant.example.com
repoUrl: https://github.com/johnsmith/voice-assistant
---

# Voice Assistant App

## Overview

[Project description and details]

## Features

- Feature 1
- Feature 2

## Technologies Used

- ElevenLabs API
- [Other technologies]

## Getting Started

[Installation and usage instructions]
```

### 6. Validation Checks

Before creating a project, verify:

- Author ID matches an existing author file or create one
- Categories are from the valid list above
- Date is in YYYY-MM-DD format
- All URLs are properly formatted (start with https://)
- Image path follows the convention: `/images/[project-name].png`

### 7. Architecture Notes

- Projects are stored as MDX files in `/projects/` directory
- Fumadocs processes these automatically via `source.config.ts`
- The `getProjects()` function in `/apps/www/lib/projects.ts` fetches and enriches project data
- Author data is loaded from YAML files in `/authors/` directory
- Projects appear automatically on the showcase pages once created

## Quick Project Creation Template

Use this when asked to "create a project":

```bash
# 1. Check/create author (if needed)
/authors/[author-id].yml

# 2. Create project MDX
/projects/(root)/[project-name].mdx

# 3. Include all required frontmatter fields
# 4. Add project content in markdown
```

Remember: The system automatically picks up new projects - no need to update any indexes or lists!

---
> Source: [elevenlabs/showcase](https://github.com/elevenlabs/showcase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
