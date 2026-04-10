## xkids

> This file provides guidance for agentic coding agents working with the XKids Learning Center codebase.

# AGENTS.md

This file provides guidance for agentic coding agents working with the XKids Learning Center codebase.

## Development Commands

### Docker Development
- **Start development environment**: `./docker/scripts/dev-start.sh`
  - Enables file watching with live reload
  - Starts the app at http://localhost:8080
  - Log monitoring available at http://localhost:8081
- **Start with docker-compose**: `docker-compose --profile development up -d`
- **Stop development**: `docker-compose --profile development down`

### Production Deployment
- **Build production image**: `docker build -t xkids-learning-center .`
- **Run production**: `docker run -d -p 8080:80 --name xkids-app xkids-learning-center`
- **Start with docker-compose**: `docker-compose up -d`
- **Multi-platform build**: `docker buildx build --platform linux/amd64,linux/arm64 -t xkids-learning-center .`

### Container Management
- **View logs**: `docker-compose logs` or `docker logs xkids-app`
- **Stop container**: `docker stop xkids-app`
- **Remove container**: `docker rm xkids-app`
- **Health check**: Application serves `/health` endpoint for container orchestration

### Testing Commands
- **No automated testing framework** - This is a static web application
- **Manual testing**: Access http://localhost:8080 and test all interactive elements
- **Cross-browser testing**: Test in Chrome, Firefox, Safari, and Edge
- **Responsive testing**: Test on mobile, tablet, desktop, and TV viewport sizes
- **Keyboard navigation testing**: Verify arrow keys and Enter/Space work on all interactive elements

## Code Style Guidelines

### Project Architecture
- **Static Web Application**: Pure HTML/CSS/JavaScript without build tools or frameworks
- **Modular CSS Architecture**: Separate files for variables, base styles, components, and responsive design
- **Component-Based JavaScript**: Separate JS files for navigation, audio, and animations
- **Inline CSS for Lesson Pages**: Each lesson page (malay*.html, iqra*.html) includes inline styles for modularity

### HTML Structure
- **DOCTYPE**: Always use `<!DOCTYPE html>`
- **Language**: Set `lang="en"` on html element
- **Meta Tags**: Include charset UTF-8 and viewport meta tag
- **Semantic HTML5**: Use appropriate semantic elements (header, main, section, nav, footer)
- **Bootstrap Integration**: Use Bootstrap 5.3.0 CSS via CDN for basic grid and components
- **Accessibility**: Include proper ARIA labels, alt text, and keyboard navigation support

### CSS Conventions
- **CSS Variables**: Centralize design tokens in `:root` using CSS custom properties
- **Naming Convention**: Use kebab-case for class names (`.level-button`, `.module-card`)
- **BEM Methodology**: Follow Block-Element-Modifier pattern for component styles
- **Responsive Design**: Use mobile-first approach with CSS media queries
- **Gradient System**: Use predefined gradient variables for consistent theming
- **Transitions**: Use CSS custom properties for transition timing and easing functions

### JavaScript Standards
- **Vanilla JS**: No frameworks or libraries except Bootstrap CSS
- **ES6+ Features**: Use modern JavaScript features (const/let, arrow functions, template literals)
- **Function Naming**: Use descriptive camelCase function names (`initializeKeyboardNavigation`)
- **Error Handling**: Use try-catch blocks for Web Audio API and other browser-dependent features
- **Event Handling**: Use `addEventListener` with proper event delegation
- **Global Functions**: Export functions to global scope for inline script access

### File Organization
```
src/
├── index.html              # Main landing page
├── assets/
│   ├── css/               # Stylesheets (variables, base, components, responsive)
│   └── js/                # JavaScript modules (navigation, audio, animations)
└── pages/
    ├── malay/             # Malaysian reading lessons (malay1.html - malay6.html)
    └── iqra/              # Arabic reading lessons (iqra1.html - iqra4.html)
```

