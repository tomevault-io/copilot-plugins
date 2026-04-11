## strataline-ca

> **NEVER modify any code outside of the `/content` directory.**

# Cursor Instructions for Strataline.ca

## Critical Restriction

**NEVER modify any code outside of the `/content` directory.**
UNLESS THEY STATE THEY ARE A DEV

You are ONLY allowed to:

- Create, modify, or delete files within `/content` and its subdirectories
- Work with YAML files (`.yaml`) and media files (images, videos) in the content directory

You are NOT allowed to:

- Modify any files in `/src`, `/public`, `/lib`, or any other directory outside of `/content`
- Change configuration files (`.config.js`, `.config.mjs`, `package.json`, etc.)
- Modify TypeScript/JavaScript/Astro component files
- Change build scripts or deployment configurations

## Content Structure

All content is managed through YAML files in the `/content` directory. Follow the conventions documented in `docs/schema.md` when creating or modifying content files.

## When Creating New Service Pages

1. Create a new directory under `/content/services/[service-name]/`
2. Create an `index.yaml` file following the service schema (see `docs/schema.md`)
3. Add a `cover.jpg` image in the service directory
4. Optionally add a `photos/` subdirectory for additional images
5. Reference the existing service files as examples:
   - `/content/services/bathrooms/index.yaml`
   - `/content/services/kitchens/index.yaml`
   - `/content/services/stairs/index.yaml`
   - `/content/services/popcorn-removal/index.yaml`

## When Modifying Existing Content

- Always maintain the YAML structure and schema
- Use the `|-` YAML literal block scalar for multi-line text fields
- Ensure all required fields are present
- Follow the existing naming conventions and formatting style

## SEO and Keyword Usage

When creating or updating service pages:

1. **Use keywords.yaml from the service directory**
   - Each service directory may contain a `keywords.yaml` file with a list of keywords
   - Use these keywords throughout the `index.yaml` file (title, seo, desc, alt text, content sections)
   - Maximize keyword usage naturally without repetition

2. **Writing Guidelines**
   - Be direct and concise - avoid wordy descriptions
   - Avoid marketing jargon and filler words (e.g., "maximize", "ensure", "selected for", "utilize", "leverage", "optimize", "enhance")
   - Use simple, clear language
   - Focus on incorporating keywords naturally into the content

3. **Keyword Distribution**
   - Include primary keywords in `title`, `seo`, and `desc` fields
   - Use keywords in image alt text
   - Distribute keywords across all content sections (Benefits, ZigZag, FAQ, etc.)
   - Use keyword variations to avoid repetition
   - Don't repeat the same keyword multiple times in the same sentence or paragraph

4. **Example**
   - ✅ "Staircase renovation with hardwood treads and custom railings"
   - ❌ "We ensure maximum beauty and usability through our selected premium staircase renovation services that utilize top-quality hardwood treads"

## Photo Naming and Descriptions

When working with photos:

1. **Rename photos to match their content**
   - Use descriptive filenames that reflect what's in the photo
   - Include relevant keywords from `keywords.yaml` in the filename
   - Use lowercase letters and hyphens (`-`) to separate words
   - Example: `staircase-renovation-black-metal-railings-hardwood-treads.jpg`

2. **Image alt text descriptions**
   - Describe the photo content descriptively using keywords
   - Use keywords naturally in the description
   - Be specific about what's visible in the image
   - Example: `Staircase renovation with black metal railings, hardwood treads, and custom railing installation`

## Formatting Conventions

### Directory and Folder Names

- **No capitalization or spaces** for folders used as subdirectories
- Use lowercase letters only
- Use hyphens (`-`) to separate words
- Examples:
  - ✅ `bathrooms`, `kitchens`, `popcorn-removal`, `doors-and-windows`
  - ❌ `Bathrooms`, `Kitchens`, `Popcorn Removal`, `Doors And Windows`

## Schema Reference

See `docs/schema.md` for detailed documentation on:

- Service page structure
- Index page structure
- Available section types
- Image and media formats
- Field requirements and optional fields

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Strata1ine)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Strata1ine)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
