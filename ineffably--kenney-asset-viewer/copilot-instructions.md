## kenney-asset-viewer

> I will give the files in ./cursor/rules/**/* as much importance as this file and other ".claude" files

I will give the files in ./cursor/rules/**/* as much importance as this file and other ".claude" files
I will Replace Cursor with Claude in these rules given they were meant for Cursor, but, I am also able to use them.
I will review the headers with the ruletype in each file to understand how to treat that context

# Claude Development Notes

## Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production  
- `npm run build:release` - Build release version with embedded data
- `npm run create-release` - Create complete release bundle with assets
- `npm run type-check` - Check TypeScript types
- `npm test` - Run test suite
- `npm run lint` - Run ESLint

## Known Issues
- Jest tests are currently failing due to Ant Design Input.Search destructuring in test environment
- CSS imports in Jest need proper module mapping configuration
- Need to investigate Jest/Ant Design compatibility for the StickyHeader component

## Recent Changes
- Implemented unified sticky header with search/filter controls
- Added lazy loading with AnimatedImage component using intersection observer
- Moved inline styles to dedicated CSS files for better maintainability
- Enhanced visual feedback with scroll-triggered shadows and borders

## Testing Protocol
Always run these commands before committing:
1. `npm run type-check`
2. `npm run build` 
3. `npm test`

If any of these fail, fix the issues before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ineffably)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ineffably)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
