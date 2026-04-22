## pawel-codes

> This is a Commodore 64-themed vanity site hosted on GitHub Pages using Jekyll. It features a BASIC interpreter interface powered by wwwbasic.js.

# CLAUDE.md - Project Guidelines for pawel.codes

## Project Overview
This is a Commodore 64-themed vanity site hosted on GitHub Pages using Jekyll. It features a BASIC interpreter interface powered by wwwbasic.js.

## File Structure
```
pawel.codes/
├── _config.yml              # Jekyll configuration
├── CNAME                    # GitHub Pages domain
├── index.html               # Main C64 interface
├── 404.html                 # C64-styled 404 page
├── CLAUDE.md                # This file - project guidelines
├── README.md                # Public documentation
├── REDIRECT_GUIDE.md        # Private docs for managing redirects
│
├── _data/                   # Jekyll data files
│   └── redirects.yml        # Centralized redirect configuration
│
├── _includes/               # Jekyll includes
│   └── head.html           # Common head elements
│
├── _layouts/               # Jekyll layouts
│   └── default.html        # Base layout
│   └── redirect.html       # Redirect template
│
├── _redirects/             # Redirect pages
│   └── decommission.md     # Example redirect
│
├── assets/                 # Static assets
│   ├── css/
│   │   └── c64.css        # C64 styling
│   ├── js/
│   │   ├── c64-terminal.js # Main terminal interface
│   │   ├── commands.js     # Built-in commands (HELP, DIR, etc.)
│   │   ├── programs.js     # BASIC program library
│   │   └── wwwbasic.js     # BASIC interpreter (vendored)
│   └── fonts/
│       ├── C64_Pro_Mono-STYLE.woff2
│       └── C64_Pro-STYLE.woff2
│
└── docs/                   # Reference documentation
    ├── C64-FONTS-USAGE.md  # Font documentation
    └── BASIC-REFERENCE.md  # BASIC command reference
```

## Development Guidelines

### Jekyll Configuration
- Use minimal Jekyll plugins (only jekyll-redirect-from)
- No complex build processes or JavaScript frameworks
- Keep it simple for GitHub Pages compatibility
- Use Jekyll data files for configuration when possible

### JavaScript Guidelines
- Vanilla JavaScript only (no frameworks/transpilation)
- ES6 features are acceptable (GitHub Pages serves to modern browsers)
- Modular structure with separate files for commands, programs, and terminal
- Use IIFE pattern or ES6 modules for encapsulation
- Vendor wwwbasic.js directly (don't use CDN for reliability)

### C64 Authenticity
- Use uppercase by default for all text
- Implement proper C64 color scheme (#6C5EB5 on #352879)
- Font size should be multiples of 8px (16px recommended)
- Terminal should be 40 columns wide
- Respect C64 BASIC conventions and error messages

### Command System Architecture
```javascript
// Command routing priority:
// 1. Check built-in commands (commands.js)
// 2. Check program library (programs.js)
// 3. Check localStorage programs (future feature)
// 4. Pass to wwwbasic interpreter
// 5. Show error if nothing matches
```

### Redirect Management
- Store all redirects in `_data/redirects.yml`
- Generate redirect pages from data file
- Use Jekyll front matter for redirect_to
- Document all redirects in REDIRECT_GUIDE.md

### Testing Checklist
- [ ] Site loads correctly on GitHub Pages
- [ ] CNAME file is present and correct
- [ ] All redirects work (test with curl -I)
- [ ] C64 font renders correctly
- [ ] Terminal accepts input
- [ ] BASIC programs execute
- [ ] Commands work (HELP, DIR, CREDITS)
- [ ] Error handling displays proper messages

## Code Style
- Use consistent indentation (2 spaces)
- Comment complex logic
- Keep functions small and focused
- Use descriptive variable names
- Follow C64 naming conventions where applicable

## Security Considerations
- Sanitize all user input before passing to wwwbasic
- Don't execute arbitrary JavaScript
- localStorage programs should be sandboxed
- No external API calls or data collection

## Performance Guidelines
- Minimize initial page load
- Lazy load wwwbasic.js if needed
- Use CSS for animations (cursor blink)
- Cache compiled BASIC programs
- Keep total page size under 1MB

## Accessibility
- Ensure keyboard navigation works
- Provide screen reader hints where possible
- High contrast colors (already C64 authentic)
- Monospace font for readability

## Future Features (Planned)
- [ ] Save/Load programs to localStorage
- [ ] Sound effects (SID chip emulation)
- [ ] Sprite graphics support
- [ ] Multiplayer shared programs
- [ ] C64 color palette switcher

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/makenai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
