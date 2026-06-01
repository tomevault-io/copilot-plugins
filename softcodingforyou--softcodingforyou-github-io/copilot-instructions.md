## softcodingforyou-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is David Baum's professional portfolio website hosted on GitHub Pages at `https://softcodingforyou.github.io`. It's a static website showcasing his work as a Research Engineer and Software Developer under the pseudonym "SoftcodingForYou".

## Technology Stack

### Frontend Architecture
- **Pure HTML/CSS/JavaScript** - No frameworks or build process required
- **Static Site** - Served directly from GitHub Pages
- **Responsive Design** - Mobile-first approach with CSS Grid and Flexbox
- **Green Color Scheme** - Matches SoftcodingForYou branding

### File Structure
```
├── index.html              # Main HTML file with semantic structure
├── styles.css              # CSS with custom properties and responsive design
├── script.js               # Vanilla JavaScript for interactivity
├── assets/images/          # Image assets (profile picture and logo)
└── README.md              # Comprehensive project documentation
```

## Development Commands

### Local Development
- **Serve locally**: Simply open `index.html` in a web browser
- **No build process**: All files are ready for production
- **Live reload**: Use a simple HTTP server like `python -m http.server 8000`

### Testing
- **Browser testing**: Test in Chrome, Firefox, Safari, Edge
- **Mobile testing**: Use browser dev tools or real devices
- **Accessibility**: Check with screen readers and keyboard navigation

## Code Architecture & Patterns

### CSS Organization
- **CSS Custom Properties** for consistent theming (green color palette)
- **BEM-like naming** for maintainable class names
- **Mobile-first responsive** breakpoints at 768px and 480px
- **Flexbox/Grid layout** for modern positioning

### JavaScript Features
- **Smooth scrolling** navigation
- **Mobile menu toggle** with hamburger animation
- **Skill bar animations** triggered by intersection observer
- **Throttled scroll events** for performance
- **No external dependencies** - pure vanilla JS

### HTML Structure
- **Semantic markup** with proper heading hierarchy
- **Accessibility features** with ARIA labels and proper focus management
- **SEO optimized** with meta tags and structured content
- **Social media links** with proper external link attributes

## Content Management

### Profile Information
- **Personal details**: Research Engineer at InteraXon Inc. (Muse), PhD in Neurophysiology
- **Professional roles**: Former Co-founder & CTO of Helment, Freelance Developer
- **Skills**: Python (Full Stack), Flutter/Dart, MATLAB, Unity, REST API
- **Languages**: German, French, English, Spanish

### Featured Projects
1. **Securicata** - DNS-level content blocker for families
   - Links: https://securicata.app/ and https://mi.securicata.app/
2. **NeuriGUI** - Electrophysiological signal processing app
3. **GitHub projects** - Various neuroscience and mobile apps

### Additional Work
- **Acupuntura Tung** - Healthcare tech consulting
  - Google Play: https://play.google.com/store/apps/details?id=com.acupuntura.tung
  - Instagram: @acupuntura_hoy
- **Club de Astronomía Ciudad del Este** - Community astronomy club
  - Website: https://astronomiaciudaddeleste.godaddysites.com/
  - Instagram: @astronomiaciudaddeleste

## Image Assets

### Asset Management
- **Profile picture**: `assets/images/DM_BAUM_Profile_Picture.png`
- **Logo**: `assets/images/SoftcodingForYou_round.png`
- **Optimization**: Images are web-optimized PNG files
- **Responsive**: CSS handles different screen sizes

### Image Usage Guidelines
- Profile picture uses `object-fit: cover` for circular cropping
- Logo uses `object-fit: contain` to maintain aspect ratio
- Both images are same container size (180px) for visual balance

## Styling Guidelines

### Color Palette
```css
--primary-color: #059669;    /* Emerald green */
--secondary-color: #047857;  /* Darker emerald */
--accent-color: #10b981;     /* Lighter green */
--surface: #f0fdf4;          /* Very light green */
--border: #d1fae5;           /* Light green borders */
```

### Typography
- **Font family**: Inter (Google Fonts)
- **Hierarchy**: Clear heading structure (h1-h4)
- **Responsive**: Font sizes scale appropriately on mobile

### Interactive Elements
- **Buttons**: Green theme with hover states
- **Skill bars**: Animated progress bars with green gradients
- **Cards**: Subtle shadows with hover lift effects

## GitHub Pages Configuration

### Deployment
- **Automatic deployment** from main branch
- **Custom domain**: Can be configured via CNAME
- **HTTPS**: Automatically provided by GitHub Pages
- **No Jekyll**: Uses `.nojekyll` file to disable Jekyll processing

### Performance Considerations
- **Static assets**: All files are optimized for web delivery
- **Minimal HTTP requests**: Few external dependencies
- **Fast loading**: Lightweight CSS and JavaScript

## Maintenance Notes

### Content Updates
- **Projects**: Add new projects to the Featured Projects section
- **Skills**: Update skill levels in the Programming section
- **Experience**: Add new professional experience to Additional Work section
- **Links**: Keep social media and external links current

### Technical Updates
- **Browser compatibility**: Test new CSS features for support
- **Accessibility**: Regular testing with screen readers
- **Performance**: Monitor loading times and optimize as needed
- **Security**: Keep external CDN links (Font Awesome, Google Fonts) updated

## Contact Information
- **LinkedIn**: https://www.linkedin.com/in/david-marcel-baum/
- **GitHub**: https://github.com/softcodingforyou
- **Professional focus**: Neuroscience, Brain-Computer Interfaces, Mobile Development

---
> Source: [SoftcodingForYou/softcodingforyou.github.io](https://github.com/SoftcodingForYou/softcodingforyou.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
