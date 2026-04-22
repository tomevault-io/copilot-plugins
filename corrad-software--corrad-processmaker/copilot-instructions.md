## corrad-processmaker

> When working with components:

# Development Guidelines

## Component Development

When working with components:
- Use existing Rose UI components (prefixed with `rs-`) instead of creating new ones
- Customize components through props rather than creating variants
- Follow the established naming conventions and component structure
- Place new Rose UI components in the [components](mdc:components) directory
- Place new FormKit components in the [components/formkit](mdc:components/formkit) directory

## Form Development

When creating forms:
- Use FormKit for all form inputs and validation
- Refer to [formkit.config.js](mdc:formkit.config.js) for available plugins and configuration
- For complex forms, use the Form Builder feature
- Implement custom FormKit inputs when needed

## State Management

When managing state:
- Use Pinia stores for global state
- Store files should be placed in the [stores](mdc:stores) directory
- Follow the existing pattern in [stores/formBuilder.js](mdc:stores/formBuilder.js)
- Use Vue's Composition API for component-local state

## API Development

When creating new API endpoints:
- Place new API routes in the [server](mdc:server) directory
- Use Prisma for database operations
- Implement proper validation and error handling
- Follow RESTful API design principles

## Styling Guidelines

When styling components:
- Use Tailwind CSS utility classes
- Custom styling should be placed in the [assets](mdc:assets) directory
- Follow the existing color scheme and design patterns
- Ensure responsive design for all components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/corrad-software) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
