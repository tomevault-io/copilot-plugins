## sfggc-nextjs

> The project uses a custom Bootstrap build with SCSS modules:


# SCSS and Styling Conventions

## SCSS Architecture

The project uses a custom Bootstrap build with SCSS modules:

- [src/scss/sfggc-bootstrap.scss](mdc:src/scss/sfggc-bootstrap.scss) - Custom Bootstrap variables and overrides
- [src/scss/sfggc.scss](mdc:src/scss/sfggc.scss) - Main stylesheet with global styles
- Component-specific `.module.scss` files for scoped styles

## CSS Custom Properties

The project uses CSS custom properties for theming:

```scss
html {
  --sfggc-title-font-family: "DM Serif Display", "Times New Roman", var(--sfggc-font-sans-serif);
}

body {
  text-shadow: 1px 1px 1px var(--sfggc-body-text-shadow-color);
}
```

## Responsive Design Pattern

Mobile-first approach with Bootstrap breakpoints:

```scss
.component {
  // Mobile styles (default)
  font-size: 1rem;
  
  @media (min-width: 576px) {
    // Small devices
    font-size: 1.1rem;
  }
  
  @media (min-width: 768px) {
    // Medium devices
    font-size: 1.2rem;
  }
  
  @media (min-width: 992px) {
    // Large devices
    font-size: 1.3rem;
  }
}
```

## Section Styling Pattern

Sections follow a consistent pattern with background images and shades:

```scss
section {
  position: relative;
  margin-left: 0.5rem;
  margin-right: 0.5rem;
  
  .section-heading {
    color: var(--sfggc-section-heading-color);
    background-color: var(--sfggc-section-heading-bg-color);
  }
  
  .section-image-background {
    position: absolute;
    z-index: -50;
    background-repeat: no-repeat;
    background-position: center top;
    background-size: cover;
  }
  
  .section-background-shade {
    position: absolute;
    z-index: -49;
    background-color: black;
  }
}
```

## SCSS Module Usage

Use SCSS modules for component-specific styles:

```scss
// ComponentName.module.scss
.ComponentName {
  padding: 1rem;
  
  .Content {
    text-align: center;
    
    .Title {
      font-family: var(--sfggc-title-font-family);
      font-size: calc(2rem + 5vw);
    }
  }
}
```

## Typography

- Use `var(--sfggc-title-font-family)` for headings
- Use `calc()` for responsive font sizes
- Apply text shadows for better readability over images

## Image Handling

- Use responsive images with multiple sizes
- Store images in [src/images/](mdc:src/images/) organized by category
- Use background images for decorative elements
- Apply background shades for text readability

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StarlightEnt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/StarlightEnt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
