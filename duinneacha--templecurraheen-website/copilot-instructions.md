## templecurraheen-website

> Gallery image management and data structure


# Gallery Management Guidelines

## Gallery Data Structure ([src/\_data/gallery.json](mdc:src/_data/gallery.json))

### Image Object Format

```json
{
  "src": "/assets/imgs/filename.jpg",
  "alt": "Descriptive alt text",
  "title": "Display title",
  "description": "Longer description for caption",
  "category": "general|headstones|historical",
  "date": "YYYY or YYYY-MM-DD",
  "graveReference": "grave-slug-name" // optional
}
```

### Categories

- **general**: Entrance views, grounds, overall cemetery shots
- **headstones**: Individual graves, monuments, burial markers
- **historical**: Archive photos, documentation, vintage images

## Adding New Images

### File Placement

- Store images in [assets/imgs/](mdc:assets/imgs/)
- Use descriptive filenames (kebab-case preferred)
- Supported formats: .jpg, .png

### Gallery Integration Process

1. Add physical image files to `assets/imgs/`
2. Add metadata entries to `gallery.json`
3. Images automatically appear on gallery page with filtering
4. Clickable lightbox functionality included

### Gallery Features ([src/gallery/index.njk](mdc:src/gallery/index.njk))

- Responsive grid layout
- Category filtering buttons
- Lightbox modal for full-size viewing
- Automatic responsive image sizing
- Links to related grave records when `graveReference` provided

## Image Optimization

- Large images are acceptable (500KB+ files present)
- Images auto-sized to grid: `h-48 object-cover`
- Lazy loading applied: `loading="lazy"`
- Alt text required for accessibility

## Gallery Page Structure

- Filter buttons for all categories
- Masonry-style grid layout
- JavaScript for filtering and lightbox
- Contribution section for community photos

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duinneacha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/duinneacha)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
