## drive-cost

> This document provides guidance for AI agents (like Claude, GPT-4, etc.) working on the Fuel Cost Calculator project.

# AI Agent Development Guide

This document provides guidance for AI agents (like Claude, GPT-4, etc.) working on the Fuel Cost Calculator project.

## Project Overview

This is a Vue 3 single-page application that calculates and compares journey costs between traditional fuel vehicles (Diesel/Petrol) and Electric vehicles. The project uses Vite as the build tool and Tailwind CSS for styling.

## Technology Stack

- **Framework**: Vue 3 with Composition API
- **Build Tool**: Vite
- **Styling**: Tailwind CSS
- **CSS Processing**: PostCSS
- **Package Manager**: npm

## Project Structure

```
drive-cost/
├── src/
│   ├── components/     # Vue components
│   ├── App.vue        # Main application component
│   └── main.js        # Application entry point
├── public/            # Static assets
├── dist/              # Production build output
├── index.html         # HTML entry point
├── package.json       # Dependencies and scripts
├── vite.config.js     # Vite configuration
├── postcss.config.js  # PostCSS configuration
└── Makefile           # Build automation
```

## Development Guidelines

### Code Style

- Use Vue 3 Composition API (`<script setup>`) for components
- Follow Vue 3 best practices for reactivity and component composition
- Use Tailwind CSS utility classes for styling
- Keep components modular and reusable
- Use meaningful variable and function names

### Testing Changes

Before committing changes:

1. Run the development server: `npm run dev`
2. Test all calculator functionality:
   - Diesel/Petrol calculations
   - Electric vehicle calculations
   - Unit conversions (miles/km, UK/US gallons)
   - Real-time updates as values change
3. Build for production: `npm run build`
4. Preview production build: `npm run preview`

### Common Tasks

#### Adding a New Component

1. Create component file in `src/components/`
2. Use `<script setup>` syntax
3. Import and register in parent component or `App.vue`
4. Apply Tailwind CSS classes for styling

#### Modifying Calculations

- Calculation logic should be in computed properties or reactive functions
- Ensure conversions are accurate:
  - UK gallon = 4.54609 litres
  - US gallon = 3.78541 litres
  - Miles to km conversion factor
  - kWh per 100km conversions
- Per-distance tax calculations:
  - Both diesel/petrol and electric vehicles support optional per-distance tax
  - Tax is calculated as: `distance * taxPerDistance`
  - Tax defaults to 0 when not specified
  - Tax is added to the base fuel/electricity cost
  - Tax amount adjusts based on the distance unit (miles or km)

#### Styling Updates

- Use Tailwind utility classes where possible
- Check responsive design (mobile, tablet, desktop)
- Maintain consistent spacing and color scheme

### Build and Deployment

- **Development**: `npm run dev` - Hot reload enabled
- **Production Build**: `npm run build` - Creates optimized bundle in `dist/`
- **Preview**: `npm run preview` - Test production build locally
- **Makefile**: Check `Makefile` for additional build commands

## Key Features to Preserve

When making changes, ensure these core features remain functional:

1. **Dual Calculator Interface**: Side-by-side comparison of fuel types
2. **Real-time Updates**: Calculations update as user types
3. **Flexible Units**: Support for both imperial and metric units
4. **Accurate Conversions**: Proper handling of UK vs US gallons
5. **Clear Comparison**: Savings calculation and display
6. **Responsive Design**: Works on all screen sizes
7. **Per-Distance Tax**: Optional tax per mile/km for both fuel types (defaults to 0)

## Potential Improvements

When suggesting or implementing enhancements, consider:

- Adding unit tests (Vitest)
- Implementing input validation and error handling
- Adding preset values or saved calculations
- Supporting additional vehicle types (Hybrid, Hydrogen)
- Adding fuel efficiency comparisons
- Implementing charts/graphs for visual comparison
- Adding currency conversion options
- Dark mode support
- Accessibility improvements (ARIA labels, keyboard navigation)

## Dependencies

Key dependencies in `package.json`:

- `vue`: Core framework
- `vite`: Build tool and dev server
- `tailwindcss`: Styling framework
- `postcss`: CSS processing
- `autoprefixer`: CSS vendor prefixes

## Common Issues and Solutions

### Issue: Calculations not updating

- Check reactive references (`ref`, `computed`)
- Ensure v-model bindings are correct
- Verify input types (number vs string)

### Issue: Styling not applied

- Check Tailwind configuration
- Ensure PostCSS is processing correctly
- Verify class names are correct

### Issue: Build fails

- Clear `node_modules` and reinstall: `rm -rf node_modules && npm install`
- Check for syntax errors in Vue components
- Verify all imports are correct

## Version Control

- Main branch: `main`
- Current tracking shows modified files in `src/` and `node_modules/`
- Always test changes before committing
- **IMPORTANT: Use conventional commit messages** (see details below)

### Conventional Commits

**All commits MUST follow the Conventional Commits specification.**

This project uses [Cocogitto](https://github.com/cocogitto/cocogitto) for managing conventional commits and changelog generation (configured in `cog.toml`).

#### Commit Message Format

Each commit message must follow this format:

```
<type>: <description>

[optional body]

[optional footer(s)]
```

#### Commit Types

Use one of these commit types:

- **feat**: A new feature (triggers minor version bump)
- **fix**: A bug fix (triggers patch version bump)
- **docs**: Documentation only changes
- **style**: Changes that don't affect the meaning of the code (white-space, formatting, etc.)
- **refactor**: A code change that neither fixes a bug nor adds a feature
- **perf**: A code change that improves performance
- **test**: Adding missing tests or correcting existing tests
- **build**: Changes that affect the build system or external dependencies
- **ci**: Changes to CI configuration files and scripts
- **chore**: Other changes that don't modify src or test files

#### Examples

Good commit messages:
```
feat: add currency conversion option to calculator
fix: correct UK gallon to liter conversion factor
docs: update README with installation instructions
refactor: simplify fuel cost calculation logic
test: add unit tests for electric vehicle calculations
chore: update dependencies to latest versions
```

Bad commit messages (DO NOT use these):
```
✗ Updated files
✗ Fix bug
✗ WIP
✗ changes
✗ misc updates
```

#### Breaking Changes

For breaking changes, add `!` after the type or add `BREAKING CHANGE:` in the footer:

```
feat!: change API response format

BREAKING CHANGE: The API now returns costs in cents instead of dollars
```

#### Scope (Optional)

You can add a scope to provide additional context:

```
feat(calculator): add hybrid vehicle support
fix(ui): correct responsive layout on mobile
docs(readme): add deployment instructions
```

**Remember: Following conventional commits is mandatory for all changes to this repository.**

## Contact and Resources

- Repository: [unicornops/drive-cost](https://github.com/unicornops/drive-cost)
- Vue 3 Docs: https://vuejs.org/
- Vite Docs: https://vitejs.dev/
- Tailwind CSS Docs: https://tailwindcss.com/

## Agent-Specific Notes

When working on this project:

1. **Always read existing files** before making changes
2. **Test calculations carefully** - accuracy is critical
3. **Maintain responsive design** - check mobile views
4. **Preserve existing functionality** - don't break working features
5. **Follow Vue 3 patterns** - use Composition API consistently
6. **Keep it simple** - this is a focused calculator app
7. **Document complex logic** - especially conversion formulas

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/unicornops)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/unicornops)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
