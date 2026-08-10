## 04-sanity-integration

> This document outlines how Sanity CMS is integrated into the Mkdirs template.

# Sanity CMS Integration

This document outlines how Sanity CMS is integrated into the Mkdirs template.

## Configuration Files

- **sanity.config.ts**: Main Sanity configuration file
- **sanity.cli.ts**: CLI configuration for Sanity
- **sanity-typegen.json**: Configuration for type generation
- **schema.json**: Generated Sanity schemas in JSON format
- **sanity.types.ts**: Generated TypeScript types for Sanity schemas

## Schema Structure

The Sanity schema is organized in the `src/sanity` directory:

- **src/sanity/schemas**: Contains all content type definitions
- **src/sanity/lib**: Utility functions for working with Sanity
- **src/sanity/plugins**: Additional plugins for Sanity Studio
- **src/sanity/components**: Custom components in Sanity Studio

## Content Types

Key content types defined in the schema include:

- **Item**: Directory listing entries
- **Category**: Listing categories
- **Tag**: Listing tags
- **Collection**: Listing collections
- **Group**: Listing category groups
- **Blog Posts**: Blog content
- **Authors**: Content creators
- **Pages**: Static page content
- **Settings**: Global site settings

## Data Fetching

Sanity data is fetched using GROQ queries through the following methods:

1. **Client API**: Using the Sanity client to fetch data
2. **Next.js Data Fetching**: Server components that directly fetch Sanity data
3. **API Routes**: API endpoints that serve Sanity data

## Preview Functionality

The template includes a preview mode for content editing:

- **Draft Preview**: See draft content before publishing

## Studio Customization

The Sanity Studio interface is customized for the Mkdirs use case:

- **Custom Desk Structure**: Organized content editing interface
- **Custom Input Components**: Specialized input fields for directories
- **Sanity Plugins**: Additional functionality through Sanity plugins

## Type Generation

Types are automatically generated from Sanity schemas:

- Run `pnpm typegen` to update TypeScript types after schema changes
- Generated types ensure type safety when working with Sanity content

---
> Source: [MkThingsHQ/mkdirs](https://github.com/MkThingsHQ/mkdirs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
