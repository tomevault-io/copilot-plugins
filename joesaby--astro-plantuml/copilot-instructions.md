## astro-plantuml

> This is an Astro integration that automatically converts PlantUML code blocks in markdown files to diagrams using the PlantUML server.

# Astro PlantUML Integration Project

## Overview
This is an Astro integration that automatically converts PlantUML code blocks in markdown files to diagrams using the PlantUML server.

## Project Structure
```
astro-plantuml/
├── src/               # Source TypeScript files
│   ├── index.ts      # Main integration entry point
│   ├── remark-plugin.ts  # Remark plugin implementation
│   ├── types.ts      # TypeScript type definitions
│   └── utils.ts      # Utility functions (base64 encoding)
├── dist/             # Compiled JavaScript files (generated)
└── docs/             # Documentation site (Starlight)
```

## Key Features
- Automatic PlantUML diagram rendering
- Configurable PlantUML server URL
- Error handling with fallback
- CSS class injection for styling
- TypeScript support

## Development Commands
```bash
# Build the integration
npm run build

# Watch mode for development
npm run dev
```

## Testing & Validation
Before committing changes:
1. Run `npm run build` to ensure TypeScript compilation succeeds
2. Test the integration in your Astro project
3. Verify all PlantUML diagram types render correctly

## Integration Architecture
The integration uses Astro's hook system to inject a remark plugin that:
1. Finds code blocks with language "plantuml" in the markdown AST
2. Extracts the PlantUML content before syntax highlighting
3. Encodes it using PlantUML's custom base64 encoding with raw deflate
4. Fetches the diagram from the PlantUML server
5. Replaces the code block with an HTML image element

## Publishing Checklist
- [ ] Update version in package.json
- [ ] Ensure "withastro" keyword is present
- [ ] Run `npm run build`
- [ ] Test the integration
- [ ] Update README with any new features
- [ ] Run `npm publish`

## Common Issues
1. **Build errors**: Ensure all dependencies are installed
2. **Diagram rendering fails**: Check PlantUML server URL and network connectivity
3. **TypeScript errors**: Run `npm run build` to catch type issues early

## Recent Changes (v0.1.2)
- Switched from rehype to remark plugin to process PlantUML before syntax highlighting
- Fixed encoding by using `deflateRawSync` instead of `deflateSync`
- The remark plugin runs before Shiki to prevent language warnings

## Important Notes
- The package is published to npm as `astro-plantuml` version 0.1.2
- Documentation site available in the /docs directory

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joesaby)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joesaby)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
