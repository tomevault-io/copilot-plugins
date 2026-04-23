## homepage-lite

> go test -v ./... # Verbose output

# Developer Guidelines for Agents

## Build & Test Commands
```bash
# Build (Go 1.25.1+)
make build
# or
go build .

# Run tests (no tests currently exist)
go test ./...
go test -v ./... # Verbose output
go test ./... -run TestName # Run specific test

# Lint & Format
go vet ./...
gofmt -w . # Format code
golangci-lint run # If installed
```

## Project Structure
```
homepage-lite/
├── main.go                 # Main application entry point
├── config.yaml            # Configuration file
├── templates/
│   └── index.html         # Main HTML template
├── static/
│   ├── styles.css         # Main stylesheet
│   ├── themes/            # Theme CSS files
│   │   ├── default.css
│   │   ├── catppuccin-latte.css
│   │   ├── tokyo-night.css
│   │   ├── nord.css
│   │   ├── dracula.css
│   │   └── gruvbox.css
│   └── dashboard-icons/   # Custom service icons
└── Makefile              # Build commands
```

## Code Style Guidelines

### Imports
- Group stdlib imports first, then external packages
- Group imports by blank lines, ordered alphabetically within groups
- Use full import paths, avoid dot imports

### Types and Interfaces
- Use meaningful names that describe the type's purpose
- Place interface definitions near first use
- Define custom types for complex structures (see Config, SystemMetrics)
- Consider future extensibility in struct design

### Error Handling
- Use descriptive error messages with context
- Wrap errors using fmt.Errorf with %w for error chains
- Handle all error cases explicitly
- Use early returns for error conditions

### Naming Conventions
- Use camelCase for variable/function names
- Use PascalCase for exported names
- Keep names concise but descriptive
- Use consistent abbreviations (URL, SSE, API)

### Code Organization
- Group related variables and constants
- Use mutex protection for shared resources
- Implement clean shutdown and resource cleanup
- Document complex logic with clear comments

### Testing
- Write tests for all new features
- Use table-driven tests where appropriate
- Mock external dependencies
- Test error conditions and edge cases

### HTTP Handlers
- Validate input parameters
- Set appropriate headers
- Use proper status codes for errors
- Keep handlers focused and single-purpose

## Frontend Guidelines

### CSS
- Use CSS custom properties for theming (defined in theme files)
- Keep inline styles minimal (only for dynamic layout control)
- Use flexbox for layout (avoid grid unless necessary)
- Maintain consistent spacing (1rem, 1.5rem, 2rem)
- Card max-width: 400px (except in bottom mode for bookmarks)

### HTML Templates
- Use semantic HTML
- Keep templates simple and maintainable
- Use Iconify for icons: `<span class="iconify" data-icon="mdi:icon-name"></span>`

### JavaScript
- Minimal JS usage (mostly in inline scripts)
- Use localStorage for persisting user preferences (theme, layout)
- Event listeners for search, theme switching, layout switching

### Theming
To add a new theme:
1. Create `static/themes/theme-name.css`
2. Define CSS custom properties (--card-bg, --text-primary, etc.)
3. Set background image via --theme-background
4. Configure body::before filter for image styling
5. Add theme to selector in `templates/index.html`

## Configuration

### Settings Structure
```go
type Settings struct {
    Title     string `yaml:"title"`
    Port      int    `yaml:"port"`
    ShowTitle bool   `yaml:"show_title"`
}
```

### Service Structure
```go
type Service struct {
    Name        string `yaml:"name"`
    URL         string `yaml:"url"`
    Description string `yaml:"description"`
    Icon        string `yaml:"icon"`
}
```

### Bookmark Structure
```go
type Bookmark struct {
    Name string `yaml:"name"`
    URL  string `yaml:"url"`
    Abbr string `yaml:"abbr"`
}
```

## Version Management
- Version set via ldflags during build: `-ldflags "-X main.Version=..."`
- Use `make build` which automatically sets version from git tag or commit
- Version displayed in footer

## Additional Notes
- No Cursor rules (.cursor/rules/) or Copilot rules (.github/copilot-instructions.md) configured
- Project uses Go templates for frontend interactions and Server-Sent Events for real-time updates
- Background images should be high-quality (2560x width recommended)
- Theme background images hosted on Unsplash for performance

---
> Source: [jkerdreux-imt/homepage-lite](https://github.com/jkerdreux-imt/homepage-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