### Import/Dependency Management
- **External Dependencies**: Only Bootstrap 5.3.0 CSS and Google Fonts (Poppins) via CDN
- **Internal Dependencies**: Load CSS files in order (variables → base → components → responsive)
- **JavaScript Loading**: Load JS files at end of body, initialize functions in inline scripts
- **No Package Manager**: This project doesn't use npm, yarn, or any package managers

### Code Formatting
- **Indentation**: Use 4 spaces for HTML, CSS, and JavaScript
- **Line Length**: Keep lines under 100 characters when possible
- **HTML Attributes**: Place each attribute on new line for complex elements
- **CSS Properties**: Group related properties, use shorthand where appropriate
- **JavaScript Functions**: Use consistent spacing around operators and braces

### Naming Conventions
- **Files**: kebab-case (malay1.html, navigation.js)
- **CSS Classes**: kebab-case with BEM methodology (`.level-button`, `.module-card__header`)
- **JavaScript Functions**: camelCase with descriptive names (`initializeAudioEffects`)
- **CSS Variables**: kebab-case with semantic grouping (`--primary-color`, `--transition-smooth`)
- **HTML IDs**: Use sparingly, prefer classes; when used, use kebab-case

### Error Handling
- **Web Audio API**: Wrap in try-catch blocks, fail silently
- **Browser Compatibility**: Check for feature support before use
- **Graceful Degradation**: Ensure core functionality works without JavaScript
- **User Feedback**: Provide visual and audio feedback for all interactions

### Performance Guidelines
- **Asset Optimization**: Minimize external dependencies, use CDN for libraries
- **Image Optimization**: Use appropriate image formats and sizes
- **CSS Efficiency**: Avoid complex selectors, leverage CSS variables
- **JavaScript Efficiency**: Use event delegation, minimize DOM queries
- **Caching**: Leverage browser caching for static assets

### Security Considerations
- **CSP Headers**: Configured in nginx for content security
- **No Inline Scripts**: Avoid inline JavaScript except for initialization
- **XSS Prevention**: Sanitize user inputs, use textContent instead of innerHTML
- **HTTPS Ready**: Application works over HTTPS in production

### Browser Support
- **Modern Browsers**: Chrome, Firefox, Safari, Edge (latest versions)
- **Mobile Browsers**: iOS Safari, Chrome Mobile
- **Smart TV**: Optimized for TV remote navigation
- **Fallback**: Basic functionality works without JavaScript

### Interactive Elements
- **Click Events**: Use `addEventListener('click', handler)` for buttons and interactive elements
- **Keyboard Navigation**: Implement arrow key navigation for level selection
- **Touch Support**: Ensure touch events work on mobile devices
- **Audio Feedback**: Use Web Audio API for click sounds and success/error feedback

### Color and Theming
- **Color System**: Use CSS custom properties for all colors
- **Gradients**: Predefined gradient variables for consistent theming
- **Accessibility**: Ensure sufficient color contrast (WCAG AA compliance)
- **Theme Variants**: Support different themes (primary, secondary, islamic, success, warning, error)

### Animation Guidelines
- **CSS Animations**: Use CSS custom properties for timing and easing
- **JavaScript Animations**: Use requestAnimationFrame for smooth animations
- **Performance**: Prefer CSS transforms and opacity for better performance
- **User Preference**: Respect `prefers-reduced-motion` media query

### Development Workflow
- **Local Development**: Use Docker development environment with live reload
- **File Watching**: Changes to `src/` directory are reflected immediately
- **Testing**: Manual testing in browser, no automated test framework
- **Deployment**: Docker-based deployment to production environments

### Code Review Checklist
- [ ] HTML is semantic and accessible
- [ ] CSS follows BEM methodology and uses variables
- [ ] JavaScript is modular and error-handled
- [ ] Interactive elements work with keyboard and touch
- [ ] Responsive design works across viewports
- [ ] Audio feedback is implemented and error-handled
- [ ] Code follows naming conventions and formatting standards
- [ ] No console.log statements in production code
- [ ] External dependencies are minimal and necessary

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/exrienz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/exrienz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
