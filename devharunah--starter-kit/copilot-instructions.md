## starter-kit

> Guidelines for extending the starter kit with new MVP blocks and components


# Extensibility and MVP Blocks Integration

## Starter Kit Philosophy

This is a **starter kit designed for rapid MVP development and extension**. Always consider how new features can be added and how existing components can be enhanced.

## MVP Blocks Integration

### Official Documentation
- **Main Site**: [https://blocks.mvp-subha.me/](https://blocks.mvp-subha.me/)
- **Component Library**: [https://blocks.mvp-subha.me/docs/](https://blocks.mvp-subha.me/docs/)
- **Installation Guide**: [https://blocks.mvp-subha.me/docs/guide/installation](https://blocks.mvp-subha.me/docs/guide/installation)
- **Adding Components**: [https://blocks.mvp-subha.me/docs/guide/add-a-block](https://blocks.mvp-subha.me/docs/guide/add-a-block)

### CLI Installation
```bash
# Install MVP Blocks CLI
npm install -g mvpblocks

# Add new components
npx mvpblocks add [component-name]

# Examples:
npx mvpblocks add flip-card
npx mvpblocks add pricing-card
npx mvpblocks add testimonial
npx mvpblocks add pricing
npx mvpblocks add team
npx mvpblocks add cta
```

### Available Component Categories

#### Foundation Components
- **Colors & Theming**: [https://blocks.mvp-subha.me/docs/foundation/colors](https://blocks.mvp-subha.me/docs/foundation/colors)
- **Installation**: [https://blocks.mvp-subha.me/docs/guide/installation](https://blocks.mvp-subha.me/docs/guide/installation)

#### Basic Components
- **Buttons**: [https://blocks.mvp-subha.me/docs/components/basic/buttons](https://blocks.mvp-subha.me/docs/components/basic/buttons)
- **Loaders**: [https://blocks.mvp-subha.me/docs/components/basic/loaders](https://blocks.mvp-subha.me/docs/components/basic/loaders)
- **Modals**: [https://blocks.mvp-subha.me/docs/components/basic/modals](https://blocks.mvp-subha.me/docs/components/basic/modals)
- **Pagination**: [https://blocks.mvp-subha.me/docs/components/basic/pagination](https://blocks.mvp-subha.me/docs/components/basic/pagination)
- **Progress**: [https://blocks.mvp-subha.me/docs/components/basic/progress](https://blocks.mvp-subha.me/docs/components/basic/progress)

#### Required Components
- **Footers**: [https://blocks.mvp-subha.me/docs/components/required/footers](https://blocks.mvp-subha.me/docs/components/required/footers)
- **Headers**: [https://blocks.mvp-subha.me/docs/components/required/headers](https://blocks.mvp-subha.me/docs/components/required/headers)
- **Logo Cloud**: [https://blocks.mvp-subha.me/docs/components/required/logo-cloud](https://blocks.mvp-subha.me/docs/components/required/logo-cloud)

#### Main Sections
- **About**: [https://blocks.mvp-subha.me/docs/mainsections/about](https://blocks.mvp-subha.me/docs/mainsections/about)
- **Contact**: [https://blocks.mvp-subha.me/docs/mainsections/contact](https://blocks.mvp-subha.me/docs/mainsections/contact)
- **CTA**: [https://blocks.mvp-subha.me/docs/mainsections/cta](https://blocks.mvp-subha.me/docs/mainsections/cta)
- **FAQs**: [https://blocks.mvp-subha.me/docs/mainsections/faqs](https://blocks.mvp-subha.me/docs/mainsections/faqs)
- **Features**: [https://blocks.mvp-subha.me/docs/mainsections/features](https://blocks.mvp-subha.me/docs/mainsections/features)
- **Hero**: [https://blocks.mvp-subha.me/docs/mainsections/hero](https://blocks.mvp-subha.me/docs/mainsections/hero)
- **Pricing**: [https://blocks.mvp-subha.me/docs/mainsections/pricing](https://blocks.mvp-subha.me/docs/mainsections/pricing)
- **Team**: [https://blocks.mvp-subha.me/docs/mainsections/team](https://blocks.mvp-subha.me/docs/mainsections/team)
- **Testimonials**: [https://blocks.mvp-subha.me/docs/mainsections/testimonials](https://blocks.mvp-subha.me/docs/mainsections/testimonials)

#### Forms
- **Authentication**: [https://blocks.mvp-subha.me/docs/forms/authentication](https://blocks.mvp-subha.me/docs/forms/authentication)
- **Multi Step Forms**: [https://blocks.mvp-subha.me/docs/forms/multi-step-forms](https://blocks.mvp-subha.me/docs/forms/multi-step-forms)

#### Cards
- **Basic Cards**: [https://blocks.mvp-subha.me/docs/cards/basic-cards](https://blocks.mvp-subha.me/docs/cards/basic-cards)
- **Flip Card**: [https://blocks.mvp-subha.me/docs/cards/card-flip](https://blocks.mvp-subha.me/docs/cards/card-flip)
- **Codes**: [https://blocks.mvp-subha.me/docs/cards/codes](https://blocks.mvp-subha.me/docs/cards/codes)
- **Glow Cards**: [https://blocks.mvp-subha.me/docs/cards/glow-cards](https://blocks.mvp-subha.me/docs/cards/glow-cards)
- **Product Cards**: [https://blocks.mvp-subha.me/docs/cards/product-cards](https://blocks.mvp-subha.me/docs/cards/product-cards)
- **Spotlight Cards**: [https://blocks.mvp-subha.me/docs/cards/spotlight-cards](https://blocks.mvp-subha.me/docs/cards/spotlight-cards)
- **X (Twitter) Cards**: [https://blocks.mvp-subha.me/docs/cards/x-twitter-cards](https://blocks.mvp-subha.me/docs/cards/x-twitter-cards)

#### Interactive Components
- **Chatbot UI**: [https://blocks.mvp-subha.me/docs/chatbot/chatbot-ui](https://blocks.mvp-subha.me/docs/chatbot/chatbot-ui)
- **Conversation**: [https://blocks.mvp-subha.me/docs/chatbot/conversation](https://blocks.mvp-subha.me/docs/chatbot/conversation)

#### Dashboards
- **Admin Dashboard**: [https://blocks.mvp-subha.me/docs/dashboards/admin-dashboard](https://blocks.mvp-subha.me/docs/dashboards/admin-dashboard)
- **User Dashboard**: [https://blocks.mvp-subha.me/docs/dashboards/user-dashboard](https://blocks.mvp-subha.me/docs/dashboards/user-dashboard)

#### Creative Components
- **Globe**: [https://blocks.mvp-subha.me/docs/creative/globe](https://blocks.mvp-subha.me/docs/creative/globe)
- **Interactive Tooltip**: [https://blocks.mvp-subha.me/docs/creative/interactive-tooltip](https://blocks.mvp-subha.me/docs/creative/interactive-tooltip)
- **Motion Number**: [https://blocks.mvp-subha.me/docs/creative/motion-number](https://blocks.mvp-subha.me/docs/creative/motion-number)
- **Scroll Animation**: [https://blocks.mvp-subha.me/docs/creative/scroll-animation](https://blocks.mvp-subha.me/docs/creative/scroll-animation)

#### Grids
- **Bento Grids**: [https://blocks.mvp-subha.me/docs/grids/bento-grids](https://blocks.mvp-subha.me/docs/grids/bento-grids)
- **Gallery**: [https://blocks.mvp-subha.me/docs/grids/gallery](https://blocks.mvp-subha.me/docs/grids/gallery)
- **Masonry Grids**: [https://blocks.mvp-subha.me/docs/grids/masonry-grids](https://blocks.mvp-subha.me/docs/grids/masonry-grids)

#### Text Animations
- **Circular Text**: [https://blocks.mvp-subha.me/docs/text-animations/circular-text](https://blocks.mvp-subha.me/docs/text-animations/circular-text)
- **Random Unveil**: [https://blocks.mvp-subha.me/docs/text-animations/random-unveil](https://blocks.mvp-subha.me/docs/text-animations/random-unveil)
- **Scroll Velocity**: [https://blocks.mvp-subha.me/docs/text-animations/scroll-velocity](https://blocks.mvp-subha.me/docs/text-animations/scroll-velocity)
- **Text Reveal**: [https://blocks.mvp-subha.me/docs/text-animations/text-reveal](https://blocks.mvp-subha.me/docs/text-animations/text-reveal)
- **Typewriter**: [https://blocks.mvp-subha.me/docs/text-animations/typewriter](https://blocks.mvp-subha.me/docs/text-animations/typewriter)

#### Backgrounds
- **Gradient Bars**: [https://blocks.mvp-subha.me/docs/backgrounds/gradient-bars](https://blocks.mvp-subha.me/docs/backgrounds/gradient-bars)

#### Skeletons
- **Skeletons**: [https://blocks.mvp-subha.me/docs/skeletons/skeletons](https://blocks.mvp-subha.me/docs/skeletons/skeletons)

## Theme Customization

### Color System
- **Customization Guide**: [https://blocks.mvp-subha.me/docs/foundation/colors](https://blocks.mvp-subha.me/docs/foundation/colors)
- **CSS Variables**: Update [globals.css](mdc:src/app/globals.css) for consistent theming
- **Tailwind Integration**: Use CSS variables with Tailwind classes

### Adding New Themes
1. **Define CSS Variables**: Add new color schemes to [globals.css](mdc:src/app/globals.css)
2. **Create Theme Variants**: Implement theme switching logic
3. **Update Components**: Ensure all components support theme switching
4. **Documentation**: Update component documentation with theme options

## Extension Patterns

### Adding New Pages
1. **Create Route**: Add new directory in `src/app/`
2. **Add Navigation**: Update [navigation.tsx](mdc:src/Components/layouts/navigation.tsx)
3. **Follow Patterns**: Use existing page structure as template
4. **SEO**: Add proper metadata and descriptions

### Adding New Components
1. **Choose Location**: 
   - `src/Components/mvp-blocks/` for pre-built components
   - `src/Components/ui/` for reusable UI components
   - Feature-specific directories for custom components
2. **Follow Naming**: Use consistent naming conventions
3. **TypeScript**: Define proper interfaces and types
4. **Styling**: Follow the established design system

### Adding New API Routes
1. **Create Endpoint**: Add new route in `src/app/api/`
2. **Follow Patterns**: Use existing API structure
3. **Error Handling**: Implement proper error responses
4. **Authentication**: Add auth checks where needed

### Adding New Data Sources
1. **Mock Data**: Extend [data.json](mdc:public/data/data.json)
2. **Types**: Define TypeScript interfaces
3. **Utilities**: Create helper functions in `src/lib/`
4. **API Integration**: Connect to real APIs when ready

## Best Practices for Extension

### Component Development
- **Reusability**: Make components configurable and reusable
- **Props Interface**: Define clear TypeScript interfaces
- **Error Boundaries**: Implement proper error handling
- **Loading States**: Add loading and error states
- **Responsive Design**: Ensure mobile compatibility
- **Accessibility**: Follow WCAG guidelines

### Code Organization
- **Feature-based**: Group related functionality together
- **Separation of Concerns**: Keep UI, logic, and data separate
- **Documentation**: Add JSDoc comments for complex functions
- **Testing**: Write tests for critical functionality

### Performance Considerations
- **Code Splitting**: Use dynamic imports for heavy components
- **Image Optimization**: Use Next.js Image component
- **Bundle Size**: Monitor and optimize bundle size
- **Caching**: Implement proper caching strategies

### Maintenance
- **Version Control**: Use semantic versioning
- **Changelog**: Keep track of changes and updates
- **Documentation**: Update documentation with new features
- **Dependencies**: Keep dependencies up to date

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devharunah) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
